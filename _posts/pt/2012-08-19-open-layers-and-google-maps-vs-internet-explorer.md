---
title: "OpenLayers e Google Maps vs. Internet Explorer"
lang: pt
last_modified_at: 2012-03-19T16:00:00-03:00
categories:
  - articles
tags:
  - code
  - openlayers
  - php
show_overlay_excerpt: false
header:
  image: /assets/images/2012/openlayers.jpg
  overlay_image: /assets/images/2012/openlayers.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

:uri-blank: https://arthurmolina.github.io/images/blank.gif

Estava eu desenvolvendo o meu site com mapas utilizando o [OpenLayers](http://www.openlayers.org/) como framework e, obviamente, testando e melhorando tudo no Firefox. A razão é muito simples: Firebug. Esta ferramenta melhora em muito a vida de um desenvolvedor front-end! Acho que depois ainda vou fazer uma avaliação dos debuggers de browsers. Mas isto fica para um outro post.

Apesar do Firefox já ter incorporado um debugger e de todos eles trabalharem com o mesmo comando de aviso de mensagens (console.log e outros), não consigo me desapegar do Firebug. Isto tem seu lado ruim também pois quando eu preciso testar a minha aplicação em outro navegador fico perdido. Até o Internet Explorer já incorporou sua ferramenta de debugger, e neste não tem como incluir um Firebug Lite como o Chrome (ou tem?). Então eu tenho que usar ele mesmo. Apesar de ter funções bem interessantes, como a possibilidade de testar a página em versões anteriores, no geral eu acho ele bem fraquinho…

Mas estavamos falando de OpenLayers. Estou usando vários Basemaps (que são os mapas que ficam por baixo das camadas que você vai trabalhar. Só é permitido mostrar um basemap por vez, afinal eles ocupam todo o espaço e não seria mesmo possível ver mais de um de uma vez, a não ser que usassemos transparência (depois vou testar essa possibilidade)…

Mas eis o problema. Estou usando como base o Google Maps, Google Satellite, Google Hibrid, Google Terrain e o OpenStreetMap, sendo este último a camada padrão. No Firefox, eles funcionavam numa boa. Quando ia testar a aplicação nos outros navegadores, elas abriam no OpenStreetMap (por padrão) e também parecia funcionar.

Um belo dia (depois não ficou tão belo assim) vou fazer uma demonstração do aplicativo e na maquina que eu ia mostrar não tinha o Firefox, apenas o IE 9 (pelo menos era o 9!) e as outras camadas base (as do Google) simplesmente não apareciam. Depois da vergonha que passei nessa demonstração fui procurar a razão para tal e não encontrava nada sobre o assunto. Atualizei a versão do OpenLayers, na época, para a 2.11 e nada. Procurei dentro do .js do OpenLayers, dentro das folhas de estilo e nada… De tanto procurar a razão desisti.

Quando estava já no final do desenvolvimento deste projeto, já em vias de entrar em produção, me empenhei mais uma vez para resolver este bug. A versão 2.12 do OpenLayers tinha acabado de sair do forno e, pensando que este bug estaria resolvido, atualizei mais uma vez. E mais uma vez, nada.

Mais um bom tempo de pesquisa quando encontrei a solução em um [fórum do Drupal](https://www.drupal.org/node/1364304): para corrigir o problema eu precisava apenas substituir o arquivo {uri-blank}[blank.gif] que estava com algum defeito e impedia a visualização das imagens do Google.

Foi só substituir este arquivo nas imagens do OpenLayers que o Google Maps apareceu no IE como mágica.

Mas ficam algumas perguntas:

- O que tinha no blank.gif anterior que impedia de visualizar?
- Por que este problema só ocorria no IE?
- Como eu consegui perder tanto tempo com uma coisa tão tola?
