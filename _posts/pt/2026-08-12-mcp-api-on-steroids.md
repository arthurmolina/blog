---
title: "MCP: uma API em steroids (e por que construímos uma só para a nossa operação)"
lang: pt
last_modified_at: 2026-08-12T20:00:00-03:00
categories:
  - articles
tags:
  - mcp
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

A palavra da moda agora é MCP (Model Context Protocol). O protocolo foi lançado pela Anthropic em novembro de 2024 como um padrão aberto para conectar modelos de linguagem a sistemas externos, e em pouquíssimo tempo virou o jeito padrão de dar "mãos" a um LLM. Claude, ChatGPT, Cursor, Gemini, todos usando MCP. Agora existem sites dedicados a cadastrar e listar uma diversidade enorme de servidores MCP.

Na prática, um servidor MCP nada mais é que uma API anabolizada. Uma mistura de REST com GraphQL, com uma diferença que muda tudo: quem consome não é um desenvolvedor lendo documentação, é um modelo de linguagem decidindo em tempo real o que chamar e em que ordem.

## O que o MCP herdou do REST

Até pouco tempo atrás, essa comparação seria forçada. O MCP nasceu como um protocolo stateful e bidirecional sobre JSON-RPC: o cliente abria uma conexão, fazia um handshake de `initialize`, recebia um `Mcp-Session-Id` e carregava essa sessão em todas as chamadas seguintes. Isso funcionava muito bem para um processo rodando no seu laptop e muito mal para um servidor remoto com mais de uma instância.

A especificação 2026-07-28 é a maior revisão do protocolo desde o lançamento, e aproximou o MCP do REST em três pontos.

**Statelessness.** O handshake de `initialize` e o header `Mcp-Session-Id` foram removidos. Cada requisição carrega no próprio campo `_meta` a versão do protocolo, a identidade e as capacidades do cliente, então qualquer instância do servidor pode atender qualquer requisição. Um load balancer round-robin comum resolve, sem sticky session e sem session store compartilhado. Quando o servidor precisa manter estado entre chamadas, ele devolve um handle explícito que o modelo passa de volta como argumento, algo muito parecido com o ID de um recurso na URL.

**Infraestrutura HTTP de sempre.** O transporte HTTP agora expõe o método chamado em um header (`Mcp-Method`). Isso permite rotear, aplicar rate limit e política de autorização no gateway sem abrir o corpo da requisição. A autorização segue OAuth, cada vez mais alinhada com OpenID Connect.

**Cache.** As respostas de `tools/list`, `resources/list` e `prompts/list` podem ser cacheadas pelo cliente pelo tempo que o servidor indicar no campo `ttlMs`.

Há ainda uma herança mais antiga: no MCP, *resources* são identificados por URI, exatamente como recursos REST.

## O que o MCP herdou do GraphQL

Do GraphQL, o MCP herdou a ideia de uma API que documenta a si mesma. Um cliente GraphQL faz introspection e descobre o schema. Um cliente MCP chama `tools/list` e recebe cada ferramenta com nome, descrição em linguagem natural e um JSON Schema dos parâmetros (e, opcionalmente, da saída). Não existe um Swagger separado que pode ficar desatualizado em relação ao código: o contrato é servido pelo próprio servidor.

<!-- PREENCHER: trocar pelo JSON de uma ferramenta real do seu servidor -->
```json
{
  "name": "search_items",
  "description": "Fulltext search over catalog Items (name, description, keywords, category) to resolve item_id. A numeric query looks up by id directly. Pass region_code to restrict results to a specific market.",                                    
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Free-text search string or numeric item id"
      },
      "region_code": {
        "type": "integer",
        "description": "Region code to restrict results to a local market"
      },
      "limit": {
        "type": "integer",
        "description": "Maximum number of results to return (default: 20, max: 50)"
      }
    },
    "required": ["query"]
  }
}
```

Repare que a descrição não é escrita para um humano. Ela é, na prática, um prompt: diz ao modelo *quando* usar a ferramenta. Isso muda bastante a forma de pensar documentação.

Também veio do GraphQL o endpoint único. Tudo passa pelo mesmo endereço, e o que muda é o método JSON-RPC chamado (`tools/call`, `resources/read` e assim por diante), não a URL. Não que isso seja uma coisa boa. Todo mundo que já teve que debugar em GraphQL sabe o quão chato é descobrir de que método está vindo um erro dentro de uma query com múltiplas entradas.

## O que o MCP deixou para trás

O GraphQL entrega ao cliente uma linguagem de consulta. O cliente monta a query que quiser, com a profundidade que quiser, e o servidor que lute para se defender com limite de profundidade, análise de custo e persisted queries. O MCP não tem isso. O cliente só pode chamar as operações que o servidor decidiu expor, com os parâmetros que o servidor definiu. A superfície fica fechada por construção.

E o famoso N+1? Aqui vale ser honesto. O N+1 é um problema clássico do GraphQL (cada resolver disparando uma query por item, geralmente resolvido com DataLoader), e o MCP não o elimina por mágica. O problema só muda de camada. Se você expõe ferramentas granulares demais, como `buscar_pedido` e `buscar_cliente`, o modelo vai chamá-las uma a uma, em loop, para responder algo como "quais pedidos de amanhã ainda não foram pagos?". São N+1 chamadas de ferramenta, cada uma com latência de rede e custo em tokens.

A defesa é de design, não de protocolo: ferramentas orientadas a intenção, que devolvem de uma vez tudo o que o modelo precisa para aquela tarefa. Uma chamada a `pedidos_pendentes(data_entrega:)` no lugar de dez chamadas a `buscar_pedido`. Do lado do servidor, o bom e velho `includes` do ActiveRecord continua fazendo o trabalho pesado.

Saber o que o usuário quer é importante. Em vez de expor os models e entregar o dado bruto devemos perguntar o que o usuário quer e responder com a informação lapidada.

## Uma API REST bem documentada não daria conta?

Essa discussão é legítima e não tem resposta única. Quem acha que sim argumenta que os LLMs já leem especificações OpenAPI muito bem, que um servidor MCP é mais uma superfície para manter, versionar e proteger, e que muitos servidores MCP por aí são apenas wrappers finos sobre uma API REST que já existia. Há também a questão de segurança: prompt injection e descrições de ferramentas maliciosas são riscos reais quando um modelo age com base em texto que ele não controla.

Quem acha que não lembra que o MCP padroniza o lado do cliente: um mesmo servidor funciona em qualquer cliente compatível, sem escrever código de integração para cada um. Descoberta e autorização também são padronizadas. E, principalmente, uma API REST é desenhada em torno de recursos, enquanto um bom servidor MCP é desenhado em torno de intenções. Espelhar endpoints REST um para um em ferramentas costuma dar exatamente no N+1 descrito acima. Por fim, o protocolo oferece recursos pensados para interação com pessoas, como pedir uma confirmação ao usuário no meio de uma chamada, acompanhar tarefas longas e até renderizar interfaces vindas do servidor.

Na minha experiência, a resposta depende de quem está do outro lado. E foi pensando nisso que tomamos uma decisão um pouco diferente.

## O nosso caso: um MCP para dentro de casa

Trabalho na Reventals, um marketplace da TapGoods para aluguel de equipamentos para festas e eventos. Na prática, funciona como um e-commerce, com algumas particularidades.

Quando se fala em MCP para e-commerce, a imagem que vem à cabeça é um agente comprando em nome do cliente. A nossa proposta foi outra: um servidor MCP interno, feito para os nossos operadores. Quem usa é o time de operação, que antes dependia do painel administrativo, ou de mim, para modificar informaçoes de páginas destinadas à marketing e específicas pra cada região, adicionar parceiros, pesquisar items e categorias. Com o servidor conectado ao Claude Desktop ou Claude Code (sim, o nosso time de operação tem usado também o Claude Code com sucesso), eles consultam e gerenciam o site em linguagem natural.

Alguns motivos nos levaram a começar por dentro. O público é conhecido e autenticado, o que reduz bastante o risco. Cada pergunta nova da operação, que antes viraria um ticket ou uma tela nova no admin, passa a poder ser respondida combinando ferramentas que já existem. E um uso interno é um laboratório seguro para aprender a desenhar ferramentas antes de expor qualquer coisa a clientes.

### Decisões de design

**Ferramentas por intenção, não por tabela.** Em vez de espelhar os models do Rails, cada ferramenta corresponde a algo que um operador de fato pergunta ou faz. Por exemplo temos páginas de casamentos por regiões e exemplos de estilos de casamento em cada região, com o auxílio do Claude, o time consegue não só cadastrar novas páginas como também ter ajuda para criar copys.

**Leitura separada de escrita.** Ferramentas de consulta e ferramentas que alteram dados são tratadas de forma diferente. Deixamos explícito os métodos de escrita e leitura com anotações de ferramenta como `destructiveHint`, lista restrita de operações de escrita.

**Auditoria.** Cada `tools/call` é registrado com operador, argumentos e resultado. As alterações são guardadas de modo que podemos recuperar caso alguma informação se perca (vai que o Claude decide alucinar).

**Descrições como prompt.** As descrições das ferramentas foram escritas e reescritas olhando para o comportamento do modelo, não para a leitura de um desenvolvedor. Contamos com o time de operações fazendo testes assistidos e informando como queria "conversar" com o modelo.

## Conclusão

MCP não é mágica. É design de API com um consumidor diferente, e a especificação mais recente, ao se tornar stateless, deixou o protocolo ainda mais parecido com o que a gente já conhece de REST. A parte nova de verdade está em desenhar ferramentas para um modelo que decide sozinho o que chamar.

E talvez o uso mais interessante não seja o mais óbvio. Antes de colocar um agente para conversar com os seus clientes, vale perguntar quem dentro da sua empresa passa o dia pedindo relatórios, abrindo tickets ou navegando por telas de admin. Pode ser que o seu primeiro servidor MCP seja para essas pessoas.
