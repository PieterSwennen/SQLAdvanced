# H5 Writing Control Structures

## Continue Statement - example

```sql
DECLARE
  v_total NUMBER := 0;
BEGIN
  <<BeforeTopLoop>>
  FOR i IN 1..10 LOOP
    v_total := v_total + 1;
    DBMS_OUTPUT.PUT_LINE('Total is: '|| v_total);

    FOR j in 1..10 LOOP
      CONTINUE BeforeTopLoop WHEN i + j > 5;
      v_total := v_total + 1;
    END LOOP;
  END LOOP;
END;
```
<div style="page-break-after: always;"></div>
## Exericse 1
> Execute this script to create the `messages` table
>
> ```sql
> DROP TABLE messages;
> CREATE TABLE messages (results VARCHAR2(80));
> ```
>
> Write a PL/SQL block to insert numbers into the messages table.  
> A\) Insert the numbers 1 to 10, excluding 6 and 8.  
> B\) Commit before the end of the block.  
> C\) Execute a `SELECT` statement to verify that your PL/SQL block worked.  
> You should see the following output:
>
> ```
> RESULTS
> ---------------------------------------
> 1
> 2
> 3
> 4
> 5
> 7
> 9
> 10
>
> 8 rows selected
> ```

```sql
BEGIN
  for i IN 1..10 LOOP
    if i NOT IN (6,8)
      THEN
        INSERT INTO messages
        values(i);  
    END if;
   END loop;
END;
```

of

```sql
BEGIN
  FOR i in 1..10 LOOP
    IF i= 6 or i=8
      THEN
        null;
    ELSE
      INSERT INTO messages(results)
        VALUES(i);
    END if;
  END loop;
COMMIT;
END;
```
<div style="page-break-after: always;"></div>
## Exercise 2
> Execute this script to create the `emp` table
>
> ```sql
> DROP TABLE emp; --Only when the emp table exists.
> CREATE TABLE emp AS SELECT *
>                     FROM employees;
> ALTER TABLE emp ADD stars VARCHAR2(50);
> ```
>
> Create a PL/SQL block that inserts an asterisk in the stars column for every $1,000 of the employee's salary.
>
> A\) In the **declarative** section of the block, declare a variable `v_empno` of type `emp.eployee_id` and initialize it to 176. Declare a variable `v_asterisk` of type `emp.stars`  
> and initialize it to `NULL`. Create a variable `sal` of type `emp.salary`.
>
> B\) In the **executable** section, write logic to append an asterisk \(\*\)to the string for every $1,000 of the salary amount.  
> For example, if the employee earns $8,000, the string of asterisks should contain eight asterisks. If the employee earns $12,500, the string of  
> asterisks shoud contain 13 asterisks.
>
> C\) Update the stars column for the employee with the string of asterisks. Commit before the end of the block.

```sql
DECLARE
    v_empno    emp.employee_id%TYPE  := 0;
    v_asterisk emp.stars%TYPE        := null;
    v_sal      emp.salary%TYPE;
    v_count    NUMBER(10);
BEGIN
    SELECT COUNT(employee_id)
  INTO v_count
  FROM emp;

    FOR i IN 1..v_count
     LOOP
    SELECT employee_id
    INTO v_empno
        FROM
            (SELECT *
        FROM emp
        WHERE employee_id > v_empno)
        WHERE ROWNUM <= 1;

        EXIT WHEN i = v_count + 1;

        SELECT NVL(ROUND(salary/1000), 0)
    INTO v_sal
    FROM emp
    WHERE employee_id = v_empno;

        DBMS_OUTPUT.PUT_LINE(v_sal);

        FOR i IN 1..v_sal
          LOOP
               v_asterisk := v_asterisk || '*';
          END LOOP;

        UPDATE emp
    SET stars = v_asterisk
        WHERE employee_id = v_empno;

        v_asterisk := null;
    END LOOP;
    COMMIT;
END;
/

SELECT *
FROM emp;
```
<div style="page-break-after: always;"></div>
## Extra Oefening 1
> Schrijf een anoniem blok dat het volgende doet:
>
> * je laat de gebruiker een afdelingsid ingeven en een percentage voor loonsverhoging
> * je toont van het ingegeven afdelingsid de naam van de afdeling en alle werknemers die er werken met hun salaris  
> * je geeft iedereen een loonsverhoging  \(ingevoerde percentage\)
> * vervolgens toon je terug alle werknemers met hun nieuwe salaris

```sql
SET SERVEROUTPUT ON
SET AUTOPRINT ON
SET VERIFY OFF
DECLARE
  TYPE employee_table 
    IS TABLE OF employees%ROWTYPE INDEX BY PLS_INTEGER;

  v_dep_name         departments.department_name%TYPE;
  v_new_salary       employees.salary%TYPE;
  v_employees        employee_table;
  
  v_dep_id           
    departments.department_id%TYPE := '&department_id';
    
  v_raise_percentage 
    NUMBER(2) := '&raise_percentage';

BEGIN
  SELECT department_name
  INTO v_dep_name
  FROM departments
  WHERE department_id = v_dep_id;

  DBMS_OUTPUT.PUT('Department : ');
  DBMS_OUTPUT.PUT(v_dep_name);
  DBMS_OUTPUT.NEW_LINE();

  SELECT * BULK COLLECT
  INTO v_employees
  FROM employees
  WHERE department_id = v_dep_id;

  FOR i in 1.. v_employees.COUNT LOOP
    UPDATE employees
    SET salary = (salary * (v_raise_percentage/100+1))
    WHERE employee_id = v_employees(i).employee_id;

    SELECT salary
    INTO v_new_salary
    FROM employees
    WHERE employee_id = v_employees(i).employee_id;

    DBMS_OUTPUT.NEW_LINE();
    DBMS_OUTPUT.PUT(v_employees(i).first_name ||' ');
    DBMS_OUTPUT.PUT(v_employees(i).last_name);
    DBMS_OUTPUT.NEW_LINE();
    DBMS_OUTPUT.PUT(v_employees(i).salary || ' -> ');
    DBMS_OUTPUT.PUT(v_new_salary);
    DBMS_OUTPUT.NEW_LINE();
  END LOOP;
ROLLBACK;
END;
```

> Model oplossing:

```sql
DECLARE
   v_department_name departments.department_name%type;
   v_perc            number(3);
   
   v_department_id   
     employees.department_id%type := &departementid;
BEGIN
  SELECT department_name
  INTO v_department_name
  FROM departments
  WHERE department_id = v_department_id;

  DBMS_OUTPUT.PUT_LINE( 'het gekozen departement is : ' || 
                         v_department_name);

  FOR lijn IN ( SELECT last_name, employee_id, salary
                FROM employees
                WHERE department_id = v_department_id)
    LOOP
      DBMS_OUTPUT.PUT_LINE( 
        lijn.employee_id || ' ' ||
        lijn.last_name || ': ' ||
        lijn.salary
      );
    END LOOP;

  v_perc := &percentage;

  UPDATE employees
  SET salary = salary * (1 + v_perc/100)
  WHERE department_id = v_department_id;

  DBMS_OUTPUT.PUT_LINE(
    'aantal salarisverhogingen : ' || sql%rowcount
  );
  DBMS_OUTPUT.PUT_LINE('situatie na wijziging:');

  FOR lijn IN (SELECT last_name, employee_id, salary
               FROM employees
               WHERE department_id = v_department_id)
    LOOP
      DBMS_OUTPUT.PUT_LINE( 
        lijn.employee_id || ' ' ||
        lijn.last_name || ': '||
        lijn.salary
      );
    END LOOP;
END;
/
```
<div style="page-break-after: always;"></div>
## Extra Oefening 2
> Schrijf een anoniem blok dat onderstaand overzicht maakt:  
> per stad toon je de afdelingen die erin gevestigd zijn met hun aantal werknemers.  Toon enkel de steden waar effectief afdelingen gevestigd zijn.

```sql
DECLARE
  TYPE location_table_type 
    IS TABLE OF locations%ROWTYPE INDEX BY PLS_INTEGER;
  TYPE department_table_type 
    IS TABLE OF departments%ROWTYPE INDEX BY PLS_INTEGER;

  v_locations     location_table_type;
  v_departments   department_table_type;
  v_dep_name      departments.department_name%TYPE;
  v_amount_emp    NUMBER;
  v_country_name  countries.country_name%TYPE;

BEGIN
  SELECT DISTINCT l.* BULK COLLECT
  INTO v_locations
  FROM departments d
  LEFT JOIN locations l
  ON(l.location_id = d.location_id)
  ORDER BY l.location_id;

  FOR i in 1.. v_locations.COUNT LOOP
    SELECT country_name
    INTO v_country_name
    FROM countries
    WHERE country_id = v_locations(i).country_id;

    DBMS_OUTPUT.PUT(v_locations(i).location_id ||' ');
    DBMS_OUTPUT.PUT(v_locations(i).city);
    DBMS_OUTPUT.PUT_LINE(' ('||v_country_name||')');
    DBMS_OUTPUT.PUT_LINE('------------------------------');

    SELECT * BULK COLLECT
    INTO v_departments
    FROM departments
    WHERE location_id = v_locations(i).location_id;

    FOR j in 1.. v_departments.COUNT LOOP
      SELECT COUNT(employee_ID)
      INTO v_amount_emp
      FROM employees
      WHERE department_id = v_departments(j).department_id;

      DBMS_OUTPUT.PUT(v_departments(j).department_name||' : ');
      DBMS_OUTPUT.PUT(v_amount_emp);
      DBMS_OUTPUT.PUT(' Werknemers');
      DBMS_OUTPUT.NEW_LINE();
    END LOOP;
    DBMS_OUTPUT.NEW_LINE();
  END LOOP;
END;
```

> Model oplossing:

```sql
DECLARE
   v_country_name countries.country_name%type;
BEGIN
  FOR locrec IN ( SELECT *
                  FROM locations
                  WHERE location_id
                  IN (SELECT location_id
                      FROM departments)) 
    LOOP
      SELECT country_name
      INTO v_country_name
      FROM countries
      WHERE country_id = locrec.country_id;

      DBMS_OUTPUT.PUT_LINE(
        '==> ' ||locrec.location_id || 
        ' ' || locrec.city || 
        ' (' || v_country_name || 
        ')');

      FOR deprec IN (
        SELECT department_name, count(*) aantal
        FROM departments d
        JOIN employees e
        USING (department_id)
        WHERE d.location_id = locrec.location_id
        GROUP BY department_name
        ) 
        LOOP
          DBMS_OUTPUT.PUT_LINE(
            deprec.department_name || ': ' || 
            deprec.aantal
          );
        END LOOP;
     END LOOP;
END;
```



