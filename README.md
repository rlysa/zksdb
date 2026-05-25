ТАБЛИЦЫ SCOTT
EMP: empno PK номер; ename имя; job должность; mgr руководитель -> EMP.EMPNO; hiredate дата найма; sal зарплата; comm комиссия; deptno отдел -> DEPT.DEPTNO.
DEPT: deptno PK номер отдела; dname название; loc место. SALGRADE: grade класс; losal/hisal границы зарплаты. Связь: e.sal BETWEEN s.losal AND s.hisal.
СЛОВАРЬ И V$
USER_TABLES таблицы; USER_TAB_COLUMNS поля; USER_CONSTRAINTS ограничения; USER_VIEWS представления; USER_INDEXES индексы.
V$SESSION сеансы; V$TRANSACTION транзакции; V$PARAMETER параметры; V$TABLESPACE табл. пространства; V$DATAFILE файлы; V$LOG/V$LOGFILE redo.
ОСНОВНЫЕ КОМАНДЫ
DML: INSERT INTO t(c1,c2) VALUES(v1,v2); UPDATE t SET c=v WHERE условие; DELETE FROM t WHERE условие; COMMIT; ROLLBACK;
DDL: CREATE TABLE t(...); CREATE TABLE t AS SELECT ...; CREATE VIEW v AS SELECT ...; CREATE SEQUENCE s START WITH n INCREMENT BY n; CREATE INDEX i ON t(c);
GRANT: СИСТЕМНЫЕ ПРИВИЛЕГИИ
GRANT CREATE SESSION TO hr;  -- подключение
GRANT CREATE TABLE, CREATE VIEW, CREATE SEQUENCE TO hr;  -- создавать в своей схеме
GRANT CREATE ANY TABLE, CREATE ANY INDEX TO hr;  -- создавать в чужих схемах
GRANT ALTER USER TO hr;  -- менять пароли/параметры пользователей
GRANT SELECT ANY DICTIONARY TO hr;  -- смотреть словарь/V$
GRANT CREATE SESSION TO hr WITH ADMIN OPTION;  -- можно передавать системную привилегию дальше
REVOKE CREATE TABLE FROM hr;
GRANT: ОБЪЕКТНЫЕ ПРИВИЛЕГИИ
GRANT SELECT ON scott.emp TO hr;  GRANT INSERT, DELETE ON scott.dept TO hr;
GRANT UPDATE(dname) ON scott.dept TO hr;  GRANT REFERENCES(deptno) ON scott.dept TO hr;
GRANT SELECT ON scott.emp TO hr WITH GRANT OPTION;  -- можно передавать объектное право дальше
REVOKE SELECT ON scott.emp FROM hr;
РОЛИ
CREATE ROLE test_role [IDENTIFIED BY pass]; GRANT SELECT ON scott.emp TO test_role; GRANT test_role TO hr; SET ROLE test_role IDENTIFIED BY pass; REVOKE test_role FROM hr;
ALTER USER
ALTER USER hr IDENTIFIED BY newpass; ALTER USER hr ACCOUNT LOCK|UNLOCK;
ALTER USER hr DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp QUOTA 10M ON users;
ALTER USER hr IDENTIFIED EXTERNALLY; DROP USER hr CASCADE;
ALTER TABLE
ALTER TABLE emp ADD col NUMBER; ALTER TABLE emp MODIFY col VARCHAR2(30); ALTER TABLE emp DROP COLUMN col;
ALTER TABLE dept ADD CONSTRAINT pk_dept PRIMARY KEY(deptno);
ALTER TABLE emp ADD CONSTRAINT fk_emp_dept FOREIGN KEY(deptno) REFERENCES dept(deptno) [ON DELETE CASCADE];
ALTER TABLE emp ADD CONSTRAINT chk_sal CHECK(sal>0); ALTER TABLE dept MODIFY dname CONSTRAINT nn_dname NOT NULL;
ALTER TABLE emp ENABLE|DISABLE CONSTRAINT fk_emp_dept; ALTER TABLE emp DROP CONSTRAINT fk_emp_dept;
ALTER SYSTEM / ALTER DATABASE
SHOW PARAMETER name; ALTER SYSTEM SET db_cache_size=100M SCOPE=MEMORY|SPFILE|BOTH;
ARCHIVE LOG LIST; SHUTDOWN IMMEDIATE; STARTUP MOUNT; ALTER DATABASE ARCHIVELOG|NOARCHIVELOG; ALTER DATABASE OPEN;
ALTER SYSTEM SWITCH LOGFILE; ALTER DATABASE DATAFILE 4 OFFLINE|ONLINE;
RMAN: BACKUP DATABASE PLUS ARCHIVELOG; RESTORE/RECOVER DATAFILE 4; RESTORE/RECOVER TABLESPACE users;
Data Pump: expdp scott/tiger tables=emp directory=DP_DIR dumpfile=emp.dmp; impdp scott/tiger tables=emp directory=DP_DIR dumpfile=emp.dmp table_exists_action=append;

Билет № 1
Краткий ответ
SELECT выбирает столбцы из таблиц: SELECT список_столбцов FROM таблица WHERE условие ORDER BY столбец. Ограничение столбцов выполняется списком SELECT, строк - WHERE. DISTINCT удаляет строки-дубликаты. ORDER BY выполняет сортировку по возрастанию ASC или убыванию DESC.
Безопасность данных - состояние защищенности от случайного или преднамеренного воздействия. Основные свойства: конфиденциальность, целостность, доступность. Угрозы: НСД, уничтожение, модификация, раскрытие, отказ в обслуживании.
Практика
SELECT deptno
FROM emp
GROUP BY deptno
HAVING COUNT(*) <= 3;

CREATE USER hr IDENTIFIED BY hr;
GRANT CREATE SESSION, CREATE TABLE TO hr;
GRANT CREATE ANY TABLE TO hr;
ALTER USER hr QUOTA UNLIMITED ON USERS;

CONNECT hr/hr
CREATE TABLE hr.t_hr (id NUMBER);
CREATE TABLE scott.t_hr (id NUMBER);
Билет № 2
Краткий ответ
Соединение таблиц выполняется условием WHERE или оператором JOIN. Декартово произведение получается при отсутствии условия соединения. Внешнее соединение возвращает также строки без соответствия. Самосоединение - соединение таблицы с самой собой через псевдонимы.
Принципы безопасности: законность, системность, комплексность, непрерывность, разграничение доступа, минимальные привилегии, персональная ответственность, контроль и аудит.
Практика
SELECT MAX(sal) - AVG(sal) AS diff
FROM emp;

CREATE USER hr IDENTIFIED BY hr;
GRANT CREATE SESSION TO hr;
GRANT SELECT, INSERT, UPDATE, DELETE ON scott.dept TO hr;
Билет № 3
Краткий ответ
Подзапрос - запрос, вложенный в другой оператор SELECT, INSERT, UPDATE или DELETE. Однострочный подзапрос возвращает одну строку и применяется с =, >, <. Многострочный возвращает несколько строк и применяется с IN, ANY, ALL.
Нарушитель ИБ - субъект, совершающий попытку нарушения правил доступа. Нарушители классифицируются по расположению, правам, уровню знаний, мотивации и ресурсам. Модель нарушителя описывает цели, возможности, доступ, средства и сценарии действий.
Практика
SELECT ename, hiredate
FROM emp
WHERE hiredate > (SELECT hiredate FROM emp WHERE ename = 'JONES');

CREATE USER hr IDENTIFIED BY hr;
GRANT CREATE SESSION TO hr;
GRANT UPDATE (dname) ON scott.dept TO hr;
Билет № 4
Краткий ответ
Функции SQL бывают однострочные и групповые. Однострочные выполняются для каждой строки, групповые - для группы строк и возвращают одно значение. TO_CHAR преобразует дату в строку заданного формата.
Механизмы защиты СУБД: идентификация и аутентификация, разграничение доступа, роли и привилегии, аудит, ограничения целостности, шифрование, резервирование и восстановление.
Практика
SELECT ename, TO_CHAR(hiredate, 'DD-MONTH-YYYY', 'NLS_DATE_LANGUAGE=RUSSIAN') AS hiredate
FROM emp;

CREATE USER hr IDENTIFIED BY hr;
GRANT CREATE SESSION, ALTER USER TO hr;

CONNECT hr/hr
ALTER USER scott IDENTIFIED BY tiger;
Билет № 5
Краткий ответ
GROUP BY группирует записи по значениям столбцов. Групповые функции: COUNT, SUM, AVG, MIN, MAX. WHERE ограничивает строки до группировки, HAVING ограничивает сгруппированные данные.
Модели доступа: дискреционная, мандатная, ролевая. В мандатной модели доступ определяется метками безопасности субъекта и объекта; пользователь не может произвольно передавать права.
Практика
SELECT TO_CHAR(hiredate, 'DAY', 'NLS_DATE_LANGUAGE=RUSSIAN') AS day_name,
       COUNT(*) AS emp_count
FROM emp
GROUP BY TO_CHAR(hiredate, 'DAY', 'NLS_DATE_LANGUAGE=RUSSIAN');

CREATE USER hr IDENTIFIED BY hr
DEFAULT TABLESPACE sysaux
QUOTA 10M ON sysaux
ACCOUNT LOCK;
Билет № 6
Краткий ответ
DML - команды манипулирования данными: SELECT, INSERT, UPDATE, DELETE, MERGE. Они изменяют или выбирают строки таблиц и выполняются в рамках транзакций.
При мандатном доступе возникает многоэкземплярность: один и тот же объект может иметь несколько версий с разными метками. Индексы усложняются, потому что поиск должен учитывать метку безопасности и не раскрывать скрытые строки.
Практика
SELECT empno, ename
FROM emp
WHERE mgr IS NULL;

CREATE USER hr IDENTIFIED BY hr
DEFAULT TABLESPACE sysaux
ACCOUNT LOCK;
ALTER USER hr ACCOUNT UNLOCK;
GRANT CREATE SESSION TO hr;
Билет № 7
Краткий ответ
DDL - команды определения данных: CREATE, ALTER, DROP, TRUNCATE, RENAME. ALTER изменяет структуру объекта, TRUNCATE быстро удаляет все строки таблицы, DROP удаляет объект.
Пользователи создаются CREATE USER. Аутентификация бывает по паролю, внешняя через ОС, глобальная, а для администраторов - через парольный файл или ОС-группы.
Практика
SELECT d.deptno, d.dname
FROM dept d
WHERE NOT EXISTS (SELECT 1 FROM emp e WHERE e.deptno = d.deptno);

CREATE USER hr IDENTIFIED BY hr;
GRANT CREATE SESSION, CREATE ANY INDEX TO hr;
GRANT SELECT ON scott.emp TO hr;

CONNECT hr/hr
CREATE INDEX scott.idx_emp_job ON scott.emp(job);
Билет № 8
Краткий ответ
Таблица создается CREATE TABLE с указанием столбцов и типов. Таблица на основании другой создается CREATE TABLE ... AS SELECT. Структура изменяется ALTER TABLE: ADD, MODIFY, DROP COLUMN.
Системные привилегии дают право выполнять действия в базе: CREATE SESSION, CREATE TABLE, ALTER USER. Выдаются GRANT, отменяются REVOKE. WITH ADMIN OPTION позволяет передавать системную привилегию другим.
Практика
SELECT deptno
FROM emp
GROUP BY deptno
HAVING SUM(sal) = (SELECT MAX(SUM(sal)) FROM emp GROUP BY deptno);

ALTER USER sys IDENTIFIED BY admin;
Билет № 9
Краткий ответ
Объекты схемы: таблицы, представления, последовательности, индексы, процедуры, функции, пакеты, триггеры. Представление - сохраненный запрос. Простое представление строится по одной таблице, сложное содержит соединения, группировки или выражения.
Объектные привилегии дают права на конкретный объект: SELECT, INSERT, UPDATE, DELETE, EXECUTE. Выдаются GRANT, отменяются REVOKE. WITH GRANT OPTION позволяет передавать объектные права другим.
Практика
SELECT ename
FROM emp
WHERE TO_CHAR(hiredate, 'Q') = '4';

CREATE USER ops$user IDENTIFIED EXTERNALLY;
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW TO ops$user;
Билет № 10
Краткий ответ
Последовательность - объект схемы, генерирующий уникальные числовые значения. Используются NEXTVAL и CURRVAL; параметры: START WITH, INCREMENT BY, MINVALUE, MAXVALUE, CYCLE, CACHE.
Парольная аутентификация хранит пароли пользователей в виде хэшей в словаре данных. Для администраторов SYSDBA/SYSOPER используется парольный файл или внешняя аутентификация ОС.
Практика
SELECT d.deptno, COUNT(e.empno) AS emp_count
FROM dept d LEFT JOIN emp e ON e.deptno = d.deptno
GROUP BY d.deptno
ORDER BY d.deptno;

CREATE USER os_user IDENTIFIED EXTERNALLY;
GRANT CREATE SESSION TO os_user;
Билет № 11
Краткий ответ
Физическая структура СУБД включает файлы данных, управляющие файлы и журнальные файлы. Пользовательский процесс выполняет клиентское приложение, серверный процесс обслуживает запрос. Экземпляр Oracle состоит из SGA и фоновых процессов.
Роль - именованный набор привилегий. CREATE ROLE создает роль, GRANT выдает привилегии роли и роль пользователю, REVOKE отменяет. Роль с паролем активируется SET ROLE role IDENTIFIED BY password.
Практика
SELECT job, MIN(sal) AS min_sal, MAX(sal) AS max_sal
FROM emp
GROUP BY job
ORDER BY job;

CREATE USER ops$user IDENTIFIED EXTERNALLY;
GRANT CREATE SESSION TO ops$user;
GRANT SELECT ON scott.emp TO ops$user;
Билет № 12
Краткий ответ
Физическая структура базы данных Oracle: файлы данных, управляющие файлы, файлы REDO LOG, архивные журналы, файл параметров и парольный файл.
Аудит - система протоколирования действий. Включается параметром AUDIT_TRAIL, события задаются AUDIT, отключаются NOAUDIT, результаты просматриваются в DBA_AUDIT_TRAIL и связанных представлениях.
Практика
INSERT INTO dept(deptno, dname, loc)
SELECT MIN(deptno) + 5, 'TEST', 'MOSCOW'
FROM dept;

CREATE USER hr IDENTIFIED BY hr;
GRANT CREATE SESSION TO hr;
CREATE VIEW scott.v_emp_cols AS SELECT empno, ename FROM scott.emp;
GRANT SELECT ON scott.v_emp_cols TO hr;
Билет № 13
Краткий ответ
Экземпляр создается и удаляется средствами DBCA или скриптами CREATE DATABASE; удаление выполняется после остановки экземпляра и удаления файлов/служб. База создается CREATE DATABASE.
Ограничения целостности: NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, CHECK. Создаются в CREATE TABLE или ALTER TABLE, удаляются DROP CONSTRAINT, включаются ENABLE, отключаются DISABLE.
Практика
CREATE TABLE emp_20_30 AS
SELECT empno, ename, sal
FROM emp
WHERE deptno IN (20, 30);

CREATE USER hr IDENTIFIED BY hr;
CREATE ROLE test_role;
GRANT SELECT, INSERT ON scott.emp TO test_role;
GRANT test_role TO hr;
CONNECT hr/hr
SELECT * FROM scott.emp;
INSERT INTO scott.emp(empno, ename) VALUES (9999, 'TEST');
CONNECT / AS SYSDBA
REVOKE test_role FROM hr;
Билет № 14
Краткий ответ
Словарь данных - совокупность системных таблиц и представлений с описанием объектов базы. Представления: USER_OBJECTS, USER_TABLES, DBA_USERS, DBA_DATA_FILES, V$SESSION.
PRIMARY KEY задает уникальность и NOT NULL; UNIQUE задает уникальность, но допускает NULL. Для этих ограничений Oracle создает или использует уникальный индекс.
Практика
CREATE TABLE emp_copy
TABLESPACE sysaux
AS SELECT * FROM emp;

CREATE USER hr IDENTIFIED BY hr;
CREATE ROLE test_role IDENTIFIED BY test;
GRANT SELECT ON scott.emp TO test_role;
GRANT test_role TO hr;
CONNECT hr/hr
SET ROLE test_role IDENTIFIED BY test;
SELECT * FROM scott.emp;
Билет № 15
Краткий ответ
Динамические представления производительности V$ показывают состояние экземпляра: V$INSTANCE, V$DATABASE, V$SESSION, V$DATAFILE, V$LOG. Часть доступна на NOMOUNT, часть после MOUNT или OPEN.
FOREIGN KEY обеспечивает ссылочную целостность: значения дочернего столбца должны существовать в родительском ключе или быть NULL. Может иметь ON DELETE CASCADE.
Практика
CREATE VIEW emp_salgrade AS
SELECT e.ename, s.grade
FROM emp e JOIN salgrade s ON e.sal BETWEEN s.losal AND s.hisal;
-- Представление с соединением обычно не является изменяемым.

ALTER TABLE emp ADD CONSTRAINT pk_emp_ename PRIMARY KEY (ename)
USING INDEX TABLESPACE sysaux;
Билет № 16
Краткий ответ
SELECT проходит синтаксический анализ, оптимизацию, выполнение и выборку строк. DML читает нужные блоки, формирует undo и redo, изменяет буферы и фиксируется COMMIT.
Транзакция - логическая единица работы. Управление: COMMIT, ROLLBACK, SAVEPOINT. Явная транзакция управляется пользователем; неявные фиксации происходят при DDL и штатном завершении.
Практика
CREATE UNIQUE INDEX idx_emp_ename_u ON emp(ename);
UPDATE emp SET ename = 'SMITH' WHERE empno = 7499;

ALTER TABLE emp ADD CONSTRAINT pk_emp_empno PRIMARY KEY (empno);
ALTER TABLE emp DISABLE CONSTRAINT pk_emp_empno;
INSERT INTO emp(empno, ename) VALUES (7369, 'DUP');
ALTER TABLE emp ENABLE CONSTRAINT pk_emp_empno;
Билет № 17
Краткий ответ
Запуск экземпляра: NOMOUNT - создан экземпляр, MOUNT - открыты управляющие файлы, OPEN - открыта база. Остановка: SHUTDOWN NORMAL, TRANSACTIONAL, IMMEDIATE, ABORT.
Транзакции физически обеспечиваются undo-сегментами, redo-журналами, блокировками и SCN; согласованное чтение строится по undo.
Практика
CREATE SEQUENCE dept_seq START WITH 95 INCREMENT BY 1 MAXVALUE 99 NOCYCLE;
INSERT INTO dept(deptno, dname, loc) VALUES (dept_seq.NEXTVAL, 'D95', 'MOSCOW');

INSERT INTO emp(empno, ename) VALUES (7369, 'DUP');
ALTER TABLE emp ADD CONSTRAINT pk_emp_empno PRIMARY KEY (empno);
DELETE FROM emp WHERE empno = 7369 AND ename = 'DUP';
ALTER TABLE emp ADD CONSTRAINT pk_emp_empno PRIMARY KEY (empno);
Билет № 18
Краткий ответ
Файл параметров бывает статический PFILE и серверный SPFILE. Значения выводятся SHOW PARAMETER, V$PARAMETER. ALTER SYSTEM меняет параметры в MEMORY, SPFILE или BOTH.
Блокировки защищают данные при транзакциях. Read only транзакция задает снимок данных на момент начала и не допускает DML в этом сеансе.
Практика
SELECT COUNT(*)
FROM dba_constraints
WHERE owner = 'SCOTT' AND table_name = 'EMP';

ALTER TABLE scott.emp ADD CONSTRAINT pk_emp_ename PRIMARY KEY (ename);

SELECT COUNT(*)
FROM dba_constraints
WHERE owner = 'SCOTT' AND table_name = 'EMP';

ALTER TABLE scott.emp ADD CONSTRAINT pk_emp_empno_ename PRIMARY KEY (empno, ename);
INSERT INTO scott.emp(empno, ename) VALUES (7369, 'SMITH');
Билет № 19
Краткий ответ
REDO LOG состоит из журнальных групп и их членов. LGWR записывает redo-записи в текущую группу; при заполнении выполняется log switch. В ARCHIVELOG заполненные журналы архивируются, в NOARCHIVELOG перезаписываются без архивации.
Шифрование на уровне БД выполняется TDE для табличных пространств/столбцов; на уровне сети - средствами Oracle Net, SSL/TLS и настройками SQLNET.
Практика
SELECT DISTINCT object_type
FROM all_objects
WHERE owner = 'SCOTT'
ORDER BY object_type;

ALTER TABLE emp ADD CONSTRAINT fk_emp_dept
FOREIGN KEY (deptno) REFERENCES dept(deptno);
Билет № 20
Краткий ответ
Логическая структура: табличные пространства, сегменты, экстенты, блоки данных. Физическая структура ОС: файлы данных, управляющие файлы, redo-журналы.
Oracle Label Security реализует мандатное разграничение через политики, метки, уровни, группы и категории; доступ определяется сравнением метки пользователя и метки строки.
Практика
SELECT table_name, tablespace_name
FROM dba_tables
WHERE owner = 'SCOTT';

ALTER TABLE dept ADD CONSTRAINT pk_dept PRIMARY KEY (deptno);
INSERT INTO dept(deptno, dname) VALUES ((SELECT deptno FROM dept WHERE ROWNUM = 1), 'DUP');
ALTER TABLE dept DROP CONSTRAINT pk_dept;
Билет № 21
Краткий ответ
Табличное пространство - логическая область хранения, состоящая из файлов данных. Бывает постоянное, временное, undo. Создается CREATE TABLESPACE, удаляется DROP TABLESPACE, переводится OFFLINE/ONLINE или READ ONLY/READ WRITE.
SQL-инъекция - внедрение SQL-кода во входные данные. Типы: классическая, union, error-based, blind, time-based. Защита: bind-переменные, проверка ввода, минимальные права, запрет конкатенации SQL.
Практика
SELECT username, osuser
FROM v$session
WHERE username IS NOT NULL;

ALTER TABLE dept MODIFY dname CONSTRAINT nn_dept_dname NOT NULL;
INSERT INTO dept(deptno, dname) VALUES (99, NULL);
ALTER TABLE dept MODIFY dname NULL;
Билет № 22
Краткий ответ
Временное табличное пространство хранит временные сегменты сортировок и hash-операций. Создается CREATE TEMPORARY TABLESPACE, назначается пользователю ALTER USER ... TEMPORARY TABLESPACE. При повреждении временный файл можно удалить и создать заново.
В ARCHIVELOG возможны горячие копии и восстановление до точки сбоя; плюс - минимальная потеря данных, минус - нужны архивные журналы и больше места.
Практика
SELECT ROUND(SUM(bytes) / 1024 / 1024, 2) AS mb
FROM v$datafile;

ALTER TABLE emp ADD CONSTRAINT fk_emp_dept_cascade
FOREIGN KEY (deptno) REFERENCES dept(deptno)
ON DELETE CASCADE;
Билет № 23
Краткий ответ
Сегменты отката/undo хранят старые версии данных для отката и согласованного чтения. Управление может быть ручным через rollback segments или автоматическим через UNDO tablespace.
В NOARCHIVELOG резервирование выполняется при закрытой базе; плюс - простота и меньше места, минус - восстановление только до последней полной копии.
Практика
SELECT t.name AS tablespace_name, d.name AS datafile_name
FROM v$tablespace t JOIN v$datafile d ON d.ts# = t.ts#;

-- Сеанс 1
UPDATE emp SET sal = sal WHERE empno = 7369;
-- Сеанс 2
UPDATE emp SET sal = sal WHERE empno = 7499;
-- Сеанс 1
UPDATE emp SET sal = sal WHERE empno = 7499;
-- Сеанс 2
UPDATE emp SET sal = sal WHERE empno = 7369;
Билет № 24
Краткий ответ
Временные сегменты создаются для сортировок, группировок и временных таблиц. Индексные сегменты хранят структуру индекса и экстенты индекса.
Полное восстановление открытой базы в ARCHIVELOG после сбоя диска: перевести поврежденный файл OFFLINE, восстановить его из backup, применить archived/online redo, вернуть ONLINE.
Практика
SELECT l.group#, f.member, l.status, l.sequence#
FROM v$log l JOIN v$logfile f ON f.group# = l.group#
ORDER BY l.group#, f.member;

-- SCOTT
UPDATE dept SET dname = dname WHERE deptno = 10;

-- SYS
SELECT s.username
FROM v$transaction t JOIN v$session s ON s.saddr = t.ses_addr;
Билет № 25
Краткий ответ
Табличный сегмент хранит данные таблицы. Виды: обычные, кластерные, секционированные, индекс-организованные, вложенные, временные. Структура: заголовок сегмента, экстенты, блоки данных.
Если табличное пространство потеряно при STARTUP/SHUTDOWN, восстановление выполняется в MOUNT: restore datafile, recover datafile, затем open.
Практика
SELECT sequence#,
       TO_CHAR(first_time, 'DD-MM-YYYY HH24:MI:SS') AS switch_time
FROM v$log_history
ORDER BY sequence#;

-- Сеанс SCOTT
SET TRANSACTION READ ONLY;
SELECT * FROM dept;

-- Второй сеанс
UPDATE dept SET dname = dname || '_1' WHERE deptno = 10;
COMMIT;

-- Первый сеанс
SELECT * FROM dept;
COMMIT;
Билет № 26
Краткий ответ
ARCHIVELOG сохраняет заполненные redo-журналы и позволяет восстановление до сбоя. NOARCHIVELOG не архивирует журналы, поэтому восстановление возможно только до последней холодной копии.
Если backup потерянного табличного пространства отсутствует, восстановление невозможно; объект создают заново или восстанавливают всю базу из имеющихся копий с потерей данных.
Практика
SELECT h.sequence#,
       (SELECT COUNT(*) FROM v$loghist lh WHERE lh.sequence# = h.sequence#) AS changes_count
FROM v$log_history h;

RMAN> SHUTDOWN IMMEDIATE;
RMAN> STARTUP MOUNT;
RMAN> BACKUP DATABASE;
RMAN> ALTER DATABASE OPEN;
RMAN> SHUTDOWN IMMEDIATE;
-- удалить файл SYSTEM
RMAN> STARTUP MOUNT;
RMAN> RESTORE TABLESPACE SYSTEM;
RMAN> RECOVER TABLESPACE SYSTEM;
RMAN> ALTER DATABASE OPEN;
Билет № 27
Краткий ответ
Информация ARCHIVELOG: ARCHIVE LOG LIST, V$DATABASE, V$ARCHIVED_LOG, параметры LOG_ARCHIVE_DEST. Переход: SHUTDOWN IMMEDIATE, STARTUP MOUNT, ALTER DATABASE ARCHIVELOG, ALTER DATABASE OPEN.
Неполное восстановление выполняется UNTIL TIME, UNTIL SCN или UNTIL SEQUENCE и завершается OPEN RESETLOGS.
Практика
SHOW PARAMETER db_
SELECT name, value FROM v$parameter WHERE name LIKE 'db_%';

RMAN> SHUTDOWN IMMEDIATE;
RMAN> STARTUP MOUNT;
RMAN> BACKUP DATABASE;
RMAN> ALTER DATABASE OPEN;
-- повредить datafile USERS
SQL> ALTER DATABASE DATAFILE '<USERS_DATAFILE>' OFFLINE;
RMAN> RESTORE DATAFILE '<USERS_DATAFILE>';
RMAN> RECOVER DATAFILE '<USERS_DATAFILE>';
SQL> ALTER DATABASE DATAFILE '<USERS_DATAFILE>' ONLINE;
Билет № 28
Краткий ответ
NLS-поддержка задает язык, территорию, форматы дат, чисел и валют. На сервере параметры NLS_DATABASE_PARAMETERS и NLS_INSTANCE_PARAMETERS, на клиенте NLS_LANG и NLS_SESSION_PARAMETERS.
Recovery Manager (RMAN) выполняет backup, restore, recover, catalog, validate, duplicate. Достоинства: инкрементальные копии, контроль целостности, автоматизация; недостатки: требуется настройка и знание RMAN.
Практика
SHOW PARAMETER table
SELECT name, value FROM v$parameter WHERE name LIKE '%table%';

RMAN> SHUTDOWN IMMEDIATE;
RMAN> STARTUP MOUNT;
RMAN> BACKUP DATABASE;
RMAN> ALTER DATABASE OPEN;
RMAN> SHUTDOWN IMMEDIATE;
-- удалить datafile USERS
RMAN> STARTUP MOUNT;
SQL> ALTER DATABASE DATAFILE '<USERS_DATAFILE>' OFFLINE;
SQL> ALTER DATABASE OPEN;
RMAN> RESTORE TABLESPACE USERS;
RMAN> RECOVER TABLESPACE USERS;
SQL> ALTER DATABASE DATAFILE '<USERS_DATAFILE>' ONLINE;
Билет № 29
Краткий ответ
На клиенте язык и территория Oracle задаются переменной NLS_LANG, параметрами реестра Windows, настройками среды и ALTER SESSION SET NLS_*.
Логическое резервирование выполняется export/import или Data Pump expdp/impdp; сохраняются объекты схем и данные, а не физические файлы БД.
Практика
ALTER SYSTEM SET db_cache_size = 100M SCOPE=MEMORY;
ALTER SYSTEM SET db_cache_size = 100M SCOPE=BOTH;
-- MEMORY действует до перезапуска; BOTH меняет память и SPFILE. Для PFILE постоянное значение меняют вручную в файле.

RMAN> BACKUP TABLESPACE users, sysaux;
RMAN> BACKUP CURRENT CONTROLFILE;
SQL> ALTER SYSTEM SWITCH LOGFILE;
Билет № 30
Краткий ответ
Сетевая инфраструктура Oracle: listener, Oracle Net, service name, tnsnames.ora, listener.ora, sqlnet.ora. Клиент подключается к сервису БД через сетевой псевдоним или строку подключения.
Средства защиты СУБД входят в подсистему защиты АИС и дополняют ОС, сеть и приложение: аутентификация, авторизация, аудит, шифрование, резервирование и контроль целостности должны согласовываться на всех уровнях.
Практика
ALTER SYSTEM SET shared_pool_size = 100M SCOPE=MEMORY;
ALTER SYSTEM SET shared_pool_size = 100M SCOPE=SPFILE;
-- MEMORY меняет работающий экземпляр; SPFILE вступит в силу после перезапуска.

expdp scott/tiger tables=emp directory=DATA_PUMP_DIR dumpfile=emp.dmp logfile=emp_exp.log

DELETE FROM emp;
COMMIT;

impdp scott/tiger tables=emp directory=DATA_PUMP_DIR dumpfile=emp.dmp logfile=emp_imp.log table_exists_action=append
