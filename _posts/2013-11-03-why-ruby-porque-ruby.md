---
title: "Why ruby? Por que Ruby?"
lang: en
last_modified_at: 2013-11-03T16:00:00-03:00
categories:
  - articles
tags:
  - code
  - architecture
  - urban planning
  - statistics
show_overlay_excerpt: false
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

This is the first time I'm taking a blog article and translating it into Portuguese. I did this because I consider the reasons listed to be very relevant and can serve as a basis for debate. Here is just the translated version, if you want to read the original, [click here](https://blog.codinghorror.com/why-ruby/).

É a primeira vez que estou pegando um artigo de blog e traduzindo para português. Fiz isso porque considero as razões elencadas muito pertinentes e podem servir como base de debates. Aqui está apenas a versão traduzida, se quiser ler o original, [clique aqui](https://blog.codinghorror.com/why-ruby/).

Com a palavra, o sr. Atwood do CodingHorror:

**Por que Ruby?**

Tenho sido desenvolvedor Microsoft por algumas décadas até hoje. Eu dei meus primeiros passos em vários sabores do [Microsoft Basic para PC](http://www.codinghorror.com/blog/2008/04/everything-i-needed-to-know-about-programming-i-learned-from-basic.html), e recebi meu primeiro pagamento programando em Microsoft FoxPro, Microsoft Access, e Microsoft Visual Basic.Eu vi o futuro da programação, meus amigos, e ele é terriveis aplicações de CRUD rodando em maquinas [Wintel](http://pt.wikipedia.org/wiki/Wintel)!

É claro, construimos o Stack Overflow em Microsoft.NET. Este é o grande motivo de ele ainda ser tão rápido quanto é. Então uma das perguntas mais frequentemente feitas depois que anunciamos o [Discourse](http://www.codinghorror.com/blog/2013/02/civilized-discourse-construction-kit.html) foi:

```
Por que vocês não desenvolveram o Discourse em .NET também?
```

Deixemos claro uma coisa: Eu amo .NET. Uma das grandes emoções da minha carreira profissional foi ter a oportunidade de colocar um adesivo do Coding Horror nas mãos de [Anders Hejlsberg](http://en.wikipedia.org/wiki/Anders_Hejlsberg). Perdoe meu fanboy interior por um momento, mas caramba, eu ainda me arrepio. Devem ter talvez uns cinquenta projetistas de linguagem de classes de computador pelo mundo. Anders é apenas um deles que criou o Turbo Pascal e o Delphi. É graças à experiência de Anders que o .NET começou com uma linguagem tão bem projetada – literalmente o que o Java deveria ter sido em todo o nível concebível – e tem evoluído extraordinariamente de modo continuo em meios práticos pelos últimos 10 anos, aproveitando os pontos fortes de outras linguagens dinamicamente influentes.

![image-center](/assets/images/2013/por-que-ruby1.png){: .align-center}

Dito tudo isso, é verdade que eu intencionalmente escolhi não usar .NET para o meu próximo projeto. Então você pode esperar encontrar alguém bravo arengando aqui sobre o quão mais feliz estou deixando as amarras opressoras dos meus senhores Microsoft para trás. Livre afinal, livre, pelo menos, graças a Deus todo-poderoso eu sou livre enfim!

Sinto muito. [Já escrevi este post cinco anos atrás](http://www.codinghorror.com/blog/2007/05/giving-up-on-microsoft.html).

Como qualquer programador pragmático, eu escolhi a ferramenta apropriada para o trabalho em mãos. E por mais que eu ame o .NET, ele seria uma escolha extraordinariamente pobre para um projeto 100% software livre como o Discourse. Por que? Três razões em especial:

1 O licenciamento. Meu Deus, o licenciamento. Nem é tanto pelo dinheiro, como é infernal o alucinante nível de complexidade em ter certeza de que todo o seu software é licenciado apropriadamente: determinando em qual nível e edição você está licenciado, quem é licenciado para usar o que, qual servidores são licenciados… Han? Que? Desculpa, passei lá por um minuto, quando fui atacado por [doninhas raivosas do licenciamento](http://www.codinghorror.com/blog/2009/07/oh-you-wanted-awesome-edition.html).

Eu não estou inclinado a fazer grandes declarações sobre o futuro do software, mas se há algo que vai mata o software comercial, deixe-me dizer-lhe, não será um software de código aberto.

2 O atrito. Se você quer construir um software open source verdadeiramente viável, você precisa de pessoas para contribuir com o seu projeto, de modo que se trate de uma coisa vivente, respirando e crescendo. E a não ser que você possa baixar todo o software que precisa para hackear o seu projeto de forma livre por toda a Internet, sem amarras, haverá atrito demais.

Se Stack Overflow me ensinou algo, é que hoje nós vivemos num mundo onde o próximo engenheiro de software brilhante pode vir de qualquer lugar do planeta. Estou falando de lugares que esse [programador americano e feio](http://www.codinghorror.com/blog/2009/03/the-ugly-american-programmer.html) nunca escutou falar, onde se fala línguas doidas e sem sentido que eu não entendo. Mas entenda. Afaste-se um pouco enquanto eu explodo seus cérebros, gente: esses programadores brilhantes ainda desenvolvem usando as mesmas palavras-chave que nós! Eu sei, louco né?

Levantar e rodar em um ambiente Microsoft é simplesmente muito difícil para um desenvolvedor, digamos, na Argentina, ou no Nepal, ou na Bulgária. Cadeias de sistemas Operacionais, linguagens e ferramentas livres são os grandes equalizadores, a base para a nova grande geração de programadores por todo o mundo onde estão em vias de nos ajudar a mudar o mundo.

3 O Ecosistema. Quando eu estava no [Stack Exchange](http://blog.stackoverflow.com/2012/02/stack-exchange-open-source-projects/), nós brigamos para fazer o máximo que podiamos para que a nossa infraestrutura fosse open source. Foi algo que deixamos explícito nas nossas orientações de compensação, essa ideia de que nós seriamos (parcialmente) julgados pelo quanto poderíamos fazer em público, e tentar deixar o quanto de artefatos públicos e úteis do nosso trabalho quanto podíamos. Mas por que todo o Stack Exchange não foi criado sob [Creative Commons contributions](http://blog.stackoverflow.com/2009/06/stack-overflow-creative-commons-data-dump/) desde o primeiro dia?

Você pode certamente desenvolver software código aberto em .Net. E muitos o fazem. Mas isto nunca parece natural. Isto nunca parece correto. Ninguém aceita uma correção para uma biblioteca de classe do core do .Net, não importa o quanto você tente. parece que você está nadando contra a corrente, em um mundo de pequenas e grandes empresas usando .Net onde elas não estão de fato interessadas em compartilhar o códigos com o mundo – provavelmente porque eles sabem que seriam ruim se o fizessem de qualquer forma. É que não faz parte da cultura do Microsoft .Net fazer coisas open source, especialmente coisas ruins. Se você tem medo de que as coisas que você compartilha sejam ruins, este medo vai te tornar incapaz de realmente e profundamente retribuir. O aspecto mais, digamos delicioso… das comunidades de código aberto é como eles não têm medo de deixar “tudo para fora”, por assim dizer.

Então como resultado, para qualquer tarefa em .Net você deve ter – se tiver sorte – de escolher entre talvez duas bibliotecas talvez decentes. Por outro lado, em qualquer linguagem open source popular, você terá facilmente uma dúzia de alternativas para a mesma função. Sim, talvez seis delas quebrem, seja obsoletas, não funcionem, ou sejam absolutamente loucas. Mas ei, mesmo levando em conta algumas deteriorações naturais de fonte aberta, você ainda está à frente por um fator de três! Você que sai ganhando!

Eu escrevi há [cinco anos atrás](http://www.codinghorror.com/blog/2007/05/giving-up-on-microsoft.html):

"Eu sou pragmático. No momento, eu escolho viver no universo Microsoft. Mas isso não significa que sou ignorante de como os outros vivem. Sempre há mais de um jeito de fazer, e só porque eu escolhi uma em particular não quer dizer que seja a certa – nem mesmo uma maneira particularmente boa. Escolher ser provincial e insular é um caminho infalível para a ignorância. aprenda como os outros vivem. Tente conhecer alguns desenvolvedores que não vivem exatamente no mesmo mundo que você. Procure as ferramentas que estão usando, e porque. Se depois de molhar o seu pé nos dois lados da cerca, você decidir que o outro lado está vivendo melhor e deseja se juntar a eles, então te desejo um amoroso adeus."

Eu não vivo mais no universo Microsoft. Certo, errado, bom, mau, é assim que ele saiu para o projeto que queríamos construir.

![image-center](/assets/images/2013/por-que-ruby2.png){: .align-center}

Contudo, eu estaria mentindo se não falasse que eu realmente acredito que o tipo de projeto que estamos construindo no Discourse representa sim o futuro do software. Se você olhar de soslaio um pouco, eu acho que você pode ver um futuro não muito distante onde o .Net é um nicho especializado fora do mainstream.

Mas por que Ruby? Bem, a menor resposta e não muito glamorosa é a de que eu reduzi a lista a Python ou Ruby, e meu co-fundador original [Robin Ward](http://eviltrout.com/) vem construindo grandes aplicações em Rails desde 2006. Então isso me conquistou.

Eu sempre fui um pouco intrigado com o Ruby, principalmente por causa do [absolutamente transbordante louvor que Steve Yegge fez com a linguagem muito tempo atrás em 2006](https://sites.google.com/site/steveyegge2/tour-de-babel). Eu nunca esqueci disso.

"Na maior parte, Ruby pegou o processamento de string e integração com o Unix do Perl, o que significa que a sintaxe é idêntica, e assim por ali mesmo, antes de qualquer coisa acontecer, você já tem o melhor de Perl. E isso é um grande começo, especialmente se você não pega o resto do Perl.

Mas então Matz pegou o melhor de processamento de listas do Lisp, o melhor de Orientação a Objetos do Smalltalk e outras linguagens, e o melhor de iterators do CLU, e basicamente o melhor de tudo de todos.

E ele de alguma forma fez com que tudo funcionasse junto tão bem que você nem percebe que é tudo isso. Eu aprendi Ruby mais rápido do que qualquer outra linguagem, de umas 30 ou 40 no total: levou uns 3 dias para que eu me sentisse muito mais confortável usando Ruby, do que em Perl, depois de oito anos de codificação em Perl. Ele é tão consistente que você começa a ser capaz de adivinhar como as coisas vão funcionar, e na maioria das vezes você está certo. É lindo. E divertido. E prático."

Steve é um desses programadores poliglotas que eu [respeito tanto](http://www.codinghorror.com/blog/2012/07/but-you-did-not-persuade-me.html) que eu basicamente só tomo seja qual for a sua opinião, desde que não se trate de algo maluco como o controle de armas ou feminismo ou T’Pau, e aceito como fato.

Peço desculpas, Steve. Sinto muito que me levou 7 anos para dar a volta a Ruby. Mas talvez era melhor esperar um tempo de qualquer maneira:

* Ruby tem uma performance decente, mas você realmente precisa jogar uma maquina rápida para ele para ter um bom desempenho. Sim, eu sei, [linguagens interpretadas são o que são](http://www.codinghorror.com/blog/2006/02/the-day-performance-didnt-matter-any-more.html), e cache, banco de dados, rede, blá blá blá. Ainda assim, obtivemos as maquinas mais rápidas que você pode comprar para os servidores Discourse, 4.0 Ghz Ivy Bridge Xeons, e a performance é apenas … boa no hardware mais rápido de hoje. Não muito bom. Bom.

Sim, eu vou admitir que estou completamente mimado pela performance do JIT compilado do .Net. É o que eu estou acostumado. Eu faço às vezes definhar para os maus velhos tempos de. NET quando poderíamos construir páginas que servem em bem menos de 50 milissegundos sem pensar muito. Linguagens interpretadas não vão ser capazes de alcançar esses níveis de desempenho. Mas eu só posso imaginar como o desempenho bruto Ruby tinha que estar de volta na idade das trevas de 2006, quando CPUs e servidores eram cinco vezes mais lento do que são hoje! Estou muito feliz que eu estou atacando de Ruby agora, com o forte vento de muitos anos sólidos da lei de Moore em nossas costas.

- Ruby está amadurecendo muito bem na [versão 2.0 da linguagem](http://www.ruby-lang.org/en/news/2013/02/24/ruby-2-0-0-p0-is-released/?ref=blog.codinghorror.com), o que não aconteceu não mais que um mês depois do Discourse foi anunciado. Então, sim, a desvantagem é que Ruby é lento. Mas o lado positivo é que há um monte de Gems de baixa performance na Rubyland. No Discourse nós temos uma melhoria de desempenho de 20% apenas atualizando para o Ruby 2.0, e nós quase dobramos o nosso desempenho, [aumentando o limite de garbage collector padrão do Ruby](http://meta.discourse.org/t/tuning-ruby-and-rails-for-discourse/4126). De uma perspectiva de desempenho futuro, Ruby é nada mas vantagem.

- Ruby não é mais cool. Sim, você me escutou. Não é mais legal escrever código Ruby. Todas as pessoas legais se mudaram para Scala e Node.js anos atrás. Nosso projeto não é legal, é só um monte do código Ruby velho e chato. Pessoalmente, estou muito feliz que o Ruby agora é maduro o suficiente para que a comunidade não precise mais se preocupar com a pretensão de ser o garoto mais legal do bloco. Isto significa que o resto de nós que gosta apenas de deixar as coisas prontas (Get Shit Done) pode arregaçar as mangas e se concentrar na missão de construir coisas com os nossos pares, em vez de correr freneticamente tentando [desvendar a próxima coisa brilhante](http://www.codinghorror.com/blog/2008/01/the-magpie-developer.html).

E é claro que a comunida Ruby é e sempre foi, incrível. Nós nunca queremos para grandes gemas de código aberto e grandes contribuidores de código aberto. Agora é um momento fantástico para começar com o Ruby, na minha opinião, seja qual for o seu background.

(No entanto, é também importante ressaltar que o Discourse é, se alguma coisa, ainda mais um projeto JavaScript do que um projeto Ruby on Rails. Você não acredita em mim? Basta ir ao try.discourse.org e ver o código fonte. Um fórum de Discourse não é tanto de um website, mas uma aplicação bem desenvolvida em JavaScript que acontece de ser executado em seu navegador.)

Mesmo que sendo feito de boa vontade e para os melhores interesses do projeto, ainda é um pouco assustador mudar totalmente sua espécie de programação da noite para o dia depois de duas décadas. Eu sempre acreditei que os grandes programadores aprender a amar mais de uma linguagem e ambiente de programação – e espero que o projeto Discourse seja uma oportunidade para que todos possam aprender e crescer, não só comigo. Então vai e faz um [fork no GitHub](https://github.com/discourse/discourse) já!
