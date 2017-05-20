# 2 Declaring PLSQL Variables

## Exercise 2

> Create an anonymous block. Use the script of question 2 of practice 1 p.1-22.
>
> A\) Add a \*\*declarative\*\* section to this PL/SQL block.In the declarative section declare the following variables:
>
> 1. Variable \`v\_today\` of type \`DATE\`. Initialize today with \`SYSDATE\`.
>
> 2. Variable \`v\_tomorrow\` of type \`v\_today\`. use \`%TYPE\` attribute to declare this variable.
>
> B\) In the \*\*executable\*\* section, initialize the \`v\_tomorrow\` variable with an expression, which calculates tomorrow's date \(add one to the value in today\). Print the value of \`v\_today\` and \`v\_tomorrow\` after printeng "Hello World".

```sql
DECLARE
  v_msg       VARCHAR(50)   :=  'Hello World';
  v_today     DATE          :=  SYSDATE;
  v_tomorrow  v_today%TYPE;
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_msg);
  DBMS_OUTPUT.PUT_LINE('Today is: ' || v_today);
  v_tomorrow:= v_today + 1;
  DBMS_OUTPUT.PUT_LINE('Tomorrow is: ' || v_tomorrow);
END;
```

## Exercise 5

> Edit the script of exercise 2.
>
> A\) Add code to create two bind variables. Create bind variables \`b\_basic\_percent\` and \`b\_pf\_percent\` of type \`NUMBER\`.
>
> B\) In the executable section of the PL/SQL block, assign the values 45 and 12 to \`b\_basic\_percent\` and \`b\_bf\_percent\`, respectively.
>
> C\) Execute your script. Sample outut is as follows:

```
Hello World  
TODAY IS : 16-MAY-07
TOMORROW IS : 17-MAY-07

>basic_percent  
--
45  

>pr_percent
--
12
```

```sql
VARIABLE  b_basic_percent  NUMBER;
VARIABLE  b_pf_percent     NUMBER;
DECLARE
  v_msg       VARCHAR(50)     := 'Hello World';
  v_today     DATE            := SYSDATE;
  v_tomorrow  v_today%TYPE;
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_msg);
  DBMS_OUTPUT.PUT_LINE('TODAY IS' || v_today);
  v_tomorrow := v_today + 1;
  DBMS_OUTPUT.PUT_LINE('TOMORROW IS' || v_tomorrow);
  :b_pf_percent    := 12;
  :b_basic_percent := 45;
END;
```

## Extra oefening 1

> Maak een bind variable om een department\_name te stockeren.
>
> Maak een anoniem blok dat het volgende doet:
>
> * haal de departementsnaam op van het departement met de meeste employees en vang deze op in de bind variable 
> * Toon van dit departement de department\_name en de naam, voornaam en salaris van de werknemer met het hoogste salaris. Druk de inhoud van de bind variable af.

```sql
SET SERVEROUTPUT ON
VARIABLE b_dept_name VARCHAR2(20);
DECLARE
  v_dept_name   departments.department_name%type;
  v_max_emp     NUMBER;
  v_count       NUMBER;
  v_lastname    employees.last_name%type;
  v_voornaam    employees.first_name%type;
  v_salary      employees.salary%type;
  v_max_sal     NUMBER(9,2);
BEGIN
  --meeste employee
  SELECT MAX(COUNT(employee_id))
  INTO v_max_emp
  FROM employees
  GROUP BY department_id;
  DBMS_OUTPUT.PUT_LINE('Meeste employees zijn : '||v_max_emp);
  /*
  haal de departementsnaam op van het departement met de 
  meeste employees en vang deze op in de bind variable
  */
  SELECT d.department_name ,
    COUNT(e.employee_id)
  INTO v_dept_name ,
    v_count
  FROM employees e
  JOIN departments d
  ON(e.department_id = d.department_id)
  GROUP BY d.department_name
  HAVING COUNT(e.employee_id) =  v_max_emp;
  :b_dept_name              :=  v_dept_name;
  /*Zet variable v_dept_name over in b_dept_name*/

  DBMS_OUTPUT.PUT_LINE('Department name:  ' ||v_dept_name);

  /*
  Toon van dit departement de department_name en de naam, 
  voornaam en salaris van de werknemer met het hoogste salaris.
  */
  SELECT MAX(e.salary)
  INTO v_max_sal
  FROM employees e
  JOIN departments d
  ON(e.department_id      = d.department_id)
  WHERE d.department_name = v_dept_name;
  DBMS_OUTPUT.PUT_LINE('Hoogste salaris : '|| v_max_sal);
  -- naam voornaam salary of employees met highest salary
  SELECT e.last_name ,
    e.first_name,
    e.salary
  INTO v_lastname ,
    v_voornaam,
    v_salary
  FROM employees e
  JOIN departments d
  ON(e.department_id      = d.department_id)
  WHERE d.department_name = v_dept_name
  AND e.salary            = v_max_sal;
  DBMS_OUTPUT.PUT_LINE(' Naam: '||v_lastname || 
                        ' Voornaam :  ' || v_voornaam || 
                        ' Salaris:  ' || v_salary || 
                        ' Department naam : '||v_dept_name );
END;
/
PRINT b_dept_name
```



