psycopg2.ProgrammingError: permission denied for schema pg_catalog



2024-02-12 09:25:50 UTC:172.31.1.51(39500):development@warehouse_development:[31136]:ERROR: permission denied for schema pg_catalog at character 14
2024-02-12 09:25:50 UTC:172.31.1.51(39500):development@warehouse_development:[31136]:STATEMENT:
CREATE TABLE "engagements_moveout" ("id" serial NOT NULL PRIMARY KEY, "created" timestamp with time zone NOT NULL, "modified" timestamp with time zone NOT NULL, "comment" text NOT NULL)

крч по умолчанию питон не выставляет схему, в которой он должен создавать таблицы. 
дефолтная схема оказывается смотрится в search_path 


[...] tables are often referred to by unqualified names, which consist of just the table name. The system determines which table is meant by following a search path, which is a list of schemas to look in.

warehouse_development=> show search_path ;
         search_path
------------------------------
 pg_catalog, public, topology
 
 
 https://stackoverflow.com/questions/38944551/steps-to-troubleshoot-django-db-utils-programmingerror-permission-denied-for-r
 
 выставил вот так 
alter role  development(это логин пользователя) set search_path = public, topoloy, pg_catalog;

стало 
warehouse_development=> show search_path ;
         search_path         
-----------------------------
 public, topoloy, pg_catalog

думаю проблема в том, что используется RDS master admin аккаунт для похода в базу

Так же в 15 версии постгреса нельзя ходить в public схему https://www.cybertec-postgresql.com/en/error-permission-denied-schema-public/

Ещё - каждый extension ставится в каждую базу данных (НЕ СЕРВЕР) отдельно. Поэтому нужно внимательно смотреть, когда разработчики просят добавить экстеншн. Ты идешь под мастер аккаунтом и можешь попаст случайно в не ту дефолтную базу