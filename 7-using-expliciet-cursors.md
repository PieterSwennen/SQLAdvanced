# 7 Using Explicit Cursors

## Cursor Loop - Example

```sql
DECLARE
  CURSOR c_emp_cursor
  IS
    SELECT last_name
    FROM employees
    WHERE department_id = 30;

  v_emp_name employees.last_name%TYPE;
BEGIN
  OPEN c_emp_cursor;
  LOOP
    FETCH c_emp_cursor INTO v_emp_name;
    EXIT
  WHEN c_emp_cursor%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(v_emp_name);
  END LOOP;
  CLOSE c_emp_cursor;
END;
```
<div style="page-break-after: always;"></div>
## Fetching data from the cursor - Example

```sql
DECLARE
  CURSOR c_emp_cursor
  IS
    SELECT employee_id, last_name
    FROM employees
    WHERE department_id = 30;

  v_empno employees.employee_id%TYPE;
  v_lname employees.last_name%TYPE;
BEGIN
  OPEN c_emp_cursor;
  FETCH c_emp_cursor INTO v_empno, v_lname;
  DBMS_OUTPUT.PUT_LINE(v_empno || ' ' || v_lname);
END;
```

## Cursor FOR loop - Example

```sql
DECLARE
  CURSOR c_emp_cursor
  IS
    SELECT employee_id, last_name
    FROM employees
    WHERE department_id = 30;
BEGIN
  FOR emp_record IN c_emp_cursor
  LOOP
    DBMS_OUTPUT.PUT_LINE( emp_record.employee_id ||' '|| 
                          emp_record.last_name);
  END LOOP;
END;
```

## Cursor parameter - Example

```sql
DECLARE
  CURSOR c_emp_cursor (deptno NUMBER)
  IS
    SELECT last_name
    FROM employees
    WHERE department_id = deptno;
  v_emp_last_name employees.last_name%TYPE;
BEGIN
  OPEN c_emp_cursor (10);
  FETCH c_emp_cursor INTO v_emp_last_name;
  DBMS_OUTPUT.PUT_LINE(v_emp_last_name);
  CLOSE c_emp_cursor;
END;
```



