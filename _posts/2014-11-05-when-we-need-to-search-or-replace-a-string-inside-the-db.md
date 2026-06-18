---
title: "When we need to search or replace a string inside the DB"
lang: en
last_modified_at: 2014-08-17T20:00:00-03:00
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

What about when I need to find a specific string within a database but I don't know in which table or column?

Today I had a need that I keep having and I always used another way to solve it. I had never used this bank before, but they came to me asking to change the contract number to, say, '666/2014' where before it was '333/2012'. How to make? Normally I dump the database and search the text in text mode (using an editor or the grep command). But talking to my friend DBA (thanks Adolfho Lopes!) we decided to do it another way.

Below is the result, but be careful: Use in moderation! These functions are to be used in times of need and in a hurry. Never in production. Don't do that, otherwise the bank would collapse! :)

```sql
CREATE TYPE table_master_search_table AS (
    tname varchar,
    cname varchar,
    found varchar
);

CREATE OR REPLACE FUNCTION master_search_table(p_search_string varchar, p_tablecatalog varchar, p_tableschema varchar, p_tablename varchar)
RETURNS SETOF table_master_search_table AS
$$
DECLARE r record;
   r2 table_master_search_table%rowtype;
   t1 table_master_search_table;
   command varchar;
BEGIN
  FOR r IN SELECT table_name, column_name, data_type from information_schema.columns
	where table_catalog = p_tablecatalog and table_schema = p_tableschema and table_name = p_tablename LOOP
	command := 'SELECT '''|| r.table_name || ''' tname, '''|| r.column_name || ''' cname, ' || r.column_name|| ' found FROM ' || r.table_name || ' WHERE cast(' || r.column_name || ' as varchar) like ''%' || p_search_string || '%''';
	raise notice '%',command;
    FOR r2 IN EXECUTE command LOOP
     return NEXT r2;
    END LOOP;

  END LOOP;
END
$$ LANGUAGE plpgsql;

CREATE TYPE table_master_search AS (tname varchar, quant integer);
CREATE OR REPLACE FUNCTION master_search(p_search_string varchar, p_tablecatalog varchar, p_tableschema varchar)
RETURNS SETOF table_master_search AS
$$
declare
  num int:=1;
  col record;
  tab record;
  comando varchar;
  r table_master_search%rowtype;
BEGIN
  FOR tab in select table_name from information_schema.tables where table_schema=p_tableschema and table_type = 'BASE TABLE' LOOP
   comando = 'select '''|| tab.table_name || ''' tname, count(1) quant from '||tab.table_name||' where ';
   FOR col in select count(1) over (partition by table_name) numcols, row_number() over (partition by table_name) seq, * from information_schema.columns
	where table_schema = p_tableschema and table_name = tab.table_name LOOP
       comando :=  comando || 'cast(' || col.column_name ||' as varchar) like ''%' || p_search_string ||'%''';
       IF col.seq <> col.numcols then
         comando :=  comando || ' or ';
       END IF;
   END LOOP;
   RAISE notice '%',comando;
   EXECUTE comando into r;
   IF r.quant > 0 then
     RETURN next r;
   END IF;
  END LOOP;
END
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION master_search_full(p_search_string varchar, p_tablecatalog varchar, p_tableschema varchar)
RETURNS SETOF table_master_search_table AS
$$
declare
  num int:=1;
  col record;
  tab record;
  comando varchar;
  r table_master_search_table%rowtype;
BEGIN
  FOR tab IN SELECT * FROM master_search(p_search_string, p_tablecatalog, p_tableschema) LOOP
    FOR r IN SELECT * from master_search_table(p_search_string, p_tablecatalog, p_tableschema, tab.tname) LOOP
     RETURN next r;
    END LOOP;
  END LOOP;
END
$$ LANGUAGE plpgsql;

select * from master_search_table('string', 'catalogo', 'schema', 'usuarios');
select * from master_search('string', 'catalogo', 'schema');
select * from master_search_full('string', 'catalogo', 'schema');
```