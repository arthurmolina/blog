---
title: "Minimizando Gems/Bibliotecas no Ruby on Rails: Mantendo o Código Dentro do Repositório"
lang: pt
last_modified_at: 2024-01-11T16:00:00-03:00
categories:
  - Blog
tags:
  - code
  - electronics
  - arduino
show_overlay_excerpt: false
toc: true
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

Quando estamos desenvolvendo sistemas web é fácil adicionar uma nova gem ou biblioteca sem pensar muito nisso. Ela faz o que precisamos, se eu tiver que criar essa funcionalidade pode demorar muito. Vai me poupar muito trabalho. O código já está lá pronto para ser usado. Tem toda uma documentação, testes e (às vezes) toda uma comunidade fornecendo suporte e desenvolvendo novos recursos.

Então… Por que não?

Porém, embora gems e bibliotecas possam oferecer muitos benefícios, existem razões convincentes para exercer cautela e minimizar seu uso em favor de manter o código dentro do repositório.

# Incompatibilidades de Versão

Uma das principais preocupações ao depender fortemente de gems e bibliotecas externas é o risco de incompatibilidades de versão. À medida que o Rails e suas gems associadas evoluem, versões mais novas podem introduzir mudanças incompatíveis ou conflitos com o código existente. Isso pode levar a soluções de problemas trabalhosas, resolução de dependências e potencial tempo de inatividade. Ao minimizar dependências externas, os desenvolvedores podem mitigar o risco de conflitos de versão e manter maior controle sobre a estabilidade da base de código.

Me deparei com um problema assim recentemente: estava trabalhando em uma aplicação que tinha muitos anos com uma versão desatualizada do Rails e do Ruby. Muitas gems estavam antigas e precisavam ser atualizadas. O que aconteceu foi que uma das gems precisava ser atualizada de qualquer forma porque estava integrada com um serviço de terceiros. Para poder realizar a atualização, foi necessário atualizar várias outras gems juntas, causando incerteza sobre se tudo funcionaria. Ainda bem que tínhamos uma cobertura de testes aceitável e uma maravilhosa equipe de QA.

# Manutenção de Código

Cada gem ou biblioteca adicionada a um projeto Rails representa mais um trecho de código que deve ser monitorado, atualizado e mantido. Com o passar do tempo, manter a compatibilidade com dependências em evolução torna-se cada vez mais desafiador, especialmente se certas gems saírem do desenvolvimento ativo ou carecerem de documentação adequada. Ao manter o código dentro do repositório, os desenvolvedores podem simplificar os esforços de manutenção, garantindo que todos os componentes da aplicação permaneçam atualizados e coesos.

No mesmo código mencionado acima temos vários desafios que por ora são apenas dívidas técnicas, mas que em um momento inoportuno teremos que lidar. Por exemplo, esta aplicação está usando uma gem que já está deprecada; quando precisarmos atualizar outra gem que depende dela, teremos que modificar todo o código onde ela é usada para colocar um substituto. Esse momento vai chegar, de uma forma ou de outra.

# Riscos de Segurança

Dependências externas podem introduzir vulnerabilidades de segurança em uma aplicação. Embora gems de boa reputação sejam tipicamente mantidas com as melhores práticas de segurança em mente, vulnerabilidades ainda podem surgir, especialmente em bibliotecas menos conhecidas ou não mantidas. Ao minimizar o número de dependências externas, os desenvolvedores reduzem a superfície de ataque e podem implementar medidas de segurança mais eficazes adaptadas à sua aplicação específica.

Ao mesmo tempo, somos alvos de hackers que podem deliberadamente adicionar backdoors e códigos maliciosos aos pacotes baixados para nossas aplicações. A mídia já registrou vários casos de brechas de segurança em bibliotecas, principalmente do [NPM](https://www.securityweek.com/dozens-of-malicious-npm-packages-steal-user-system-data/). Mas o RubyGems também não está a salvo de inserção de [código](https://www.securityweek.com/two-malware-laced-gems-found-rubygems-repository/) [malicioso](https://www.theregister.com/2019/08/20/ruby_gem_hacked/).

# Overhead de Performance e Inchaço de Dependências

Cada gem/biblioteca incluída em um projeto Rails incorre em um overhead de performance, por menor que seja. Embora os impactos individuais de performance possam parecer negligenciáveis, eles podem se acumular com o tempo, especialmente em aplicações de grande escala com numerosas dependências. Minimizar dependências externas permite que os desenvolvedores otimizem a performance reduzindo overhead desnecessário e simplificando a execução da aplicação.

A dependência excessiva de gems/bibliotecas pode levar ao inchaço de dependências, onde um projeto Rails fica sobrecarregado com um número excessivo de dependências externas. Isso pode impedir a agilidade de desenvolvimento, aumentar a complexidade de implantação e dificultar a escalabilidade. Ao priorizar código autocontido dentro do repositório, os desenvolvedores podem manter uma base de código mais enxuta e gerenciável que é mais fácil de manter e escalar ao longo do tempo.

# Práticas

O autor do livro [Sustainable Web Development with Ruby on Rails](https://sustainable-rails.com/), David Copeland, sugere que devemos atualizar dependências Cedo e Frequentemente. Seu conselho é definir um dia por mês para as dependências serem atualizadas. Ou seja: "rodaríamos bundle update em nossas aplicações Rails, executaríamos os testes, corrigiríamos o que estava quebrado, e então estaríamos atualizados" (página 380). Outro conselho do autor é manter uma Política de Versionamento e Automatizar Atualizações de Dependências.

Outra boa forma de estar preparado para atualizações de pacotes, além de usar bibliotecas mínimas, é ter boa cobertura de testes automatizados. E nunca esquecer os testes de integridade!

Trabalhei em um sistema cujo Ruby estava na versão 2.6.7 e precisava ser atualizado para pelo menos a próxima versão porque estava hospedado no Heroku e o container que funcionava com essa versão não seria mais suportado em 2 meses (e em alguns meses a mais não estaria mais disponível para implantação). Na época em que entrei neste projeto não havia uma linha de teste escrita, e ainda não tínhamos uma equipe de QA. Foram 2 meses complicados escrevendo um mínimo de cobertura de testes para que pudéssemos realizar a atualização das versões do Ruby e do Rails com mais confiança.

# Conclusão

Embora gems/bibliotecas indiscutivelmente ofereçam funcionalidade valiosa e conveniência no desenvolvimento Ruby on Rails, seu uso indiscriminado pode introduzir numerosos desafios e riscos. Ao priorizar código autocontido dentro do repositório, os desenvolvedores podem minimizar incompatibilidades de versão, simplificar esforços de manutenção, mitigar riscos de segurança, otimizar a performance e evitar o inchaço de dependências. Em última análise, exercer contenção na adoção de dependências externas promove maior estabilidade, segurança e manutenibilidade do código, garantindo o sucesso a longo prazo das aplicações Rails.
