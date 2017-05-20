# 4 Interacting with the oracle database server

## Exercise 1

> Create a PL/SQL block that selects the maximum department ID in the \`departments\` table and store it in the \`v\_max\_deptno\` variable. Display the maximum department ID.
>
> A\) Declare a variable, \`v\_max\_deptno\` of type \`NUMBER\` in the \*\*declarative\*\* section.
>
> B\) Start the executable section with the \`BEGIN\` keywoard and include a \`SELECT\` statement to retrieve the maximum \`department\_id\` from the departments table.
>
> C\) Display \`v\_max\_deptno\` and end the executable block.
>
> D\) Execute your script. Sample output is as follows:

```
The maximum department_id is : 270
```

```sql
DECLARE
  v_max_deptno departments.department_id%TYPE;
BEGIN
  SELECT MAX(department_id)
  INTO v_max_deptno
  FROM departments;
  DBMS_OUTPUT.PUT_LINE('THE maximum department_id is : ' || v_max_deptno);
END;
```
<div style="page-break-after: always;"></div>
## Exercise 2

> Modify the PL/SQL block you create in exercise 1 to insert a new department in the departments table.
>
> A\) Load exercise 1 script. Declare two variables:  
> \`v\_dept\_name\` of type \`departements.departement\_name\`  
> \`v\_dept\_id\` of type \`NUMBER\`  
> Assign "Education" to \`v\_dept\_name\` in the declarative section.
>
> B\) You have already retrieved the current maximum departement ID from the departments table. Add 10 to it and assign the result to v\_dept\_id.
>
> C\) Include an \`INSERT\` statement to insert data into the \`departement\_name\`, \`departement\_id\` and \`location\_id\` columns of the departments table. Use values \`v\_dept\_name\` and \`v\_dept\_id\` for \`department\_name\` and \`department\_id\`, respectively, and use \`NULL\` for \`location\_id\`.
>
> D\) Use the SQL attribute \`SQL%ROWCOUNT\` to display the number of rows that are affected.
>
> E\) Execute a \`SELECT\` statement to check whether the new department is inserted. You can terminate the PL/SQL block with "/" and include the \`SELECT\` statement in your script.
>
> F\) Execute your scipt. Sample output is as follows:

```
The maximum department_id is : 280
SQL%ROWCOUNT gives 1
```

```sql
DECLARE
  v_max_deptno  departments.department_id%TYPE;
  v_dept_id     departments.department_id%TYPE;
  v_dept_name   departments.department_name%TYPE  :=  'Education';
BEGIN
  SELECT MAX(department_id)
  INTO v_max_deptno
  FROM departments;

  v_dept_id := 10 + v_max_deptno;

  INSERT
  INTO departments
    (
      department_name,
      department_id,
      location_id
    )
    VALUES
    (
      v_dept_name,
      v_dept_id,
      NULL
    );
  DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' is inserted');
END;
```
<div style="page-break-after: always;"></div>
## Exercise 3

> In exercise 2, you set loation+id to \`NULL\`. Create a PL/SQL block that updates the \`location\_id\` to 3000 for the new depatment. Us the local variable \`v\_dept\_id\` to update the row.  
> **Note**: Skip step A if you have not started a new session for this practice.
>
> A\) If you have started a new session, delete the department that you have added to the \`departments\` table and execute the exercise 2 script.
>
> B\) Start the executable block with the \`BEGIN\` keyword. Include the \`UPDATE\` statement to set the \`location\_id\` to 3000 for the new department\(\`v\_dept\_id=280\`\).
>
> C\) End the executable block with the \`END\` keyword. Terminate the PL/SQ: block with "/" and include a \`SELECT\` statement to display the department that you updated.
>
> D\) Finally, include a \`DELETE\` statement to delete the deparment that you added.
>
> E\) Execute your script. Sample output is as follows:

```
DEPARTMENT_ID   DEPARTMENT_NAME   MANAGER_ID    LOCATION_ID
-------------   ---------------   ----------    -----------
280             Education                       3000
```

```sql
DECLARE
  v_max_deptno  departments.department_id%TYPE;
  v_dept_id     departments.department_id%TYPE;
  v_dept_name   departments.department_name%TYPE := 'Education';
BEGIN
  SELECT MAX(department_id)
  INTO v_max_deptno
  FROM departments;

  v_dept_id := 10 + v_max_deptno;

  INSERT
  INTO departments
    (
      department_name,
      department_id,
      location_id
    )
    VALUES
    (
      v_dept_name,
      v_dept_id,
      NULL
    );
  UPDATE departments
  SET location_id = 3000
  WHERE department_id = v_dept_id;
END;
       /
SELECT *
FROM departments
WHERE department_id =
  (SELECT MAX(department_id) FROM departments);
DELETE
FROM departments
WHERE department_id =
  (SELECT MAX(department_id) FROM departments);
```

## Extra Oefening 1

```sql
DROP    TABLE emp; --Only when table emp already exist.
CREATE  TABLE emp AS SELECT*FROM employees;
ALTER   TABLE emp ADD stars VARCHAR2(50);
```

```sql
DECLARE
  v_sal       emp.salary%TYPE;
  v_empno     emp.employee_id%TYPE  :=  176;
  v_asterisk  emp.stars%TYPE        :=  NULL;
BEGIN
  SELECT nvl(round(salary/1000),0)
  INTO v_sal
  FROM emp
  WHERE employee_id = v_empno;

  FOR i in 1..v_sal
    LOOP
      v_asterisk:= v_asterisk||'*';
    END LOOP;

  UPDATE emp
  SET stars = v_asterisk
  WHERE employee_id = v_empno;
COMMIT;
END;
```

> Test command:

```sql
SELECT employee_id, salary, stars
FROM emp
WHERE employee_id=176;
```
<div style="page-break-after: always;"></div>
## Extra Oefening 2

> Er moeten besparingen doorgevoerd worden in de firma.
>
> * Schrijf een anoniem blok dat het volgende doet:   De afdelingshoofden van afdelingen waar minder dan 6 personeelsleden werken krijgen een salarisverlaging van 10%
> * Druk af hoeveel salarisaanpassingen gebeurden.

```sql
BEGIN
    UPDATE employees
     SET salary = salary * 0.9
     WHERE employee_id IN (SELECT manager_id
                        FROM departments
                        WHERE department_id IN (SELECT department_id
                                                FROM departments
                                                JOIN employees
                                                USING (department_id)
                                                GROUP BY  department_id
                                                HAVING COUNT(*) <= 5)
                      );
DBMS_OUTPUT.PUT_LINE('Aantal salarisverlagingen : ' || SQL%ROWCOUNT);
END;
```



