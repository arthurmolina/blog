---
title: "Sobre o Gravatar"
lang: pt
last_modified_at: 2023-04-16T20:00:00-03:00
categories:
  - articles
tags:
  - nonsense
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

Gravatar, abreviação de "Globally Recognized Avatar" (Avatar Globalmente Reconhecido), é um serviço que permite aos usuários criar uma foto de perfil que os acompanha pela web. Lançado em 2007 por Tom Preston-Werner, o Gravatar rapidamente se tornou uma escolha popular para o gerenciamento de identidade online.

Em sua essência, o Gravatar atribui um avatar único ao endereço de e-mail de cada usuário. Quando os usuários comentam em blogs, participam de fóruns ou interagem em vários sites, a imagem do Gravatar aparece automaticamente ao lado de suas contribuições. Essa representação visual consistente fomenta um senso de identidade e comunidade em diferentes plataformas online.

Uma das principais vantagens do Gravatar é sua praticidade. Os usuários podem criar ou atualizar seu avatar uma vez, e ele se propagará automaticamente por todos os sites que suportam o Gravatar, eliminando a necessidade de fazer upload de uma foto de perfil individualmente em cada plataforma. Esse processo simplificado economiza tempo e garante consistência na identidade online.

Além disso, o Gravatar oferece aos usuários controle sobre suas configurações de privacidade, permitindo que escolham entre conteúdo público, privado ou classificado para seus avatares.

Em resumo, o Gravatar simplifica o gerenciamento de identidade online, fomenta o engajamento da comunidade e oferece aos usuários uma maneira conveniente e personalizável de se representarem na web.

Além disso, não há necessidade de criar um recurso na sua aplicação para salvar a foto do usuário, não precisamos ter espaço de armazenamento para essas fotos, o usuário não precisa passar pelo trabalho de enviar sua foto para cada aplicação que quiser usar, e se ele não quiser ser identificado, ainda podemos deixar um desenho de um robô ou alienígena engraçado que muda e mantém o mesmo padrão por e-mail.

# Como fazer

Existem [várias Gems](https://rubygems.org/search?query=gravatar) que podem fazer a tarefa. Mas essa é uma funcionalidade muito fácil de ter e um simples arquivo adicionado à pasta /app/models/concerns resolve o problema. Não há necessidade de [ter uma gem e ter que lidar com os problemas de mantê-la](/blog/minimizing-gems-libraries-in-ruby-on-rails/). Especialmente porque muitas das gems de gravatar que vi já estão muito desatualizadas.

Então basta adicionar o arquivo abaixo:

```ruby
# frozen_string_literal: true

# /app/models/concerns/gravatable.rb

# From:
# https://github.com/chrislloyd/gravtastic/blob/master/lib/gravtastic.rb
# https://chrislloyd.github.io/gravtastic/

require 'digest/md5'

# Module to generate Gravatar URL
module Gravatable
  extend ActiveSupport::Concern

  # The raw MD5 hash of the users' email. Gravatar is particularly tricky as
  # it downcases all emails. This is really the guts of the module,
  # everything else is just convenience.
  def gravatar_id
    Digest::MD5.hexdigest(send(self.class.gravatar_source).to_s.downcase)
  end

  # Constructs the full Gravatar url.
  def gravatar_url(options = {})
    options = {
      rating: 'PG',
      secure: true,
      filetype: :png,
      default: :robohash, # options: 404, mp (mystery person), identicon, monsterid, wavatar, retro, robohash, blank
      size: 200
    }.merge(options)
    gravatar_hostname(options.delete(:secure)) +
      gravatar_filename(options.delete(:filetype)) +
      "?#{url_params_from_hash(self.class.process_options(options))}"
  end

  # private

  # Creates a params hash like "?foo=bar" from a hash like {'foo' => 'bar'}.
  # The values are sorted so it produces deterministic output (and can
  # therefore be tested easily).
  def url_params_from_hash(hash)
    hash.map do |key, val|
      [self.class.gravatar_abbreviations[key.to_sym] || key.to_s, val.to_s].join('=')
    end.sort.join('&')
  end

  # Returns either Gravatar's secure hostname or not.
  def gravatar_hostname(secure)
    "http#{secure ? 's://secure.' : '://'}gravatar.com/avatar/"
  end

  # Munges the ID and the filetype into one. Like "abc123.png"
  def gravatar_filename(filetype)
    "#{gravatar_id}.#{filetype}"
  end

  # Methods for the Model Class
  module ClassMethods
    attr_accessor :gravatar_source

    def gravatar_field(source = :emailx)
      @gravatar_source = source
    end

    # Some options need to be processed before becoming URL params
    def process_options(options)
      processed_options = {}
      options.each do |key, val|
        case key
        when :forcedefault
          processed_options[key] = 'y' if val
        else
          processed_options[key] = val
        end
      end
      processed_options
    end

    def gravatar_abbreviations
      { size: 's',
        default: 'd',
        rating: 'r',
        forcedefault: 'f' }
    end
  end
end
```

Em qualquer model que tenha o campo de e-mail, basta adicionar o Gravatable como abaixo:

```ruby
class User
  include Gravatable

  gravatar_field :email
end

User.last.gravatar_url({ size: 40 })
```

Como você pode ver no código, ele não é originalmente meu. Criei uma versão a partir da [gem Gravtastic](https://github.com/chrislloyd/gravtastic/commits/master/) que não é atualizada desde 2016.
