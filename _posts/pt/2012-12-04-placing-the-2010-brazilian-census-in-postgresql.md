---
title: "Importando o Censo Demográfico 2010 do IBGE no PostgreSQL"
lang: pt
last_modified_at: 2012-12-04T16:00:00-03:00
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

xistem várias razões para querer guardar os dados do Censo Universo de 2010 do IBGE no banco de dados Postgres. Uma que é mais óbvia é poder gerar querys optimizadas dos dados que quiser trabalhar. Além disso o SPSS e o ArcGIS tem opção de abrir uma query do Postgres e muitos outros sistemas também tem. Este procedimento todo deveria ser muito simples, mas a quantidade de erros nos arquivos baixados do IBGE é tão grande que esse processo me tomou um tempo absurdo e sem motivo.

Então vamos lá. Primeira coisa a fazer é baixar todos os dados do FTP do IBGE em ftp://ftp.ibge.gov.br/Censos/Censo_Demografico_2010/Resultados_do_Universo/Agregados_por_Setores_Censitarios/. Prepare tempo e banda pois são vários arquivos e bem grandes (28 arquivos totalizando mais de 2,2Gb).

Mas o problema não está no download. Primeiro, o IBGE disponibiliza os arquivos em dois formatos: XLS e CSV. O problema é que a conversão do XLS para CSV deles não saiu correta. Alguns CSV vieram com a coluna do código do setor em formato de notação científica, além disso devem ter se confundindo no arquivo do Espirito Santo que não colocaram um dos CSV e substituiram por um XLS… Ou seja, não podemos contar com os arquivos CSV salvos pelo IBGE. Outro erro que encontrei depois na hora de jogar os dados dentro das tabelas foi que os arquivos não estão padronizados!!! Por exemplo, os arquivos de Entorno para os estados do Ceará, Distrito Federal, Minas Gerais, Pernambuco e Rio Grande do Sul estão com menos colunas que os outros. Voltaremos a isso mais abaixo.

Outra questão a se pensar é no dicionário das variáveis. Cada estado tem 26 tabelas. E o dicionário de cada tabela está num arquivo PDF dentro de cada ZIP chamado BASE DE INFORMAÇÕES POR SETOR CENSITÁRIO Censo 2010 – Universo.pdf. Um formato nada interessante pois fica muito dificil de tirar os dados de modo automatizado. A solução que encontrei foi converter o PDF em DOC usando este site. E depois tive um trabalho braçal de converter cada um em uma [tabela XLS](/assets/downloads/DadosCenso2010.xls). Coloquei o arquivo disponível para quem não quiser ter o trabalho que tive.

Voltando novamente aos dados. Quem usa Linux sai em vantagem pois ele possui vários comandos que possibilitam executar uma conversão em lote. Abaixo um script shell para rodar e transformar os ZIPs que você baixou do site em um arquivo único para cada uma das 26 tabelas. Antes de rodar esse script é importante que alguns programas estejam instalados. Para isso basta rodar:

sudo apt-get install unoconv libreoffice unzip

E o script:

```bash
#!/bin/sh

mkdir xls
for i in $(ls *.zip)
do
  unzip -j $i -x *.csv *.pdf -d xls
done
cd xls
unoconv -i 59,34,UTF-8 -f csv *
cd ..

for i in $(ls ./xls/*.csv | grep -v Basico)
do
  lc=`echo $i | tr '[A-Z]' '[a-z]'`
  sed '1d' $i | sed 's/,/./g' | sed 's/X//g' > $lc.csv
done

for i in $(ls ./xls/*.csv | grep Basico)
do
  lc=`echo $i | tr '[A-Z]' '[a-z]'`
  sed '1d' $i | sed 's/,/./g' > $lc.csv
done

cat ./xls/basico*.csv.csv > Basico.csv
cat ./xls/domicilio01*.csv.csv > Domicilio01.csv
cat ./xls/domicilio02*.csv.csv > Domicilio02.csv
cat ./xls/domiciliorenda*.csv.csv > DomicilioRenda.csv
cat ./xls/entorno01*.csv.csv > Entorno_01.csv
cat ./xls/entorno02*.csv.csv > Entorno_02.csv
cat ./xls/entorno03*.csv.csv > Entorno_03.csv
cat ./xls/entorno04*.csv.csv > Entorno_04.csv
cat ./xls/entorno05*.csv.csv > Entorno_05.csv
cat ./xls/pessoa01*.csv.csv > Pessoa01.csv
cat ./xls/pessoa02*.csv.csv > Pessoa02.csv
cat ./xls/pessoa03*.csv.csv > Pessoa03.csv
cat ./xls/pessoa04*.csv.csv > Pessoa04.csv
cat ./xls/pessoa05*.csv.csv > Pessoa05.csv
cat ./xls/pessoa06*.csv.csv > Pessoa06.csv
cat ./xls/pessoa07*.csv.csv > Pessoa07.csv
cat ./xls/pessoa08*.csv.csv > Pessoa08.csv
cat ./xls/pessoa09*.csv.csv > Pessoa09.csv
cat ./xls/pessoa10*.csv.csv > Pessoa10.csv
cat ./xls/pessoa11*.csv.csv > Pessoa11.csv
cat ./xls/pessoa12*.csv.csv > Pessoa12.csv
cat ./xls/pessoa13*.csv.csv > Pessoa13.csv
cat ./xls/pessoarenda*.csv.csv > PessoaRenda.csv
cat ./xls/responsavel01*.csv.csv > Responsavel01.csv
cat ./xls/responsavel02*.csv.csv > Responsavel02.csv
cat ./xls/responsavelrenda*.csv.csv > ResponsavelRenda.csv
```

Basta colocar todos os arquivos ZIP numa pasta juntamente com esse script e executar. A linha 02 cria uma pasta. Depois, nas linhas 03 a 06 são descompactados todos os XLS nessa pasta. A linha 08 faz a conversão de todos os arquivos XLS para CSV usando delimitador ; (cujo código é 59) e delimitador de texto ” (cujo código é 34) e ainda usando o encoding UTF-8.

Finalmente as linhas subsequentes usam o comando sed duas vezes, a primeira para tirar a primeira linha de todos os arquivos e depois para substituir a virgula por ponto nos dados e junta tudo num arquivo só.

Pronto. Os arquivos estão prontos para jogar no banco. Depois criei um pequeno script em PHP que converte o XLS com a tabela XLS que falei anteriormente que vai gerar um SQL pra executar no PostgreSQL:


```php
passthru("unoconv -i 59,34,UTF-8 -f csv DadosCenso2010.xls");

$dic = file('DadosCenso2010.csv');
$sql_bd = Array();
$sql_banco = '';
$sql_populate = '';
$tit = false;
$path = '/home/artz/censo/dados/';

$sql_dic = << CREATE TABLE dicionario
(
  cod serial,
  banco character varying,
  variavel character varying,
  descricao character varying
);

SQL_DIC;
$i = 0;
foreach($dic as $v) {
  $v = explode(';', $v);

  if($tit == true) {
    if($v[0] != '') {
      if(count($sql_bd) > 0) {
        $sql_banco .= "CREATE TABLE " . $titulo . "_2010_SC (\n"
          . implode(",\n", $sql_bd)
          . ");\n\n";
        $sql_populate .= "COPY " . $titulo . "_2010_SC FROM '" . $path . $titulo . ".csv' DELIMITERS ';' CSV;\n";
        $sql_bd = Array();
        echo "Creating $titulo...\n";
      }
      $titulo = str_replace(' ', '_', tiracento($v[0]));
      $desc = $v[1];
      $tit = false;
    }
  } else {
    if($v[0] == '') {
      $tit = true;
    } else {
      $v[0] = str_replace(' ', '_', tiracento($v[0]));
      $sql_dic .= "INSERT INTO dicionario (banco, variavel, descricao) VALUES ('" . $titulo . "_2010_SC', '" .$v[0] . "', '" . $v[1] . "');\n";
      $sql_bd[] = " " . $v[0] . ($v[2]==1 ? " character varying" : " numeric");
      $tit = false;
    }
  }
}

if(count($sql_bd) > 0) {
  $sql_banco .= "CREATE TABLE " . $titulo . "_2010_SC (\n"
  . implode(",\n", $sql_bd)
  . ");\n\n";
  $sql_populate .= "COPY " . $titulo . "_2010_SC FROM '" . $path . $titulo . ".csv' DELIMITERS ';' CSV;\n";
  $sql_bd = Array();
  echo "Creating $titulo...\n";
}

function tiracento($texto){
  $trocarIsso = array('à','á','â','ã','ä','å','ç','è','é','ê','ë','ì','í','î','ï','ñ','ò','ó','ô','õ','ö','ù','ü','ú','ÿ','À','Á','Â','Ã','Ä','Å','Ç','È','É','Ê','Ë','Ì','Í','Î','Ï','Ñ','Ò','Ó','Ô','Õ','Ö','O','Ù','Ü','Ú','Ÿ',);
  $porIsso = array('a','a','a','a','a','a','c','e','e','e','e','i','i','i','i','n','o','o','o','o','o','u','u','u','y','A','A','A','A','A','A','C','E','E','E','E','I','I','I','I','N','O','O','O','O','O','0','U','U','U','Y',);
  $titletext = str_replace($trocarIsso, $porIsso, $texto);
  return $titletext;
}

$f = fopen('out.sql', 'w');
fwrite($f, $sql_dic . "\n\n" . $sql_banco . "\n\n" . $sql_populate);
?>
```

Garanto que o procedimento não é tão simples quanto parece. Mas é pelo menos uma pequena ajuda.
