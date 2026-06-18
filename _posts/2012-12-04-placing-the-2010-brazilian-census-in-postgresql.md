---
title: "Placing the 2010 Brazilian Census in PostgreSQL"
lang: en
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

There are several reasons for wanting to save IBGE 2010 Universe Census data in the Postgres database. One that is more obvious is being able to generate optimized queries of the data you want to work with. Furthermore, SPSS and ArcGIS have the option to open a Postgres query and many other systems also have it. This whole procedure should be very simple, but the number of errors in the files downloaded from IBGE is so great that this process took me an absurd amount of time for no reason.

So let's go. The first thing to do is download all the data from the IBGE FTP at ftp://ftp.ibge.gov.br/Censos/Censo_Demografico_2010/Resultados_do_Universo/Agregados_por_Setores_Censitarios/. Prepare time and bandwidth as there are several files and quite large (28 files totaling more than 2.2Gb).

But the problem is not with the download. First, IBGE makes the files available in two formats: XLS and CSV. The problem is that their conversion from XLS to CSV did not come out correctly. Some CSV came with the sector code column in scientific notation format, and they must have gotten confused in the Espirito Santo file as they did not include one of the CSV and replaced it with an XLS... In other words, we cannot count on the saved CSV files by IBGE. Another error that I found later when entering the data into the tables was that the files are not standardized!!! For example, the surrounding files for the states of Ceará, Distrito Federal, Minas Gerais, Pernambuco and Rio Grande do Sul have fewer columns than the others. We will return to this below.

Another issue to think about is the dictionary of variables. Each state has 26 tables. And the dictionary for each table is in a PDF file inside each ZIP called INFORMATION BASE BY CENSUS SECTOR 2010 Census – Universo.pdf. An uninteresting format as it is very difficult to obtain the data in an automated way. The solution I found was to convert the PDF to DOC using this website. And then I had the legwork of converting each one into an [XLS table](/assets/downloads/DadosCenso2010.xls). I made the file available for anyone who doesn't want to do the work I did.

Returning to the data again. Those who use Linux have an advantage as it has several commands that make it possible to perform a batch conversion. Below is a shell script to run and transform the ZIPs you downloaded from the website into a single file for each of the 26 tables. Before running this script it is important that some programs are installed. To do this, just run:

sudo apt-get install unoconv libreoffice unzip

And the script:

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

Just put all the ZIP files in a folder together with this script and run. Line 02 creates a folder. Then, in lines 03 to 06, all XLS in that folder are unzipped. Line 08 converts all XLS files to CSV using delimiter ; (whose code is 59) and text delimiter ” (whose code is 34) and still using UTF-8 encoding.

Finally, the subsequent lines use the sed command twice, the first to remove the first line from all files and then to replace the comma with a period in the data and join everything in a single file.

Ready. The files are ready to play in the bank. Then I created a small PHP script that converts the XLS with the XLS table that I mentioned earlier that will generate SQL to execute in PostgreSQL:

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

Be assured that the procedure is not that simple. But it's at least a little help.
