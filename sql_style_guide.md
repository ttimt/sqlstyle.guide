# SQL style guide

## Contents

- [Overview](#overview)
- [General](#general)
  - [Do](#do)
  - [Avoid](#avoid)
- [Naming conventions](#naming-conventions)
  - [General](#general-1)
  - [Tables](#tables)
  - [Columns](#columns)
  - [Aliasing or correlations](#aliasing-or-correlations)
  - [Uniform suffixes](#uniform-suffixes)
- [Query syntax](#query-syntax)
  - [Reserved words](#reserved-words)
  - [White space](#white-space)
    - [Spaces](#spaces)
    - [Line spacing](#line-spacing)
  - [Indentation](#indentation)
    - [Joins](#joins)
    - [Subqueries](#subqueries)
  - [Preferred formalisms](#preferred-formalisms)
  - [Common table expressions](#common-table-expressions)
    - [Do](#do-1)
    - [Avoid](#avoid-1)
- [Create syntax](#create-syntax)
  - [Choosing data types](#choosing-data-types)
  - [Constraints and keys](#constraints-and-keys)
- [Appendix](#appendix)
  - [Reserved keyword reference](#reserved-keyword-reference)

## Overview

This guide defines SQL conventions for our analytics team. Pick a style and
stick to it. Consistency within a query matters more than which specific
convention you choose—but following this guide makes code easier to review,
share, and maintain.

SQL style guide by [Simon Holywell][simon] is licensed under a [Creative Commons
Attribution-ShareAlike 4.0 International License][licence].
Based on a work at [https://www.sqlstyle.guide/][sqlstyleguide].

## General

### Do

* Use consistent and descriptive identifiers and names.
* Make judicious use of white space and indentation to make code easier to read.
* Store [ISO 8601][iso-8601] compliant time and date information
  (`YYYY-MM-DD` or `YYYY-MM-DDTHH:MM:SS`).
* Prefer standard SQL functions over vendor-specific ones where possible.
* Keep code succinct—avoid redundant parentheses, unnecessary quoting, or
  `WHERE` clauses that can be derived another way.
* Comment SQL where the intent is not self-evident. Use `--` for single-line
  comments and `/* */` for blocks.

```sql
SELECT user_id,  -- anonymised identifier
       event_date
  FROM events
 WHERE event_type = 'purchase';
```

```sql
/*
  Daily active users: counts distinct users who triggered
  any event on a given day.
*/
SELECT event_date,
       COUNT(DISTINCT user_id) AS dau
  FROM events
 GROUP BY event_date;
```

### Avoid

* `camelCase`—it is harder to scan quickly than `snake_case`.
* Descriptive prefixes or Hungarian notation such as `tbl_` or `v_`.
* Plurals—prefer collective or singular nouns. Use `staff` rather than
  `employees`, `event` rather than `events` where sensible.
* Quoted identifiers unless absolutely necessary. If you must use them, prefer
  SQL-92 double quotes (`"column"`) over back-ticks.
* Applying object-oriented design principles to SQL—they do not translate well
  to relational data.

## Naming conventions

### General

* Names must be unique and must not clash with [reserved keywords][reserved-keywords].
* Keep names to a maximum of 30 characters.
* Begin with a letter; do not end with an underscore.
* Use only letters, numbers and underscores.
* Avoid multiple consecutive underscores—they are hard to read.
* Use underscores where you would naturally use a space (`first_name` not
  `firstname`).
* Avoid abbreviations; when you must abbreviate, use ones that are widely
  understood.

```sql
SELECT customer_id
  FROM orders;
```

### Tables

* Use a collective or singular name (`staff`, `order`, `session`).
* Do not prefix with `tbl`, `vw`, or any other notation.
* Never give a table the same name as one of its columns.
* For relationship or bridge tables, choose a descriptive noun rather than
  concatenating table names (`assignment` rather than `staff_projects`).

### Columns

* Always use the singular name.
* Avoid using `id` as a bare column name—prefer `user_id`, `order_id`, etc.
* Do not add a column with the same name as its table.
* Use lowercase throughout.

### Aliasing or correlations

* Aliases should relate meaningfully to the object or expression they alias.
* As a rule of thumb, use the initial letter of each word in the object name.
* If two aliases would collide, append a number (`s1`, `s2`).
* Always include the `AS` keyword—it makes intent explicit.
* Alias computed columns with the name you would give them in a schema
  definition.

```sql
SELECT customer_id AS cid
  FROM orders AS o1
  JOIN returns AS r1
    ON r1.order_id = o1.order_id;
```
```sql
SELECT SUM(ol.line_revenue) AS revenue_total
  FROM order_lines AS ol;
```

### Uniform suffixes

The following suffixes have a universal meaning ensuring the columns can be read
and understood easily from SQL code. Use the correct suffix where appropriate.

* `_id`—a unique identifier such as a column that is a primary key.
* `_status`—flag value or some other status of any type such as
  `publication_status`.
* `_total`—the total or sum of a collection of values.
* `_num`—denotes the field contains any kind of number.
* `_name`—signifies a name such as `first_name`.
* `_seq`—contains a contiguous sequence of values.
* `_date`—denotes a column that contains the date of something.
* `_at`—a full timestamp such as `created_at` or `updated_at`.
* `_count`—a count of rows or occurrences such as `event_count`.
* `_size`—the size of something such as a file size or clothing.
* `_addr`—an address for the record could be physical or intangible such as
  `ip_addr`.
* `_flag`—a boolean indicator; alternatively use an `is_` prefix such as
  `is_active` or `is_deleted`.

## Query syntax

### Reserved words

Always use uppercase for [reserved keywords][reserved-keywords]
like `SELECT` and `WHERE`.

Prefer the full keyword over abbreviations where both exist (`ABSOLUTE` rather
than `ABS`). Prefer `INNER JOIN` over `JOIN` when the join type matters for
clarity.

Prefer standard ANSI SQL keywords over vendor-specific alternatives when both
produce identical results.

```sql
SELECT user_id,
       event_type
  FROM events AS e
 WHERE e.event_date > '2024-01-01';
```

### White space

To make the code easier to read it is important that the correct complement of
spacing is used. Do not crowd code or remove natural language spaces.

#### Spaces

Spaces should be used to line up the code so that the root keywords all end on
the same character boundary. This forms a river down the middle making it easy for
the readers eye to scan over the code and separate the keywords from the
implementation detail. Rivers are [bad in typography][rivers], but helpful here.

```sql
(SELECT channel,
        COUNT(session_id) AS session_count, SUM(revenue) AS total_revenue
   FROM web_sessions AS w
  WHERE w.channel = 'organic'
     OR w.channel = 'paid_search'
     OR w.channel = 'email'
  GROUP BY channel, event_date)

  UNION ALL

(SELECT channel,
        COUNT(session_id) AS session_count, SUM(revenue) AS total_revenue
   FROM app_sessions AS a
  WHERE a.channel = 'organic'
     OR a.channel = 'paid_search'
     OR a.channel = 'email'
  GROUP BY channel, event_date);
```

Notice that `SELECT`, `FROM`, etc. are all right aligned while the actual column
names and implementation-specific details are left aligned.

Although not exhaustive always include spaces:

* before and after equals (`=`)
* after commas (`,`)
* surrounding apostrophes (`'`) where not within parentheses or with a trailing
  comma or semicolon.

```sql
SELECT o.order_id, o.order_date, o.order_status
  FROM orders AS o
 WHERE o.customer_id = 'C-001'
    OR o.customer_id = 'C-002';
```

#### Line spacing

Always include newlines/vertical space:

* before `AND` or `OR`
* after semicolons to separate queries for easier reading
* after each keyword definition
* after a comma when separating multiple columns into logical groups
* to separate code into related sections, which helps to ease the readability of
  large chunks of code.

Keeping all the keywords aligned to the righthand side and the values left aligned
creates a uniform gap down the middle of the query. It also makes it much easier to
quickly scan over the query definition.

```sql
INSERT INTO orders (customer_id, order_date, order_status)
VALUES ('C-001', '2024-01-15 09:23:00.00000', 'completed'),
       ('C-002', '2024-01-15 11:47:00.00000', 'pending');
```

```sql
UPDATE orders
   SET order_status = 'completed'
 WHERE order_id = 1001;
```

```sql
SELECT o.order_id,
       o.order_date, o.order_status, o.updated_at -- grouped order metadata
  FROM orders AS o
 WHERE o.customer_id = 'C-001'
    OR o.customer_id = 'C-002';
```

### Indentation

To ensure that SQL is readable it is important that standards of indentation
are followed.

#### Joins

Joins should be indented to the other side of the river and grouped with a new
line where necessary.

```sql
SELECT o.order_id
  FROM orders AS o
       INNER JOIN customers AS c
       ON o.customer_id = c.customer_id
          AND c.account_status = 'active'

       INNER JOIN order_lines AS ol
       ON o.order_id = ol.order_id
          AND ol.is_refunded_flag = 'N';
```

The exception to this is when using just the `JOIN` keyword where it should be
before the river.

```sql
SELECT o.order_id
  FROM orders AS o
  JOIN customers AS c
    ON o.customer_id = c.customer_id
```

#### Subqueries

Subqueries should also be aligned to the right side of the river and then laid
out using the same style as any other query. Sometimes it will make sense to have
the closing parenthesis on a new line at the same character position as its
opening partner—this is especially true where you have nested subqueries.

```sql
SELECT c.customer_id,
       (SELECT MAX(order_date)
          FROM orders AS o
         WHERE o.customer_id = c.customer_id
           AND o.order_status = 'completed') AS last_order_date
  FROM customers AS c
 WHERE c.customer_id IN
       (SELECT o.customer_id
          FROM orders AS o
         WHERE o.order_date > '2024-01-01'
           AND o.order_status = 'completed');
```

### Preferred formalisms

* Make use of `BETWEEN` where possible instead of combining multiple statements
  with `AND`.
* Similarly use `IN()` instead of multiple `OR` clauses.
* Where a value needs to be interpreted before leaving the database use the `CASE`
  expression. `CASE` statements can be nested to form more complex logical structures.
* Avoid the use of `UNION` clauses and temporary tables where possible. If the
  schema can be optimised to remove the reliance on these features then it most
  likely should be.
* Use `COALESCE()` to handle `NULL` fallbacks in expressions—it is standard SQL
  and portable across engines.
* Use `IS NULL` and `IS NOT NULL` to test for the presence or absence of a value.
  Never use `= NULL` or `!= NULL`, which produce undefined results in SQL.

```sql
SELECT CASE country_code
            WHEN 'US' THEN 'North America'
            WHEN 'GB' THEN 'EMEA'
            ELSE 'Other'
       END AS region,
       COUNT(*) AS customer_count
  FROM customers
 WHERE is_active_flag = 'Y'
   AND created_at BETWEEN '2024-01-01' AND '2024-12-31'
   AND plan_type IN ('starter', 'growth', 'enterprise');
```

```sql
SELECT COALESCE(revenue_total, 0) AS revenue_total
  FROM daily_summary
 WHERE cohort_date IS NOT NULL;
```

### Common table expressions

Common table expressions (CTEs) make complex analytical queries easier to read by
breaking them into named, sequentially defined steps. Prefer CTEs over deeply
nested subqueries.

#### Do

* Name each CTE to describe what it contains, not how it is computed
  (`daily_revenue` rather than `step_1` or `subquery`).
* Use `WITH` at the top level; list additional CTEs as a comma-separated
  sequence beneath it.
* Place `AS (` on the same line as the CTE name.
* Indent the body of each CTE by four (4) spaces.
* Separate consecutive CTEs with a blank line after the closing `)`.
* Order CTEs so that each one depends only on those defined before it.
* Keep each CTE focused on a single logical step.

#### Avoid

* Generic names (`cte`, `temp`, `data`, `results`).
* Using CTEs where a simple subquery would be clearer.
* Deeply chaining CTEs where later steps reference many earlier ones—restructure
  or split the query instead.

```sql
WITH
daily_revenue AS (
    SELECT order_date,
           SUM(order_total)  AS revenue_total,
           COUNT(order_id)   AS order_count
      FROM orders
     WHERE order_status = 'completed'
     GROUP BY order_date
),

customer_first_order AS (
    SELECT customer_id,
           MIN(order_date) AS first_order_at
      FROM orders
     GROUP BY customer_id
),

daily_new_customers AS (
    SELECT first_order_at        AS order_date,
           COUNT(customer_id)    AS new_customer_count
      FROM customer_first_order
     GROUP BY first_order_at
)

SELECT dr.order_date,
       dr.revenue_total,
       dr.order_count,
       COALESCE(dnc.new_customer_count, 0) AS new_customer_count
  FROM daily_revenue AS dr
  LEFT JOIN daily_new_customers AS dnc
    ON dr.order_date = dnc.order_date
 ORDER BY dr.order_date;
```

## Create syntax

When declaring schema information it is also important to maintain human-readable
code. To facilitate this ensure that the column definitions are ordered and
grouped together where it makes sense to do so.

Indent column definitions by four (4) spaces within the `CREATE` definition.

### Choosing data types

* Where possible do not use vendor-specific data types—these are not portable and
  may not be available in older versions of the same vendor's software.
* Only use `REAL` or `FLOAT` types where it is strictly necessary for floating
  point mathematics otherwise prefer `NUMERIC` and `DECIMAL` at all times. Floating
  point rounding errors are a nuisance!

### Constraints and keys

* Tables must have at least one key to be complete and useful.
* Constraints should be given a custom name excepting `UNIQUE`, `PRIMARY KEY`
  and `FOREIGN KEY` where the database vendor will generally supply sufficiently
  intelligible names automatically.

```sql
CREATE TABLE orders (
    PRIMARY KEY (order_id),
    order_id       BIGINT         NOT NULL,
    customer_id    BIGINT         NOT NULL,
    order_status   VARCHAR(20)    NOT NULL,
    order_total    DECIMAL(10, 2) NOT NULL,
                   CONSTRAINT order_total_positive
                   CHECK(order_total >= 0)
);
```

## Appendix

### Reserved keyword reference

A list of ANSI SQL (92, 99 and 2003), MySQL 3 to 5.x, PostgreSQL 8.1, MS SQL Server 2000, MS ODBC and Oracle 10.2 reserved keywords. Use this as a quick reference when naming tables, columns, or aliases to avoid clashes with reserved words.

```sql
A
ABORT
ABS
ABSOLUTE
ACCESS
ACTION
ADA
ADD
ADMIN
AFTER
AGGREGATE
ALIAS
ALL
ALLOCATE
ALSO
ALTER
ALWAYS
ANALYSE
ANALYZE
AND
ANY
ARE
ARRAY
AS
ASC
ASENSITIVE
ASSERTION
ASSIGNMENT
ASYMMETRIC
AT
ATOMIC
ATTRIBUTE
ATTRIBUTES
AUDIT
AUTHORIZATION
AUTO_INCREMENT
AVG
AVG_ROW_LENGTH
BACKUP
BACKWARD
BEFORE
BEGIN
BERNOULLI
BETWEEN
BIGINT
BINARY
BIT
BIT_LENGTH
BITVAR
BLOB
BOOL
BOOLEAN
BOTH
BREADTH
BREAK
BROWSE
BULK
BY
C
CACHE
CALL
CALLED
CARDINALITY
CASCADE
CASCADED
CASE
CAST
CATALOG
CATALOG_NAME
CEIL
CEILING
CHAIN
CHANGE
CHAR
CHAR_LENGTH
CHARACTER
CHARACTER_LENGTH
CHARACTER_SET_CATALOG
CHARACTER_SET_NAME
CHARACTER_SET_SCHEMA
CHARACTERISTICS
CHARACTERS
CHECK
CHECKED
CHECKPOINT
CHECKSUM
CLASS
CLASS_ORIGIN
CLOB
CLOSE
CLUSTER
CLUSTERED
COALESCE
COBOL
COLLATE
COLLATION
COLLATION_CATALOG
COLLATION_NAME
COLLATION_SCHEMA
COLLECT
COLUMN
COLUMN_NAME
COLUMNS
COMMAND_FUNCTION
COMMAND_FUNCTION_CODE
COMMENT
COMMIT
COMMITTED
COMPLETION
COMPRESS
COMPUTE
CONDITION
CONDITION_NUMBER
CONNECT
CONNECTION
CONNECTION_NAME
CONSTRAINT
CONSTRAINT_CATALOG
CONSTRAINT_NAME
CONSTRAINT_SCHEMA
CONSTRAINTS
CONSTRUCTOR
CONTAINS
CONTAINSTABLE
CONTINUE
CONVERSION
CONVERT
COPY
CORR
CORRESPONDING
COUNT
COVAR_POP
COVAR_SAMP
CREATE
CREATEDB
CREATEROLE
CREATEUSER
CROSS
CSV
CUBE
CUME_DIST
CURRENT
CURRENT_DATE
CURRENT_DEFAULT_TRANSFORM_GROUP
CURRENT_PATH
CURRENT_ROLE
CURRENT_TIME
CURRENT_TIMESTAMP
CURRENT_TRANSFORM_GROUP_FOR_TYPE
CURRENT_USER
CURSOR
CURSOR_NAME
CYCLE
DATA
DATABASE
DATABASES
DATE
DATETIME
DATETIME_INTERVAL_CODE
DATETIME_INTERVAL_PRECISION
DAY
DAY_HOUR
DAY_MICROSECOND
DAY_MINUTE
DAY_SECOND
DAYOFMONTH
DAYOFWEEK
DAYOFYEAR
DBCC
DEALLOCATE
DEC
DECIMAL
DECLARE
DEFAULT
DEFAULTS
DEFERRABLE
DEFERRED
DEFINED
DEFINER
DEGREE
DELAY_KEY_WRITE
DELAYED
DELETE
DELIMITER
DELIMITERS
DENSE_RANK
DENY
DEPTH
DEREF
DERIVED
DESC
DESCRIBE
DESCRIPTOR
DESTROY
DESTRUCTOR
DETERMINISTIC
DIAGNOSTICS
DICTIONARY
DISABLE
DISCONNECT
DISK
DISPATCH
DISTINCT
DISTINCTROW
DISTRIBUTED
DIV
DO
DOMAIN
DOUBLE
DROP
DUAL
DUMMY
DUMP
DYNAMIC
DYNAMIC_FUNCTION
DYNAMIC_FUNCTION_CODE
EACH
ELEMENT
ELSE
ELSEIF
ENABLE
ENCLOSED
ENCODING
ENCRYPTED
END
END-EXEC
ENUM
EQUALS
ERRLVL
ESCAPE
ESCAPED
EVERY
EXCEPT
EXCEPTION
EXCLUDE
EXCLUDING
EXCLUSIVE
EXEC
EXECUTE
EXISTING
EXISTS
EXIT
EXP
EXPLAIN
EXTERNAL
EXTRACT
FALSE
FETCH
FIELDS
FILE
FILLFACTOR
FILTER
FINAL
FIRST
FLOAT
FLOAT4
FLOAT8
FLOOR
FLUSH
FOLLOWING
FOR
FORCE
FOREIGN
FORTRAN
FORWARD
FOUND
FREE
FREETEXT
FREETEXTTABLE
FREEZE
FROM
FULL
FULLTEXT
FUNCTION
FUSION
G
GENERAL
GENERATED
GET
GLOBAL
GO
GOTO
GRANT
GRANTED
GRANTS
GREATEST
GROUP
GROUPING
HANDLER
HAVING
HEADER
HEAP
HIERARCHY
HIGH_PRIORITY
HOLD
HOLDLOCK
HOST
HOSTS
HOUR
HOUR_MICROSECOND
HOUR_MINUTE
HOUR_SECOND
IDENTIFIED
IDENTITY
IDENTITY_INSERT
IDENTITYCOL
IF
IGNORE
ILIKE
IMMEDIATE
IMMUTABLE
IMPLEMENTATION
IMPLICIT
IN
INCLUDE
INCLUDING
INCREMENT
INDEX
INDICATOR
INFILE
INFIX
INHERIT
INHERITS
INITIAL
INITIALIZE
INITIALLY
INNER
INOUT
INPUT
INSENSITIVE
INSERT
INSERT_ID
INSTANCE
INSTANTIABLE
INSTEAD
INT
INT1
INT2
INT3
INT4
INT8
INTEGER
INTERSECT
INTERSECTION
INTERVAL
INTO
INVOKER
IS
ISAM
ISNULL
ISOLATION
ITERATE
JOIN
K
KEY
KEY_MEMBER
KEY_TYPE
KEYS
KILL
LANCOMPILER
LANGUAGE
LARGE
LAST
LAST_INSERT_ID
LATERAL
LEADING
LEAST
LEAVE
LEFT
LENGTH
LESS
LEVEL
LIKE
LIMIT
LINENO
LINES
LISTEN
LN
LOAD
LOCAL
LOCALTIME
LOCALTIMESTAMP
LOCATION
LOCATOR
LOCK
LOGIN
LOGS
LONG
LONGBLOB
LONGTEXT
LOOP
LOW_PRIORITY
LOWER
M
MAP
MATCH
MATCHED
MAX
MAX_ROWS
MAXEXTENTS
MAXVALUE
MEDIUMBLOB
MEDIUMINT
MEDIUMTEXT
MEMBER
MERGE
MESSAGE_LENGTH
MESSAGE_OCTET_LENGTH
MESSAGE_TEXT
METHOD
MIDDLEINT
MIN
MIN_ROWS
MINUS
MINUTE
MINUTE_MICROSECOND
MINUTE_SECOND
MINVALUE
MLSLABEL
MOD
MODE
MODIFIES
MODIFY
MODULE
MONTH
MONTHNAME
MORE
MOVE
MULTISET
MUMPS
MYISAM
NAME
NAMES
NATIONAL
NATURAL
NCHAR
NCLOB
NESTING
NEW
NEXT
NO
NO_WRITE_TO_BINLOG
NOAUDIT
NOCHECK
NOCOMPRESS
NOCREATEDB
NOCREATEROLE
NOCREATEUSER
NOINHERIT
NOLOGIN
NONCLUSTERED
NONE
NORMALIZE
NORMALIZED
NOSUPERUSER
NOT
NOTHING
NOTIFY
NOTNULL
NOWAIT
NULL
NULLABLE
NULLIF
NULLS
NUMBER
NUMERIC
OBJECT
OCTET_LENGTH
OCTETS
OF
OFF
OFFLINE
OFFSET
OFFSETS
OIDS
OLD
ON
ONLINE
ONLY
OPEN
OPENDATASOURCE
OPENQUERY
OPENROWSET
OPENXML
OPERATION
OPERATOR
OPTIMIZE
OPTION
OPTIONALLY
OPTIONS
OR
ORDER
ORDERING
ORDINALITY
OTHERS
OUT
OUTER
OUTFILE
OUTPUT
OVER
OVERLAPS
OVERLAY
OVERRIDING
OWNER
PACK_KEYS
PAD
PARAMETER
PARAMETER_MODE
PARAMETER_NAME
PARAMETER_ORDINAL_POSITION
PARAMETER_SPECIFIC_CATALOG
PARAMETER_SPECIFIC_NAME
PARAMETER_SPECIFIC_SCHEMA
PARAMETERS
PARTIAL
PARTITION
PASCAL
PASSWORD
PATH
PCTFREE
PERCENT
PERCENT_RANK
PERCENTILE_CONT
PERCENTILE_DISC
PLACING
PLAN
PLI
POSITION
POSTFIX
POWER
PRECEDING
PRECISION
PREFIX
PREORDER
PREPARE
PREPARED
PRESERVE
PRIMARY
PRINT
PRIOR
PRIVILEGES
PROC
PROCEDURAL
PROCEDURE
PROCESS
PROCESSLIST
PUBLIC
PURGE
QUOTE
RAID0
RAISERROR
RANGE
RANK
RAW
READ
READS
READTEXT
REAL
RECHECK
RECONFIGURE
RECURSIVE
REF
REFERENCES
REFERENCING
REGEXP
REGR_AVGX
REGR_AVGY
REGR_COUNT
REGR_INTERCEPT
REGR_R2
REGR_SLOPE
REGR_SXX
REGR_SXY
REGR_SYY
REINDEX
RELATIVE
RELEASE
RELOAD
RENAME
REPEAT
REPEATABLE
REPLACE
REPLICATION
REQUIRE
RESET
RESIGNAL
RESOURCE
RESTART
RESTORE
RESTRICT
RESULT
RETURN
RETURNED_CARDINALITY
RETURNED_LENGTH
RETURNED_OCTET_LENGTH
RETURNED_SQLSTATE
RETURNS
REVOKE
RIGHT
RLIKE
ROLE
ROLLBACK
ROLLUP
ROUTINE
ROUTINE_CATALOG
ROUTINE_NAME
ROUTINE_SCHEMA
ROW
ROW_COUNT
ROW_NUMBER
ROWCOUNT
ROWGUIDCOL
ROWID
ROWNUM
ROWS
RULE
SAVE
SAVEPOINT
SCALE
SCHEMA
SCHEMA_NAME
SCHEMAS
SCOPE
SCOPE_CATALOG
SCOPE_NAME
SCOPE_SCHEMA
SCROLL
SEARCH
SECOND
SECOND_MICROSECOND
SECTION
SECURITY
SELECT
SELF
SENSITIVE
SEPARATOR
SEQUENCE
SERIALIZABLE
SERVER_NAME
SESSION
SESSION_USER
SET
SETOF
SETS
SETUSER
SHARE
SHOW
SHUTDOWN
SIGNAL
SIMILAR
SIMPLE
SIZE
SMALLINT
SOME
SONAME
SOURCE
SPACE
SPATIAL
SPECIFIC
SPECIFIC_NAME
SPECIFICTYPE
SQL
SQL_BIG_RESULT
SQL_BIG_SELECTS
SQL_BIG_TABLES
SQL_CALC_FOUND_ROWS
SQL_LOG_OFF
SQL_LOG_UPDATE
SQL_LOW_PRIORITY_UPDATES
SQL_SELECT_LIMIT
SQL_SMALL_RESULT
SQL_WARNINGS
SQLCA
SQLCODE
SQLERROR
SQLEXCEPTION
SQLSTATE
SQLWARNING
SQRT
SSL
STABLE
START
STARTING
STATE
STATEMENT
STATIC
STATISTICS
STATUS
STDDEV_POP
STDDEV_SAMP
STDIN
STDOUT
STORAGE
STRAIGHT_JOIN
STRICT
STRING
STRUCTURE
STYLE
SUBCLASS_ORIGIN
SUBLIST
SUBMULTISET
SUBSTRING
SUCCESSFUL
SUM
SUPERUSER
SYMMETRIC
SYNONYM
SYSDATE
SYSID
SYSTEM
SYSTEM_USER
TABLE
TABLE_NAME
TABLES
TABLESAMPLE
TABLESPACE
TEMP
TEMPLATE
TEMPORARY
TERMINATE
TERMINATED
TEXT
TEXTSIZE
THAN
THEN
TIES
TIME
TIMESTAMP
TIMEZONE_HOUR
TIMEZONE_MINUTE
TINYBLOB
TINYINT
TINYTEXT
TO
TOAST
TOP
TOP_LEVEL_COUNT
TRAILING
TRAN
TRANSACTION
TRANSACTION_ACTIVE
TRANSACTIONS_COMMITTED
TRANSACTIONS_ROLLED_BACK
TRANSFORM
TRANSFORMS
TRANSLATE
TRANSLATION
TREAT
TRIGGER
TRIGGER_CATALOG
TRIGGER_NAME
TRIGGER_SCHEMA
TRIM
TRUE
TRUNCATE
TRUSTED
TSEQUAL
TYPE
UESCAPE
UID
UNBOUNDED
UNCOMMITTED
UNDER
UNDO
UNENCRYPTED
UNION
UNIQUE
UNKNOWN
UNLISTEN
UNLOCK
UNNAMED
UNNEST
UNSIGNED
UNTIL
UPDATE
UPDATETEXT
UPPER
USAGE
USE
USER
USER_DEFINED_TYPE_CATALOG
USER_DEFINED_TYPE_CODE
USER_DEFINED_TYPE_NAME
USER_DEFINED_TYPE_SCHEMA
USING
UTC_DATE
UTC_TIME
UTC_TIMESTAMP
VACUUM
VALID
VALIDATE
VALIDATOR
VALUE
VALUES
VAR_POP
VAR_SAMP
VARBINARY
VARCHAR
VARCHAR2
VARCHARACTER
VARIABLE
VARIABLES
VARYING
VERBOSE
VIEW
VOLATILE
WAITFOR
WHEN
WHENEVER
WHERE
WHILE
WIDTH_BUCKET
WINDOW
WITH
WITHIN
WITHOUT
WORK
WRITE
WRITETEXT
X509
XOR
YEAR
YEAR_MONTH
ZEROFILL
ZONE
```


[simon]: https://www.simonholywell.com/?utm_source=sqlstyle.guide&utm_medium=link&utm_campaign=md-document
    "SimonHolywell.com"
[iso-8601]: https://en.wikipedia.org/wiki/ISO_8601
    "Wikipedia: ISO 8601"
[rivers]: https://practicaltypography.com/one-space-between-sentences.html
    "Practical Typography: one space between sentences"
[reserved-keywords]: #reserved-keyword-reference
    "Reserved keyword reference"
[sqlstyleguide]: https://www.sqlstyle.guide/
    "SQL style guide by Simon Holywell"
[licence]: https://creativecommons.org/licenses/by-sa/4.0/
    "Creative Commons Attribution-ShareAlike 4.0 International License"
