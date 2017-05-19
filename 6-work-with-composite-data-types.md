# 6 Working with Composite Data Types

## Exercise 1

> Write a PL/SQL block to print information about a given country.
>
> A\) Declare a PL/SQL record bqsed on the structure of the `countries` table.
>
> B\) Declare a variable `v_country_id`. Assign `CA` to `v_country_id`.
>
> C\) In the **declarative** section, use the `%ROWTYPE` attribute and declare the `v_country_record` variable of type `countries`.
>
> D\) In the executable section, get all the information from the `countries` table by using  
> contry\_id. Display selected information about the country. Sample output is as follows:
>
> ```
> Country ID : CA Country Name : Canada Region : 2
> ```
>
> E\) You may want to execute and test the PL/SQL block for the countries with the IDs  
> DE, UK, US.

```sql
DECLARE
  v_country_rec   countries%ROWTYPE;
  v_country_id    VARCHAR2(20)        := 'CA';
BEGIN
  SELECT *
  INTO v_country_rec
  FROM countries
  WHERE country_id = UPPER(v_country_id);

  DBMS_OUTPUT.PUT_LINE('Country Id: ' || v_country_rec.country_id ||
                        ' Country Name: ' || v_country_rec.country_name ||
                        ' Region: ' || v_country_rec.region_id);
END;
```

## Exercise 2

> Create a PL/SQL block to retrieve the name of some `departments` from the departments table and  
> print each department name on the screen, incorporating an `INDEX BY` table.
>
> A\) Declare an `INDEX BY` table `dept_table_type` of type `departments.department_name`. Declare a variables  
> `my_dept_table` of type `dept_table_type` to temporarily store the name of the derpartments.
>
> B\) Declare two variables: `loop_count` and `deptno` of type `NUMBER`. Assign 10 to `loop_count`  
> and 0 to `deptno`.
>
> C\) Using a loop, retrieve the name of 10 departments and store the names in the `INDEX BY` table.  
> Start with `department_id` 10. Increase `deptno` by 10 for every iteration of the loop. The following table shows the  
> `department_id` for which you should retrieve the `department_name` and store in the `INDEX BY` table.
>
> | DEPARTEMENT\_ID | DEPARMENT\_NAME |
> | :---: | :---: |
> | 10 | Administration |
> | 20 | Marketing |
> | 30 | Purchasing |
> | 40 | Human Resources |
> | 50 | Shipping |
> | 60 | IT |
> | 70 | Public Relations |
> | 80 | Sales |
> | 90 | Executive |
> | 100 | Finance |
>
> D\) Using another loop, retrieve the department names form the `INDEX BY` table and display them.
>
> E\) Execute your script. The output is as follows:
>
> ```
> Administration
> Marketing      
> Purchasing
> Human Resources
> Shipping
> IT
> Public Relations
> Sales
> Executive
> Finance
> ```

```sql
DECLARE
    TYPE dept_table_type IS TABLE OF departments.department_name%TYPE INDEX BY PLS_INTEGER;
    my_dept_table  dept_table_type;
    loop_count     NUMBER(2) := 10;
    deptno         NUMBER(4) := 0;
BEGIN
    FOR i IN 1..loop_count
    LOOP
        deptno := 10 + deptno;
        SELECT department_name
    INTO my_dept_table(i)
        FROM departments
        WHERE department_id = deptno;
    END LOOP;

    FOR j IN 1..loop_count
    LOOP
        DBMS_OUTPUT.PUT_LINE(my_dept_table(j));
    END LOOP;
END;
```

## Exercise 3

> Modify the block that you created in exercise 2 to retrieve all information about each department  
> from the `departments` table and display the information. Us an `INDEX BY` table of records.
>
> A\) Load the script from exercise 2
>
> B\) You have declared the `INDEX BY` table to be of type `departments.department_name`. Modify the declaration of the  
> `INDEX BY` table, to temporarily store the number, name and location of the departments. Use teh `%ROWTYPE` attribute.
>
> C\) Modify the `SELECT` statement to retrieve all department information currently in the `departments` table and store  
> it in the `INDEX BY` table.
>
> D\) Using another loop, retrieve the department information from the `INDEX BY` table and display the information. Sample output is as follows.
>
> ```
> Department Number: 10 Department Name: Administration Manager Id: 200 Location Id: 1700  
> ...
> Department Number: 100 Department Name: Finance Manager Id: 108 Location Id: 1700
> ```

```sql
DECLARE
    TYPE dept_table_type IS TABLE OF departments%ROWTYPE INDEX BY PLS_INTEGER;
    my_dept_table  dept_table_type;
    loop_count     NUMBER(2) := 10;
    deptno         NUMBER(4) := 0;
BEGIN
    FOR i IN 1..loop_count
    LOOP
        deptno := 10 + deptno;
        SELECT * INTO my_dept_table(i)
        FROM departments
        WHERE department_id = deptno;
    END LOOP;

    FOR j IN 1..loop_count
    LOOP
        DBMS_OUTPUT.PUT_LINE('Department Number: ' || my_dept_table(j).department_id ||
                          ' Department Name: ' || my_dept_table(j).department_name ||
                          ' Manager Id: ' || my_dept_table(j).manager_id ||
                          ' Location Id: ' || my_dept_table(j).location_id);
    END LOOP;
END;
```

## Extra exercise 1

### Part 1

> Maak tabel/collection met als key een nr en als waarde een last\_name  
>     stop er op plaats 5 en 10 een naam in  
>     druk vervolgens alles af van plaats 1 – 10  \(als het bestaat\).

```sql
DECLARE
  TYPE ename_table_type IS TABLE OF employees.last_name%TYPE
  INDEX BY PLS_INTEGER;
  v_ename_table ename_table_type;
BEGIN
  v_ename_table(5):= 'King';
  v_ename_table(10):= 'Cameron';

  FOR i in 1..10 LOOP
    IF v_ename_table.exists(i)
      THEN
      DBMS_OUTPUT.PUT_LINE(v_ename_table(i));
    ELSE
      DBMS_OUTPUT.PUT_LINE('De rij met index '|| i ||' bestaat niet.');
    END IF;
  END LOOP;
END;
```

### Part 2

> Maak tabel/collection met als key een voornaam en als waarde een datum  
>     stop er als datum voor An in een week na vandaag en voor Jan in een maand na vandaag laat een naam ingeven en druk \(als bestaat\) de bijhorende datum af.

```sql
DECLARE
  TYPE ename_table_type IS TABLE OF employees.last_name%TYPE INDEX BY PLS_INTEGER;
  TYPE datum_table_type IS TABLE OF DATE INDEX BY VARCHAR2(20);

  v_ename_table   ename_table_type;
  v_datum_table   datum_table_type;
  v_voornaam      varchar2(20);
  v_index         PLS_INTEGER;
  v_index_datum   VARCHAR2(20);

BEGIN
  v_ename_table(5):= 'King';
  v_ename_table(10):= 'Cameron';

  FOR i in 1..10 LOOP
    IF v_ename_table.exists(i)
      THEN
        DBMS_OUTPUT.PUT_LINE(v_ename_table(i));
      ELSE
        DBMS_OUTPUT.PUT_LINE('De rij met index '|| i ||'  bestaat niet.');
      END IF;
  END LOOP;

  v_datum_table('An'):=SYSDATE + 7;
  v_datum_table('Jan') := add_months(SYSDATE,1);
  v_index_datum :=v_datum_table.first;

  WHILE v_index_datum is not null LOOP
    DBMS_OUTPUT.PUT_LINE(v_datum_table(v_index_datum));
    v_index_datum :=v_datum_table.next(v_index_datum);
  END LOOP;
END;
```

### Part 3

> Maak tabel/collection waarin de namen \(last\_name\) van alle werknemers uit de tabel employees worden opgenomen in alfabetische volgorde druk deze namen onder elkaar af. Druk aantal records in de datum\_table. Druk alle indexen en bijhorende waarden van de ename\_table

```sql
DECLARE
  TYPE lastname_table_type IS TABLE OF employees.last_name%TYPE INDEX BY PLS_INTEGER;
  TYPE datum_table_type IS TABLE OF DATE INDEX BY VARCHAR2(20);
  TYPE ename_table_type IS TABLE OF employees.last_name%TYPE INDEX BY PLS_INTEGER;

  v_ename_table       ename_table_type;
  v_aantal_rijen      INTEGER;
  v_datum_table       datum_table_type;
  v_voornaam          VARCHAR2(20);
  v_index             PLS_INTEGER;
  v_index_datum       VARCHAR2(20);
  v_lastname_table    lastname_table_type;

BEGIN
  v_ename_table(5):= 'King';
  v_ename_table(10):= 'Cameron';

  FOR i in 1..10 LOOP
    IF v_ename_table.exists(i)
      THEN
        DBMS_OUTPUT.PUT_LINE(v_ename_table(i));
    ELSE
        DBMS_OUTPUT.PUT_LINE('De rij met index '|| i ||'  bestaat niet.');
    END IF;
  END LOOP;

  v_datum_table('An'):= SYSDATE + 7;
  v_datum_table('Jan') := add_months(SYSDATE,1);
  v_index_datum :=v_datum_table.first;

  WHILE v_index_datum IS NOT NULL LOOP
    DBMS_OUTPUT.PUT_LINE(v_datum_table(v_index_datum));
    v_index_datum :=v_datum_table.next(v_index_datum);
  END LOOP;

  SELECT last_name BULK COLLECT
  INTO v_lastname_table
  FROM employees
  ORDER BY 1;

  FOR I in 1..v_lastname_table.count LOOP
    DBMS_OUTPUT.PUT_LINE(v_lastname_table(i));
  END LOOP;

  DBMS_OUTPUT.PUT_LINE('Het aantal records in de tabel: '||v_datum_table.COUNT);

  v_aantal_rijen:= v_ename_table.count;
  v_index:= v_ename_table.first;

  WHILE v_index IS NOT NULL LOOP
    DBMS_OUTPUT.PUT_LINE(v_index ||'  '||v_ename_table(v_index));
    v_index:= v_ename_table.next(v_index);
  END LOOP;
END;
```

## Extra exercise 2

> Gebruik een collection om bepaalde gegevens van alle departementen die gevestigd zijn in de US af te drukken.

```sql
DECLARE
  TYPE dept_table_type IS TABLE OF departments%ROWTYPE INDEX BY PLS_INTEGER;

  my_dept_table dept_table_type;
  v_index_dept pls_integer;
  f_loop_count NUMBER(4);
  v_deptno departments.department_id%type :=0;
  v_dep_id v_deptno%type;
BEGIN
  SELECT d.* BULK COLLECT
  INTO my_dept_table
  FROM departments d
  JOIN locations l
  ON d.location_id = l.location_id
  WHERE country_id = 'US';

/* Let op: in bovenstaande select krijg je wel een foutmelding als je in de join using(location_id) wil gebruiken*/
  FOR i IN 1 .. my_dept_table.count LOOP   
      DBMS_OUTPUT.PUT_LINE('Dept number : '|| my_dept_table(i).department_id||
                          'Dept name : '|| my_dept_table(i).department_name||
                          'Manager id : '|| my_dept_table(i).manager_id||
                          'Location id : '||my_dept_table(i).location_id);
  END LOOP;
END;
```



