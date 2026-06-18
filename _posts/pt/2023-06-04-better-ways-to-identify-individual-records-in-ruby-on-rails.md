---
title: "Melhores formas de identificar registros individuais no Ruby on Rails"
lang: pt
last_modified_at: 2023-06-04T20:00:00-03:00
categories:
  - articles
tags:
  - nonsense
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
toc: true
---

Quando queremos acessar um registro específico de um model, a forma mais fácil é usar o identificador único como chave de acesso. Por padrão em bancos de dados SQL temos uma chave primária inteira sequencial. Mas essa não é a única opção possível. Também temos a possibilidade de usar UUID, Slug e Hash ID ou Token. Cada método oferece vantagens e considerações distintas, impactando fatores como segurança, legibilidade e performance do banco de dados. Extraindo insights de implementações do mundo real e melhores práticas da indústria, exploramos as nuances de cada método e suas implicações para aplicações modernas. Vamos comparar essas abordagens e explorar por que plataformas como o YouTube optam pelos Hash IDs.

# IDs Sequenciais: Abraçando a Previsibilidade e Eficiência

Esta é a opção mais fácil de implementar, pois é a que está disponível na maioria dos casos. Quando criamos um novo model ou geramos um scaffold, o ID Sequencial já é o padrão estabelecido. IDs Sequenciais mantêm uma sequência numérica previsível, facilitando operações de banco de dados como consultas de intervalo e indexação.

## Vantagens

1. **Sequencialidade**: IDs Sequenciais mantêm uma sequência numérica previsível, facilitando operações de banco de dados como consultas de intervalo e indexação.
2. **Simplicidade**: IDs Sequenciais estão alinhados com práticas convencionais de banco de dados, exigindo configuração mínima ou dependências externas.
3. **Eficiência**: IDs Sequenciais promovem indexação e armazenamento eficientes no banco de dados, otimizando a performance em certos casos de uso.

## Desvantagens

1. **Previsibilidade**: IDs Sequenciais expõem informações de sequência do banco de dados, potencialmente permitindo que atacantes infiram padrões ou explorem vulnerabilidades.
2. **Escalabilidade**: Em ambientes distribuídos ou aplicações de alto throughput, gerenciar IDs sequenciais em múltiplos nós pode apresentar desafios de escalabilidade.
3. **Legibilidade**: IDs numéricos carecem das qualidades legíveis por humanos de strings com hash, potencialmente impactando a experiência do usuário e a estética das URLs.

## Potencialidades

1. **Otimização de Performance**: IDs Sequenciais se destacam em cenários onde o acesso sequencial ou consultas de intervalo são prevalentes, aproveitando a indexação do banco de dados para melhor performance.
2. **Compatibilidade**: IDs Sequenciais se integram perfeitamente com esquemas de banco de dados e fluxos de trabalho existentes, minimizando esforços de migração e preocupações de compatibilidade.

## Como fazer

Você não precisa fazer nada além do básico. Este já é o padrão.


# UUIDs (Identificadores Universalmente Únicos): Garantindo Unicidade Global e Interoperabilidade

Identificadores Universalmente Únicos (UUIDs) fornecem garantias incomparáveis de unicidade global em sistemas distribuídos e no tempo. Esta abordagem descentralizada para geração de identificadores alivia preocupações sobre riscos de colisão, tornando os UUIDs ideais para implantações em grande escala e arquiteturas descentralizadas. No entanto, os IDs muito longos e a falta de conexão com informações mais humanas acabam dificultando seu uso. É uma boa opção para quem tem bancos de dados distribuídos ou quer mesclar bancos de dados. Não o usaria se não tivesse uma dessas razões.

## Vantagens

1. **Unicidade**: UUIDs garantem unicidade global em sistemas distribuídos e no tempo, mitigando riscos de colisão mesmo em implantações em grande escala.
2. **Descentralização**: UUIDs podem ser gerados sem coordenação centralizada, tornando-os adequados para arquiteturas descentralizadas ou distribuídas.
3. **Padronização**: UUIDs aderem a padrões estabelecidos como RFC 4122, garantindo interoperabilidade e compatibilidade entre plataformas e frameworks.

## Desvantagens

1. **Comprimento**: UUIDs são mais longos do que IDs sequenciais ou com hash, potencialmente impactando o armazenamento no banco de dados e o overhead de transmissão de rede.
2. **Legibilidade**: UUIDs carecem de qualidades amigáveis ao humano, aparecendo frequentemente como strings aleatórias de caracteres, o que pode diminuir a estética das URLs e a experiência do usuário.
3. **Complexidade de Indexação**: Indexar UUIDs em bancos de dados pode incorrer em penalidades de performance devido ao aumento dos requisitos de armazenamento e fragmentação de índice.

## Potencialidades

1. **Unicidade Global**: UUIDs fornecem garantias de unicidade incomparáveis, tornando-os ideais para cenários onde a prevenção de colisão é primordial.
2. **Geração Descentralizada**: UUIDs facilitam arquiteturas de sistemas distribuídos permitindo geração descentralizada de IDs sem coordenação centralizada.
3. **Interoperabilidade**: Formatos UUID padronizados promovem interoperabilidade e compatibilidade entre diversas plataformas, frameworks e tecnologias.

## Como fazer

Você provavelmente já se deparou com vários artigos explicando como usar UUID no Rails, então vou colocar apenas a versão resumida. Até agora só usei UUID em banco de dados Postgres, então se você usa MySQL, estou em dívida.

```ruby
# /db/migrations/0000000000_enable_pgcrypto_extension.rb
class EnablePgcryptoExtension < ActiveRecord::Migration[6.1]
  def change
    enable_extension 'pgcrypto'
  end
end

# /config/initializers/generators.rb
Rails.application.config.generators do |g|
  g.orm :active_record, primary_key_type: :uuid
end

# /db/migrations/000000_create_users.rb
class CreateUsers < ActiveRecord::Migration[6.1]
  def change
    create_table :users, id: :uuid do |t|
      t.string :name
    end
  end
end
```

# Hash IDs: Equilibrando Segurança e Legibilidade

Hash IDs representam uma abordagem criptográfica para gerar identificadores únicos transformando valores numéricos em strings com hash. Essa técnica oferece várias vantagens, incluindo segurança aprimorada, melhor legibilidade e evitação de colisões. Ao obscurecer sequências de banco de dados, os Hash IDs mitigam o risco de ataques de enumeração e acesso não autorizado. Plataformas como o YouTube aproveitam os Hash IDs para criar URLs amigáveis ao usuário, melhorando a usabilidade e o reconhecimento da marca. No entanto, os Hash IDs também apresentam desafios, como não-sequencialidade e dependência de algoritmos de hash. Apesar dessas considerações, os Hash IDs continuam sendo uma escolha popular para aplicações que priorizam segurança e experiência do usuário.

## Vantagens

1. **Segurança**: Hash IDs obscurecem a sequência subjacente do banco de dados, melhorando a privacidade e dissuadindo atividades maliciosas como raspagem de dados e ataques de enumeração.
2. **Legibilidade**: Hash IDs podem ser ajustados para gerar strings legíveis por humanos, melhorando a experiência do usuário e a estética das URLs.
3. **Evitação de Colisões**: Algoritmos de hash se esforçam para minimizar a probabilidade de colisões, garantindo a unicidade dos IDs gerados dentro de restrições práticas.
4. **Personalização**: Hash IDs oferecem flexibilidade para adaptar esquemas de codificação e incorporar lógica específica da aplicação.

## Desvantagens

1. **Não-Sequencialidade**: Hash IDs são não-sequenciais, o que pode complicar certas operações de banco de dados como consultas de intervalo ou ordenação.
2. **Dependência de Algoritmos**: A eficácia dos Hash IDs depende da robustez criptográfica dos algoritmos de hash subjacentes.
3. **Espaço de Codificação Limitado**: Dependendo da configuração, os Hash IDs podem ter um espaço de codificação finito, potencialmente levando a colisões em ambientes de alto throughput.

## Potencialidades

1. **Segurança Aprimorada**: Com configuração adequada, os Hash IDs podem fortalecer a segurança dos dados e mitigar riscos associados à exposição de IDs numéricos.
2. **Experiência do Usuário Melhorada**: Hash IDs permitem a criação de URLs amigáveis e legíveis, melhorando a usabilidade e o reconhecimento da marca.
3. **Escalabilidade e Performance**: Hash IDs promovem indexação e recuperação eficientes em ambientes de alto throughput.
4. **Personalização e Otimização**: Hash IDs oferecem flexibilidade, permitindo esquemas de codificação personalizados e incorporação de metadados.

## Implementação de Hash ID do YouTube: Um Estudo de Caso

O YouTube, uma das maiores plataformas de compartilhamento de vídeo do mundo, utiliza extensivamente Hash IDs por várias razões:

1. **Segurança**: Hash IDs obscurecem a sequência subjacente do banco de dados, melhorando a privacidade e dissuadindo atividades maliciosas.
2. **Legibilidade**: Hash IDs permitem ao YouTube gerar URLs de vídeo amigáveis ao usuário, melhorando a usabilidade e o reconhecimento da marca.
3. **Consistência**: Hash IDs fornecem um formato consistente para identificadores de vídeo, simplificando o gerenciamento de recursos e a geração de URLs na plataforma.
4. **Escalabilidade**: Com milhões de vídeos enviados diariamente, os Hash IDs facilitam indexação e recuperação eficientes.
5. **Personalização**: O YouTube pode personalizar os Hash IDs para incorporar metadados adicionais ou esquemas de codificação.

## Como fazer

Temos o algoritmo de hash perfeito para este trabalho: Hashids. Existem várias Gems diferentes que fazem o mesmo trabalho que o código abaixo, mas tenho uma filosofia que diz que se um arquivo resolve o problema não há necessidade de adicionar outra biblioteca externa resultando em mais complexidade e problemas na atualização. Não há necessidade de [ter uma gem e ter que lidar com os problemas de mantê-la](/blog/minimizing-gems-libraries-in-ruby-on-rails/). Mas da mesma forma que fiz para o Gravatar, abri a [gem hashid-rails](https://github.com/jcypret/hashid-rails) e peguei o que é importante:

```ruby
# /app/models/concerns/hashable.rb
# frozen_string_literal: true

# Hashids - Change the numeric identifications to hash identifiers
# Based on a simplification of 'hashid-rails gem' but avoiding yet another gem that could be a concern file.
module Hashable
  extend ActiveSupport::Concern

  def hashid
    self.class.encode_id(id)
  end

  def to_param
    hashid
  end

  # Methods for the Model Class
  module ClassMethods
    def encode_id(ids)
      if ids.is_a?(Array)
        ids.map { |id| hashid_encode(id) }
      else
        hashid_encode(ids)
      end
    end

    def decode_id(ids)
      if ids.is_a?(Array)
        ids.map { |id| hashid_decode(id) }
      else
        hashid_decode(ids).first
      end
    end

    def find(*ids)
      expects_array = ids.first.is_a?(Array)

      uniq_ids = ids.flatten.compact.uniq
      uniq_ids = uniq_ids.first unless expects_array || uniq_ids.size > 1

      if uniq_ids.is_a?(Array)
        uniq_ids.map do |id|
          id.is_a?(Integer) ? super(id) : super(decode_id(id))
        end
      else
        uniq_ids.is_a?(Integer) ? super(uniq_ids) : super(decode_id(uniq_ids))
      end
    end

    def find_by_hashid(hashid)
      find_by id: decode_id(hashid)
    end

    def find_by_hashid!(hashid)
      find(decode_id(hashid))
    end

    def hashid
      encode_id(id)
    end

    private

    def hashids
      Hashids.new(self.class.name + ENV.fetch('HASHID_TOKEN', ''), 8)
    end

    def hashid_encode(id)
      return nil if id.nil?

      hashids.encode(id)
    end

    def hashid_decode(id)
      hashids.decode(id)
    rescue Hashids::InputError
      id
    end
  end
end
```

Agora basta adicionar uma linha ao model que queremos usar hashids e está tudo pronto:

```ruby
class User < ApplicationRecord
  include Hashable
end
```

# Slugs: Legibilidade Humana e Amigabilidade à URL

Slugs representam uma string legível por humanos e amigável à URL derivada de um texto fonte, tipicamente usada para representar títulos, nomes ou outros elementos descritivos.

## Vantagens

1. **Legibilidade**: Slugs criam URLs amigáveis ao humano que são fáceis de ler e entender, melhorando a experiência geral do usuário.
2. **Otimização de SEO**: Ao incluir palavras-chave relevantes e elementos descritivos na URL, os slugs contribuem para melhor visibilidade e ranking nos motores de busca.
3. **Consistência de Marca**: Slugs consistentemente formatados contribuem para o reconhecimento e consistência da marca nas páginas web.
4. **Compatibilidade**: Slugs aderem aos padrões web e são compatíveis com vários navegadores e plataformas.

## Desvantagens

1. **Unicidade**: Garantir a unicidade dos slugs em um grande conjunto de dados pode ser desafiador, especialmente em ambientes onde múltiplos usuários podem criar conteúdo simultaneamente.
2. **Limitações de Caracteres**: URLs têm limitações de caracteres, e os slugs devem aderir a essas restrições.
3. **Complexidade de Processamento de Texto**: Gerar slugs envolve processar e sanitizar entradas de texto para remover caracteres especiais, espaços em branco e outros caracteres não amigáveis à URL.
4. **Internacionalização**: Suportar conteúdo multilíngue apresenta desafios para geração de slugs.

## Potencialidades

1. **Personalização**: Slugs oferecem flexibilidade em personalização, permitindo que desenvolvedores incorporem elementos de marca, palavras-chave e metadados relevantes nas URLs.
2. **Aprimoramento de SEO**: Otimizar slugs para SEO pode impactar significativamente a visibilidade e o ranking nos motores de busca.
3. **Engajamento do Usuário**: Slugs amigáveis ao usuário melhoram o engajamento fornecendo URLs claras e descritivas que os usuários podem facilmente lembrar e compartilhar.
4. **Localização**: Adaptar slugs a diferentes idiomas e regiões facilita esforços de localização.

## Como fazer

Você poderia usar um script similar ao que forneci acima para sobrescrever o método find e usar o slug como parâmetro principal, ou simplesmente mudar a chamada no controller, que seria:

```ruby
@user = User.find(params[:id])
```

Para:

```ruby
@user = User.find_by(slug: params[:id])
```


# Conclusão: Minha Opinião

Então qual opção usar? Como sempre, a resposta é... depende.

Primeiramente, ID sequencial raramente é uma boa opção a menos que você queira que o ID sequencial faça parte da funcionalidade — por exemplo, você quer mostrar ao usuário que um ID é mais antigo que outro. Na época em que havia poucos aplicativos de mensagens e o ICQ dominava o mundo, ter um número de ICQ com menos de 6 dígitos era sinal de status, de que você era "veterano" na plataforma.

Como disse na mesma seção, UUID só faz sentido se você quiser manipular sistemas de banco de dados descentralizados. Tenha em mente que um UUID ocupa um pouco mais de espaço do que um inteiro, mas não tanto.

Minha opção preferida é o Hash ID — essa solução funciona para 99% dos casos. Não ocupa espaço extra no banco de dados, por outro lado requer um pouco mais de processamento de criptografia/descriptografia. Tenho um model de usuário? São faturas? São ordens de serviço? Joga um Hash ID. Tudo resolvido.

Por fim, se preciso que o ID faça parte de uma URL e preciso trabalhar SEO ou ter algum significado, a solução é o slug. O slug resultará em um aumento significativo no espaço do banco de dados e ainda mais criando um índice só para ele, mas em termos de ser algo mais amigável ao usuário não tem igual. Meu model de usuário é para compartilhar com outras pessoas (como no LinkedIn)? Estou criando IDs para posts de blog? Não tem o que pensar, é Slug.

Espero que essa coletânea de informações sobre o uso de ID tenha sido útil.
