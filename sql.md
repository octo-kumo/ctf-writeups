# SQL Database Version Check Cheat Sheet

```sql
-- MySQL/MariaDB
SELECT VERSION();
-- PostgreSQL
SELECT version();
-- Microsoft SQL Server
SELECT @@VERSION;
-- Oracle Database
SELECT * FROM v$version;
-- SQLite
SELECT sqlite_version();
-- DB2
SELECT service_level, fixpack_num FROM TABLE (sysproc.env_get_inst_info());
-- Informix
SELECT DBINFO('version', 'full');
-- Sybase ASE
SELECT @@version;
