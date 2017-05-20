# 3 Creating Functions
## Function - Example
```sql
CREATE OR REPLACE FUNCTION get_sal (p_id employees.employee_id%TYPE)
 RETURN NUMBER IS
  v_sal employees.salary%TYPE := 0;
BEGIN
  SELECT salary
  INTO   v_sal
  FROM   employees         
  WHERE  employee_id = p_id;
  RETURN v_sal;
END get_sal;
/
```
>Using a host variable to obtain the result

```sql
VARIABLE b_salary NUMBER
EXECUTE :b_salary := get_sal(100)
```

>Using a local variable to obtain the result

```sql
DECLARE sal employees.salary%type;
BEGIN
  sal := get_sal(100); ...
END;
```

>Use as a parameter to another subprogram

```sql
EXECUTE dbms_output.put_line(get_sal(100))
```

>Use in a SQL statement (subject to restrictions)

```sql
SELECT job_id, get_sal(employee_id) FROM employees;
```

## Function - Example
```sql
CREATE OR REPLACE FUNCTION tax(p_value IN NUMBER)
 RETURN NUMBER IS
BEGIN
   RETURN (p_value * 0.08);
END tax;
/
SELECT employee_id, last_name, salary, tax(salary)
FROM   employees
WHERE  department_id = 100;
```

## Named and mixed notation - Example
```sql
CREATE OR REPLACE FUNCTION f(
	p_parameter_1 IN NUMBER DEFAULT 1,
	p_parameter_5 IN NUMBER DEFAULT 5)
RETURN NUMBER IS
   v_var NUMBER;
BEGIN
  v_var := p_parameter_1 + (p_parameter_5 * 2);
  RETURN v_var;
END f;
```
>Execute

```sql
SELECT f(p_parameter_5 => 10) FROM DUAL;
```

## Functions: voorbeeldopgave
### Opgave 1
>Schrijf een functie ‘get_jubileumdatum’ om te berekenen wanneer een bepaalde werknemer 25 jaar in dienst is, en dus gevierd gaat worden.  Het feest vindt plaats op de eerste vrijdag volgend op de dag waarop hij 25 jaar in dienst is, tenzij dat net een vrijdag is, dan is het onmiddellijk feest.
Let op:  als deze datum al voorbij is wordt als resultaat van de functie teruggekeerd ‘werd reeds gevierd’.
Invoerparameter: familienaam van de werknemer waarvoor de jubileumsdatum opgezocht wordt.
Wanneer een naam ingegeven wordt van iemand die geen werknemer is, mag er geen fout optreden, maar wordt teruggegeven ‘onbekende WN’.

```sql
CREATE OR REPLACE FUNCTION get_jubileumdate(p_last_nameemployees.last_name%type)
  RETURN varchar2
  AS v_hire_date DATE;

  v_jubileum DATE;
BEGIN
  SELECT hire_date
  INTO v_hire_date
  FROM employees
  WHERE last_name = p_last_name;

  v_jubileum := add_months(v_hire_date , 25*12);

  IF to_char(v_jubileum, 'd') != 6
    THEN v_jubileum := next_day(v_jubileum, 'fri');
  END IF;

  IF v_jubileum < SYSDATE
    THEN RETURN 'reeds gevierd';
  ELSE RETURN TO_CHAR(v_jubileum);
  END IF;

EXCEPTION WHEN no_data_found THEN return 'onbekende wn';
END;
/
```
