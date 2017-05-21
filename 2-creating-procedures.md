# 2 Creating Procedures

## Procedure - Example

```sql
CREATE TABLE dept AS
SELECT * FROM DEPARTMENTS;
CREATE PROCEDURE add_dept
IS
  v_dept_id     dept.department_id%TYPE;
  v_dept_name   dept.department_name%TYPE;
BEGIN
  v_dept_id   :=280;
  v_dept_name :='ST-Curriculum';
  INSERT
  INTO dept
    (
      department_id,
      department_name
    )
    VALUES
    (
      v_dept_id,
      v_dept_name
    );
  DBMS_OUTPUT.PUT_LINE('Inserted '|| SQL%ROWCOUNT || 'row');
END;
```

> To execute

```sql
BEGIN
  add_dept;
END;
```

> Or

```sql
EXECUTE add_dept;
```
<div style="page-break-after: always;"></div>
## Using the `IN` parameter Mode - Example

```sql
CREATE OR REPLACE PROCEDURE raise_salary(
    p_id      IN employees.employee_id%TYPE,
    p_percent IN NUMBER)
IS
BEGIN
  UPDATE employees
  SET salary        = salary * (1 + p_percent/100)
  WHERE employee_id = p_id;
END raise_salary;
```

```sql
EXECUTE raise_salary(176,10)
```
<div style="page-break-after: always;"></div>
## Using the 'OUT' parameter mode - Example

```sql
CREATE OR REPLACE PROCEDURE query_emp
( p_id      IN  employees.employee_id%TYPE,
  p_name    OUT employees.last_name%TYPE,
  p_salary  OUT employees.salary%TYPE)
IS
BEGIN
  SELECT last_name, salary INTO p_name, p_salary
  FROM employees
  WHERE employee_id = p_id;
END query_emp;
/
```

```sql
DECLARE
  v_emp_name  employees.last_name%TYPE;
  v_emp_sal   employees.salary%TYPE;
BEGIN
  query_emp(171, v_emp_name, v_emp_sal);
  DBMS_OUTPUT.PUT_LINE(
    v_emp_name|| 
    ' earns ' || 
    to_char(v_emp_sal, '$999,999.00')
  );
END;
```

<div style="page-break-after: always;"></div>
## Deel2: procedures: voorbeeldopgaven

### Opgave 1

> Schrijf een procedure ‘toon\_eerste\_emp’ die in de medewerkerstabel zoekt naar de eerst aangeworven medewerker. Toon het nummer, de naam en de aanwervingsdatum van deze medewerker.

```sql
CREATE OR REPLACE PROCEDURE toon_eerste_emp
IS
  v_emp_id  employees.employee_id%TYPE;
  v_naam    employees.last_name%TYPE;

BEGIN
  SELECT employee_id, last_name
  INTO v_emp_id, v_naam
  FROM employees
  WHERE hire_date = (SELECT MIN(hire_date)
                     FROM employees);

  DBMS_OUTPUT.PUT_LINE(
    'Eerst aangeworven employee: '|| v_emp_id || 
    '' || v_naam
  );

END;
```

<div style="page-break-after: always;"></div>
### Opgave 2

> Schrijf een procedure die ervoor zorgt dat als in een bepaald land nieuwe wetgeving van kracht gaat ivm minimumlonen, de eventueel nodige salarisaanpassingen gemakkelijk in de databank doorgevoerd kunnen worden.  
> Invoerparameters: de landnaam en het minimumloon dat daar geldig is.  
> Uitvoerparameter: aantal gewijzigd records.

```sql
CREATE OR REPLACE PROCEDURE minimumlonen
  (p_landnaam IN countries.country_name%TYPE,
  p_minimum   IN employees.salary%TYPE,
  p_aantal    OUT number)
AS
BEGIN
  UPDATE employees
  SET salary = p_minimum
  WHERE department_id IN
    (SELECT department_id
     FROM departments
     WHERE location_id IN
      (SELECT location_id
       FROM locations
       WHERE country_id IN
        (SELECT country_id
         FROM countries
         WHERE country_name = p_landnaam)))
  AND salary < p_minimum;
  p_aantal:= sql%rowcount;
END;
```

> Execute

```sql
variable aantal number
set autoprint on
exec minimumlonen('Canada', 7000, :aantal);
```
<div style="page-break-after: always;"></div>
### Opgave 3

> Schrijf een procedure die de naam van de procedure toont die het laatst aangemaakt werd.  
> Toon eveneens de broncode van de laatst aangemaakte procedure.  
> Denk eraan dat een SELECT-statement altijd 1 rij moet opleveren.  
> Tip: maak een lus waarin je het lijnnr laat varieren van 1 t.e.m. het hoogste lijnnr van de betreffende procedure.

```sql
CREATE OR REPLACE PROCEDURE show_latest_procedure
IS
  TYPE user_source_table_text_type 
    IS TABLE OF user_source.text%TYPE INDEX BY PLS_INTEGER;

  v_timestamp                 user_objects.timestamp%TYPE;
  v_object_name               user_objects.object_name%TYPE;
  v_source_code               user_source.text%TYPE;
  v_user_source_table_text    user_source_table_text_type;

BEGIN
  SELECT object_name, TIMESTAMP
  INTO v_object_name, v_timestamp
  FROM user_objects
  WHERE object_type = 'PROCEDURE'
  AND TIMESTAMP = ( SELECT MAX(TIMESTAMP) 
                    FROM user_objects 
                    WHERE object_type = 'PROCEDURE');

  DBMS_OUTPUT.PUT_LINE('Datum: ' || v_timestamp);
  DBMS_OUTPUT.PUT_LINE('------------------------------');

  SELECT text BULK COLLECT
  INTO v_user_source_table_text
  FROM user_source
  WHERE name = v_object_name;

  FOR i in 1.. v_user_source_table_text.COUNT loop
    DBMS_OUTPUT.PUT_LINE(v_user_source_table_text(i));
  END LOOP;
END;
```

> Model oplossing 1:

```sql
IS
   v_laatst_gem_proc     user_objects.object_name%type;
   v_line                user_source.line%type;
   v_text                varchar2(90);
   v_maxlijnnr           number(2);
BEGIN
   SELECT object_name
   INTO v_laatst_gem_proc
   FROM user_objects
   WHERE created = (SELECT max(created)
                    FROM user_objects);

   DBMS_OUTPUT.PUT_LINE(
     'De laatst aangemaakte procedure is '|| 
      v_laatst_gem_proc
    );

   SELECT MAX(line)                      -- code volledig tonen
      INTO v_maxlijnnr
      FROM user_source
      WHERE name = v_laatst_gem_proc
      GROUP BY name;

   FOR i_lijnnr IN 1 .. v_maxlijnnr
   LOOP
      SELECT line, text
         INTO v_line, v_text
         FROM user_source
         WHERE name = v_laatst_gem_proc AND LINE = i_lijnnr;
      DBMS_OUTPUT.PUT_LINE(v_line || v_text);
   END loop;
END;
```

> Model oplossing 2:

```sql
CREATE OR REPLACE PROCEDURE TOON_LAATSTE_PROCEDURE_BIS
IS
  TYPE type_col_lijnnr_code   
    IS TABLE OF type_rec_lijnnr_code INDEX BY PLS_INTEGER;
  TYPE type_rec_lijnnr_code   
    IS RECORD(line user_source.line%type, text user_source.text%type);

  v_laatst_gem_proc   user_objects.object_name%type ;
  r_lijnnr_code       type_rec_lijnnr_code;

BEGIN
   SELECT object_name
      INTO v_laatst_gem_proc
      FROM user_objects
      WHERE created = (SELECT max(created) FROM user_objects);
      DBMS_OUTPUT.PUT_LINE(
        'De laatst aangemaakte procedure is '
        ||v_laatst_gem_proc
      );

   SELECT line, text BULK COLLECT
      INTO t_lijnnr_code
      FROM user_source
      WHERE name = v_laatst_gem_proc;

   FOR i_lijnnr IN 1 .. t_lijnnr_code.count
   LOOP
      DBMS_OUTPUT.PUT_LINE(
        t_lijnnr_code(i_lijnnr).line||
        ' '||t_lijnnr_code(i_lijnnr).text
      );
   END LOOP;
END;
```



