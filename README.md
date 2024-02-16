<h1>Essential PL/pgSQL for 2110322 Database Systems</h1>

#### Table of contents
1. [Block Structure](#block-structure)
2. [Variables](#variables)
3. [RECORD types](#record-types)
4. [ASSERT statement](#assert-statement)
5. [IF statement](#if-statement)
6. [Loops](#loops)
7. [CREATE FUNCTION](#create-function)
8. [PL/pgSQL parameter modes](#pl-pgsql-parameter-modes)
9. [Function that returns a table](#function-that-returns-a-table)
10. [DROP FUNCTION statement](#drop-function-statement)
11. [Calling a function](#calling-a-function)
12. [Transaction](#transaction)
13. [CREATE PROCEDURE](#create-procedure)
14. [DROP PROCEDURE statement](#drop-procedure-statement)
15. [CREATE TRIGGER](#create-trigger)
16. [DROP/DISABLE/ENABLE TRIGGER](#drop-disable-enable-trigger)
17. [CREATE VIEW](#create-view)
18. [DROP VIEW statement](#drop-view-statement)

## Block Structure <a name = "block-structure"></a>

#### Syntax of a complete block in PL/pgSQL

```pgsql
[<<label>>]
[DECLARE
    declarations]
BEGIN
    statements;
        ...
END [label];
```

- Each block has two sections: declaration (optional) and body.
- A block is ended with a semicolon (`;`) after the `END` keyword.
- The declaration section is where you declare all variables used within the body section.
- The body section is where you place the code.
- Each declaration and statement ends with a semicolon (`;`).

#### Block structure example

```pgsql
<<first_block>>
DECLARE
    member_count INT := 0;
BEGIN
    -- get the number of members
    SELECT COUNT(*)
    INTO member_count
    FROM members
    -- display a message
    RAISE NOTICE 'The number of films is %', member_count;
END first_block;
```

#### An anonymous block

Because our block has no name, it must be called with `DO`

```pgsql
DO $$
<<first_block>>
DECLARE
    member_count INT := 0;
BEGIN
    -- get the number of members
    SELECT COUNT(*)
    INTO member_count
    FROM members
    -- display a message
    RAISE NOTICE 'The number of films is %', member_count;
END first_block $$;
```

## Variables <a name="variables"></a>

#### Syntax of declaring a variable.

```pgsql
variable_name data_type [:= expression];
```

#### Basic variable example

```pgsql
DO $$
DECLARE
    first_name VARCHAR(20);
    last_name VARCHAR(20) := 'Wongmanit';
    payment NUMERIC := 13.3;
BEGIN
    -- you also can assign a value later
    first_name := 'Weerawat';

    RAISE NOTICE '% % has been paid % USD',
        first_name,
        last_name,
        payment;
END $$;
```

#### Copying data types

```pgsql
DO $$
DECLARE
    -- Declare member_id with the same data type as id column of members table
    member_id members.id%TYPE;
    -- Same with member_name
    member_name members.name%TYPE;
    -- Declare member_detail by using a whole row of members table as the data type
    member_detail members%ROWTYPE;
BEGIN
    SELECT id, name
    INTO member_id, member_name
    FROM members
    WHERE id = '01';

    SELECT *
    INTO member_detail
    FROM members
    WHERE id = '01';
END $$;
```

## RECORD types <a name="record-types"></a>

A `RECORD` variable is similar to a `ROWTYPE` variable. It can hold only one row of a result set.

Unlike a `ROWTYPE` variable, a `RECORD` vairable does not have a predefine structure. It is determind when the `SELECT` or `FOR` statement assigns an actual row to it.

#### Using RECORD with SELECT INTO

```pgsql
DO $$
DECLARE
    rec RECORD;
BEGIN
    SELECT id, name, address
    INTO rec
    FROM members
    WHERE id = '01';
END $$;
```

#### Using RECORD with FOR loop

```pgsql
DO $$
DECLARE
    rec RECORD;
BEGIN
    FOR rec IN
        SELECT id, name, address
        FROM members
        WHERE id = '01'
    LOOP
        RAISE NOTICE '% % %', rec.id, rec.name, rec.address;
    END LOOP; 
END $$;
```

## ASSERT statement <a name="assert-statement"></a>

The `ASSERT` statement is a useful shorthand for inserting debugging checks into PL/pgSQL code.

If the condition evaluates to true, the `ASSERT` statement does nothing.

#### Syntax of the ASSERT statement

```pgsql
ASSERT condition [, message];
```

#### ASSERT statement example

```pgsql
DO $$
DECLARE
    member_count INT;
BEGIN
    SELECT COUNT(*)
    INTO member_count
    FROM members;

    ASSERT member_count > 0, 'There is no data in members table';
END $$;
```

## IF statement <a name="if-statement"></a>

The following illustrates the simplest form of the `IF` statement:

```pgsql
IF condition THEN
    statements;
END IF;
```

The `IF` statement executes statements if a condition is true. If the condition evaluates to false, the control is passed to the next statement after the `END IF` part.

```pgsql
DO $$
DECLARE
    member_id members.id%TYPE;
    member_detail members%ROWTYPE;
BEGIN
    SELECT *
    INTO member_detail
    FROM members
    WHERE id = member_id;

    IF NOT FOUND THEN
        RAISE NOTICE 'Member with id % is not found.', member_id;
    END IF;
END $$;
```

The `FOUND` is a global variable that is available in PL/pgSQL procedure language. The select into statement sets the `FOUND` variable if a row is assigned or false if no row is returned.

#### IF-THEN-ELSIF-ELSE statement

```pgsql
DO $$
DECLARE
    member_id members.id%TYPE;
    member_detail members%ROWTYPE;
    member_tier VARCHAR(15);
BEGIN
    SELECT *
    INTO member_detail
    FROM members
    WHERE id = member_id;

    IF NOT FOUND THEN
        RAISE NOTICE 'Member with id % is not found.', member_id;
    ELSE
        IF member_detail.monthly_payment > 500 THEN
            member_tier := 'Platinum';
        ELSIF member_detail.monthly_payment > 250 THEN
            member_tier := 'Gold';
        ELSIF member_detail.monthly_payment > 100 THEN
            member_tier := 'Silver';
        ELSE
            member_tier := 'Normal';
        END IF;
        RAISE NOTICE 'Member with id % is % tier', member_id, member_tier;
    END IF;
END $$;
```

## Loops <a name="loops"></a>

#### Syntax of WHILE loop

```pgsql
WHILE condition LOOP
    statements;
END LOOP;
```

If the condition is true, it executes the statements. After each iteration, the `WHILE` loop evaluates the condition again.

#### WHILE loop example

```pgsql
DO $$
DECLARE
    counter INT := 0
BEGIN
    WHILE COUNTER < 5 LOOP
        RAISE NOTICE 'Counter %', counter;
        counter := counter + 1;
    END LOOP;
END $$;
```

#### Syntax of FOR loop (iterate over a range of integers)

```pgsql
FOR counter IN [REVERSE] from ... to [BY step] LOOP
    statements;
END LOOP;
```

#### FOR loop example (iterate over a range of integers)

```pgsql
FOR counter IN [REVERSE] from ... to [BY step] LOOP
    statements;
END LOOP;
```

#### Syntax of FOR loop (iterate over a result set)

```pgsql
FOR target IN query LOOP
    statements;
END LOOP;
```

#### FOR loop example (iterate over a result set)

```pgsql
DO $$
DECLARE
    rec RECORD;
BEGIN
    FOR rec IN (
        SELECT id, name
        FROM members
        LIMIT 5
    )
    LOOP
        RAISE NOTICE '% %', rec.id, rec.name
    END LOOP;
END $$;
```

## CREATE FUNCTION <a name="create-function"></a>

The `CREATE FUNCTION` statement allows you to define a new user-defined function.

#### Syntax of CREATE FUNCTION statement

```pgsql
CREATE [OR REPLACE] FUNCTION function_name(parameter_list)
RETURNS return_type
LANGUAGE PLPGSQL
AS $$
DECLARE
    -- variable declaration
BEGIN
    -- logic
END $$;
```

#### CREATE FUNCTION statement example

```pgsql
CREATE OR REPLACE FUNCTION get_member_tier(member members%ROWTYPE)
RETURNS VARCHAR
LANGUAGE PLPGSQL
AS $$
DECLARE
    member_tier VARCHAR(15);
BEGIN
    IF member_detail.monthly_payment > 500 THEN
        member_tier := 'Platinum';
    ELSIF member_detail.monthly_payment > 250 THEN
        member_tier := 'Gold';
    ELSIF member_detail.monthly_payment > 100 THEN
        member_tier := 'Silver';
    ELSE
        member_tier := 'Normal';
    END IF;

    RETURN member_tier;
END $$;
```

## PL/pgSQL parameter modes <a name="pl-pgsql-parameter-modes"></a>

|IN                      |OUT                            |INOUT                                                 |
|------------------------|-------------------------------|------------------------------------------------------|
|The default             |Explicitly specified           |Explicitly specified                                  |
|Pass a value to function| Return a value from a function|Pass a value to a function and return an updated value|

#### IN mode example

```pgsql
CREATE OR REPLACE FUNCTION find_member_by_id(member_id VARCHAR)
RETURNS VARCHAR
LANGUAGE PLPGSQL
AS $$
DECLARE
    member members%ROWTYPE
BEGIN
    SELECT *
    INTO member
    FROM members
    WHERE id = member_id;

    IF NOT FOUND THEN
        RAISE NOTICE 'Member with id % not found', member_id;
    END IF;

    return member;
END $$;
```

Because we don’t specify the mode for `member_id` parameter, it takes the in mode by default.

#### OUT mode example

```pgsql
CREATE OR REPLACE FUNCTION platinum_member_count(
    OUT counter INT
)
LANGUAGE PLPGSQL
AS $$
BEGIN
    SELECT COUNT(*)
    INTO counter -- We can assign a value to our OUT parameter directly!
    FROM members
    WHERE monthly_payment > 500;
END $$;
```

#### INOUT mode example

```pgsql
CREATE OR REPLACE FUNCTION swap(
    INOUT x INT,
    INOUT y INT
)
LANGUAGE PLPGSQL
AS $$
BEGIN
    SELECT x, y
    INTO y, x;
END $$;
```

## Function that returns a table <a name="function-that-returns-a-table"></a>

To define a function that returns a table, you use the following form of the create function statement:

```pgsql
CREATE OR REPLACE FUNCTION function_name (
    parameter_list
)
RETURNS TABLE (
    column_list
)
LANGUAGE PLPGSQL
AS $$
DECLARE
    -- variable declaration
BEGIN
    -- body
END $$;
```

#### Returning a table example

```pgsql
CREATE OR REPLACE FUNCTION get_platinum()
RETURNS TABLE (
    member_id VARCHAR,
    member_name VARCHAR
)
LANGUAGE PLPGSQL
AS $$
BEGIN
    RETURN QUERY (
        SELECT id, name
        FROM members
        WHERE monthly_payment > 500
    );
END $$;
```

## DROP FUNCTION statement <a name="drop-function-statement"></a>

#### Syntax of DROP FUNCTION

```pgsql
DROP FUNCTION [IF EXISTS] function_name(argument_list)
[CASCADE | RESTRICT]
```

## Calling a function <a name="calling-a-function"></a>

#### Calling a function example

Suppose we have a table `members`:

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |999            |'Rice Fields' |
|'02'|'Tai Pamoke'     |1              |'Pamoke'      |
|'03'|'Mookrob Mode'   |500            |'Taweewattana'|
|'04'|'Sithata Kung'   |550            |'Gabinlapat'  |
|'05'|'Jesus Nice'     |400            |'Jerusalem'   |
|'06'|'Zark Muckerberg'|200            |'Metaverse'   |

<b>Example 1</b>

```pgsql
CREATE OR REPLACE FUNCTION get_member_tier(member members%ROWTYPE)
RETURNS VARCHAR
LANGUAGE PLPGSQL
AS $$
DECLARE
    member_tier VARCHAR(15);
BEGIN
    IF member_detail.monthly_payment > 500 THEN
        member_tier := 'Platinum';
    ELSIF member_detail.monthly_payment > 250 THEN
        member_tier := 'Gold';
    ELSIF member_detail.monthly_payment > 100 THEN
        member_tier := 'Silver';
    ELSE
        member_tier := 'Normal';
    END IF;

    RETURN member_tier;
END $$;

SELECT id, name, get_member_tier(*)
FROM members;
```

<b>Output:</b>

|id  |name             |member_tier|
|----|-----------------|-----------|
|'01'|'John Chaorai'   |Platinum   |
|'02'|'Tai Pamoke'     |Normal     |
|'03'|'Mookrob Mode'   |Gold       |
|'04'|'Sithata Kung'   |Platinum   |
|'05'|'Jesus Nice'     |Gold       |
|'06'|'Zark Muckerberg'|Silver     |

<b>Example 2</b>

```pgsql
CREATE OR REPLACE FUNCTION get_platinum()
RETURNS TABLE (
    member_id VARCHAR,
    member_name VARCHAR
)
LANGUAGE PLPGSQL
AS $$
BEGIN
    RETURN QUERY (
        SELECT id, name
        FROM members
        WHERE monthly_payment > 500
    );
END $$;

SELECT *
FROM get_platinum();
```

<b>Output:</b>

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |999            |'Rice Fields' |
|'04'|'Sithata Kung'   |550            |'Gabinlapat'  |

## Transaction <a name="transaction"></a>

A database transaction is a single unit of work that consists of one or more operations.

A PostgreSQL transaction is atomic, consistent, isolated, and durable. These properties are often referred to collectively as ACID.

#### Starting a transaction

To start a transaction explicitly, you execute either one of the following statements:

```pgsql
BEGIN TRANSACTION;
```

Or

```pgsql
BEGIN WORK;
```

Or
```pgsql
BEGIN;
```

#### COMMIT statement

Suppose we have a table `members`:

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'Zark Muckerberg'|200            |'Metaverse'   |

The following statements start a new transaction and insert a new member into the members table:

```pgsql
-- from session 1
BEGIN;

INSERT INTO members VALUES('02', 'Elong Ma', 9999, 'Mars');
```

From the current session, you can see the change by retrieving data from the members:

```pgsql
-- from session 1
SELECT *
FROM members;
```

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'Zark Muckerberg'|200            |'Metaverse'   |
|'02'|'Elong Ma'       |9999           |'Mars'        |

However, you will not see the change if you connect to the PostgreSQL server in a new session and execute the query above.

```pgsql
-- from session 2
SELECT *
FROM members;
```

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'Zark Muckerberg'|200            |'Metaverse'   |

To permanently apply the change to the database, you commit the transaction by using the `COMMIT` statement:

```pgsql
-- from session 1
COMMIT;
```

Now other sessions can view the change by retrieving data from the accounts table:

```pgsql
-- from session 2
SELECT *
FROM members;
```

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'Zark Muckerberg'|200            |'Metaverse'   |
|'02'|'Elong Ma'       |9999           |'Mars'        |

#### ROLLBACK statement

If you want to undo the changes to the database, you can use the ROLLBACK statement:

```pgsql
ROLLBACK;
```

Suppose we have a table `members`:

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'Zark Muckerberg'|200            |'Metaverse'   |

The following statements use the ROLLBACK statement to roll back the changes made to the members table.

```pgsql
-- from session 1
BEGIN;

INSERT INTO members VALUES('02', 'Elong Ma', 9999, 'Mars');

ROLLBACK;
```

If you retrieve data from the members table, you’ll won’t see the changes because it was rolled back.

```pgsql
SELECT *
FROM members;
```

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'Zark Muckerberg'|200            |'Metaverse'   |

## CREATE PROCEDURE <a name="create-procedure"></a>

A drawback of user-defined functions is that they cannot execute transactions. So PostgreSQL 11 introduced stored procedures that support transactions.

#### Syntax of CREATE PROCEDURE statement

```pgsql
CREATE [OR REPLACE] PROCEDURE procedure_name(parameter_list)
LANGUAGE PLPGSQL
AS $$
DECLARE
    -- variable declaration
BEGIN
    -- stored procedure body
END $$;
```

Parameters in stored procedures can have the `IN` and `INOUT` modes but cannot have the out mode.

A stored procedure does not return a value. However, you can use the `return` statement without the expression to stop the stored procedure immediately:

```pgsql
return;
```

#### CREATE PROCEDURE example

```pgsql
CREATE OR REPLACE PROCEDURE add_payment(
    member_id VARCHAR,
    amount DECIMAL,
)
LANGUAGE PLPGSQL
AS $$
BEGIN
    -- add amount to monthly_payment of the member with member_id
    UPDATE members
    SET monthly_payment = monthly_payment + amount
    WHERE id = member_id;
END $$;
```

#### Calling a procedure

```pgsql
CALL procedure_name(argument_list);
```

#### Calling a procedure example

Suppose we have a table `members`:

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |999            |'Rice Fields' |
|'02'|'Tai Pamoke'     |1              |'Pamoke'      |
|'03'|'Mookrob Mode'   |500            |'Taweewattana'|
|'04'|'Sithata Kung'   |550            |'Gabinlapat'  |
|'05'|'Jesus Nice'     |400            |'Jerusalem'   |
|'06'|'Zark Muckerberg'|200            |'Metaverse'   |

<b>Example 1</b>

```pgsql
CREATE OR REPLACE PROCEDURE add_payment(
    member_id VARCHAR,
    amount DECIMAL,
)
LANGUAGE PLPGSQL
AS $$
BEGIN
    -- add amount to monthly_payment of the member with member_id
    UPDATE members
    SET monthly_payment = monthly_payment + amount
    WHERE id = member_id;
END $$;

CALL add_payment('02', 100.50);
```

<b>Output:</b>

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |999            |'Rice Fields' |
|'02'|'Tai Pamoke'     |101.50         |'Pamoke'      |
|'03'|'Mookrob Mode'   |500            |'Taweewattana'|
|'04'|'Sithata Kung'   |550            |'Gabinlapat'  |
|'05'|'Jesus Nice'     |400            |'Jerusalem'   |
|'06'|'Zark Muckerberg'|200            |'Metaverse'   |

<b>Example 2</b>

```pgsql
CREATE OR REPLACE PROCEDURE reset_payment()
LANGUAGE PLPGSQL
AS $$
DECLARE
    member_id members.id%TYPE;
BEGIN
    FOR member_id IN (
        SELECT id
        FROM members
    )
    LOOP
        UPDATE members
        SET monthly_payment = 0
        WHERE id = member_id;

        -- we can use COMMIT (or ROLLBACK) to update our database occasionally. 
        COMMIT;
    END LOOP;
END $$;

CALL reset_payment();
```

<b>Output:</b>

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |0              |'Rice Fields' |
|'02'|'Tai Pamoke'     |0              |'Pamoke'      |
|'03'|'Mookrob Mode'   |0              |'Taweewattana'|
|'04'|'Sithata Kung'   |0              |'Gabinlapat'  |
|'05'|'Jesus Nice'     |0              |'Jerusalem'   |
|'06'|'Zark Muckerberg'|0              |'Metaverse'   |

## DROP PROCEDURE statement <a name="drop-procedure-statement"></a>

#### Syntax of DROP FUNCTION

```pgsql
DROP PROCEDURE [IF EXISTS] procedure_name(argument_list)
[CASCADE | RESTRICT]
```

## Trigger <a name="trigger"></a>

A PostgreSQL trigger is a function invoked automatically whenever an event associated with a table occurs. An event could be any of the following: `INSERT`, `UPDATE`, `DELETE` or `TRUNCATE`.

PostgreSQL provides two main types of triggers:
- Row-level triggers
- Statement-level triggers.

The differences between the two kinds are how many times the trigger is invoked and at what time.

## CREATE TRIGGER <a name="create-trigger"></a>

To create a new trigger in PostgreSQL, you follow these steps:

- First, create a trigger function using `CREATE FUNCTION` statement.
- Second, bind the trigger function to a table by using `CREATE TRIGGER` statement.

#### Syntax of create trigger function

```pgsql
CREATE FUNCTION [OR REPLACE] trigger_function()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS $$
BEGIN
    -- trigger logic
END $$;
```

#### Syntax of create trigger (binding trigger)

```pgsql
CREATE TRIGGER trigger_name 
    {BEFORE | AFTER} { EVENT }
    ON table_name
    [FOR [EACH] { ROW | STATEMENT }]
        EXECUTE PROCEDURE trigger_function
```

#### CREATE TRIGGER example

Suppose we have a table `members`:

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |999            |'Rice Fields' |
|'02'|'Tai Pamoke'     |1              |'Pamoke'      |
|'03'|'Mookrob Mode'   |500            |'Taweewattana'|
|'04'|'Sithata Kung'   |550            |'Gabinlapat'  |
|'05'|'Jesus Nice'     |400            |'Jerusalem'   |
|'06'|'Zark Muckerberg'|200            |'Metaverse'   |

and an audit log table of each updated payment `payment_audits` with the following schema:

|attributes |data types|
|-----------|----------|
|id         |VARCHAR   |
|new_amount |DECIMAL   |
|changed_on |TIMESTAMP |

First, create a new function called `log_update_payment`:

```pgsql
CREATE OR REPLACE FUNCTION log_update_payment()
RETURNS TRIGGER
LANGUAGE PLPGSQL
AS $$
BEGIN
    IF NEW.monthly_payment <> OLD.monthly_payment THEN
        INSERT INTO payment_audits VALUES(OLD.id, NEW.monthly_payment, NOW());
    END IF;

    RETURN NEW;
END $$;
```

Second, bind the trigger function to the `members` table. The trigger name is `update_payment`. Before the value of the `monthly_payment` column is updated.

```pgsql
CREATE TRIGGER update_payment
    BEFORE UPDATE
    ON members
    FOR EACH ROW
    EXECUTE PROCEDURE log_update_payment();
```

Supposed that Zark Muckerberg updates his `monthly_payment`:

```pgsql
UPDATE members
SET monthly_payment = 500
WHERE id = '06';
```

Verify the contents of `payment_audits` table:

```pgsql
SELECT * FROM payment_audits;
```

|id  |new_amount|changed_on                |
|----|----------|--------------------------|
|'06'|500       |2024-02-16 23:57:13.123456|

The change was logged in the `payment_audits` table by the trigger.

## DROP/DISABLE/ENABLE TRIGGER <a name="drop-disable-enable-trigger"></a>

#### Syntax of drop a trigger

```pgsql
DROP TRIGGER [IF EXISTS] trigger_name
ON table_name [CASCADE | RESTRICT];
```

#### Syntax of disable a trigger

```pgsql
ALTER TABLE table_name
DISABLE TRIGGER {trigger_name | ALL};
```

#### Syntax of enable a trigger

```pgsql
ALTER TABLE table_name
ENABLE TRIGGER {trigger_name | ALL};
```

## CREATE VIEW <a name="create-view"></a>

#### Syntax of CREATE VIEW statement

```pgsql
CREATE VIEW view_name
AS
    query;
```

#### CREATE VIEW example

Suppose we have a table `members`:

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'01'|'John Chaorai'   |999            |'Rice Fields' |
|'02'|'Tai Pamoke'     |1              |'Pamoke'      |
|'03'|'Mookrob Mode'   |500            |'Taweewattana'|
|'04'|'Sithata Kung'   |550            |'Gabinlapat'  |
|'05'|'Jesus Nice'     |400            |'Jerusalem'   |
|'06'|'Zark Muckerberg'|200            |'Metaverse'   |

```pgsql
CREATE VIEW get_gold
AS (
    SELECT *
    FROM members
    WHERE monthly_payment BETWEEN 251 AND 500
);

CREATE VIEW gold_max_payment
AS (
    SELECT *
    FROM get_gold
    WHERE monthly_payment IN (
        SELECT MAX(monthly_payment)
        FROM get_gold
    )
);

SELECT id, name
FROM gold_max_payment;
```

<b>Output:</b>

|id  |name             |monthly_payment|address       |
|----|-----------------|---------------|--------------|
|'03'|'Mookrob Mode'   |500            |'Taweewattana'|

If we update `members` table, any views related to the table also get updated.

## DROP VIEW statement <a name="drop-view-statement"></a>

#### Syntax of drop view

```pgsql
DROP VIEW [IF EXISTS] view_name
[CASCADE | RESTRICT];
```