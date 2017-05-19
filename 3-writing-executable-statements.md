# H3 Writing Executable Statements

## Sequences p.9

> Creër de sequence reg\_regid\_seq om automatisch een volgnummer te kunnen geven aan een nieuwe region. Start bij 5 en verhoog telkens met 1

```sql
CREATE SEQUENCE reg_regid_seq INCREMENT BY 1 START WITH 5;
```

```sql
SELECT reg_regid_seq.nextval FROM dual
```

> Toon via een anoniem PL/SQL blok het huidige en volgende id voor region.

```sql
DECLARE
  v_curr_id regions.region_id%type NOT NULL :=  reg_regid_seq.currval;
  v_new_id  regions.region_id%type NOT NULL :=  reg_regid_seq.nextval;
BEGIN
  dbms_output.put_line(‘Het huidige region_id is ‘|| v_curr_id);
  dbms_output.put_line(‘Het volgende region_id is ‘|| v_new_id);
END;
```

## Extra oefening p.12

> Gebruik een anoniem PL/SQL blok om volgende dingen uit te voeren:  
> Voeg in de tabel job\_history een rij toe met volgende gegevens die pas worden gevraagd bij uitvoering van het blok: employee\_id = 184, start\_date = 20/9/2014, end\_date in te vullen, job\_id = ST\_CLERK en department\_id =50  
> Toon nadien  de gegevens uit deze rij op onderstaande wijze op het scherm:  
> Employee 184 is gestart op 20 september 2014 in het departement 50.  
> Zorg er ook voor dat de rij niet definitief wordt toegevoegd.

```sql
DECLARE
  v_emp_id      job_history.employee_id%type    :=  &employee_id;
  v_start_datum job_history.start_date%type     :=  to_date('&startdatum','dd/mm/yyyy');
  v_eind_datum  job_history.end_date%type       :=  to_date('&einddatum','dd/mm/yyyy');
  v_job_id      job_history.job_id%type         :=  '&job_id';
  v_dep_id      job_history.department_id%type  :=  &department_id;
BEGIN
  INSERT
  INTO job_history VALUES
    (
      v_emp_id,
      v_start_datum,
      v_eind_datum,
      v_job_id,
      v_dep_id
    );
  DBMS_OUTPUT.PUT_LINE('Employee '||v_emp_id|| 
                        ' is gestart op '|| TO_CHAR(v_start_datum,'d month yyyy')|| 
                        ' in het departement ' || v_dep_id);
END;
ROLLBACK;
```

## Exercise 3

> Edit the script of exercise 5 in chapter 2.  
>  A\) Us single-line comment syntax to comment the lines that create the bind variables.
>
> B\) Use multiple-line comments in the executable section to comment the lines that assign values to the bind variables.
>
> C\) Declare the \`v\_basic\_percent\` and \`v\_pf\_percent\` variables and initialize them to 45 and 12, respectively.  
>      Also, declare  two variables: \`v\_fname\` of type \`VARCHAR2\` and size 15, and \`v\_emp\_sal\` of type \`NUMBER\` and size 10.
>
> D\) Include the following SQL statement in the executable section:
>
> ```sql
> SELECT first_name, salary
> INTO v_fname, v_emp_sal
> FROM employees
> WHERE employee_id=110;
> ```
>
> E\) Change the line that prints "Hello World" to print "Hello" and the first name. You can comment the lines that display  
>      the dates and print the bind variables, if you want to.
>
> F\) Calculate the contribution of the employee toward provident fund \(PF\).  
>      PF is 12% of the basic salary and basic salary is 45% of the salary. Use the local variables for the calculation. Try and use  
>      only one expression to calculate the PF. Print the employee's salary and his contribution toword PF.
>
> Execute your script. Sample output is as follows:
>
> ```
> Hello John
> YOUR SALARY IS : 8200
> YOUR CONTRIBUTION TOWARDS PF : 442.8
> ```

```sql
--VARIABLE b_basic_percent NUMBER;
--VARIABLE b_pf_percent NUMBER;
DECLARE
  v_today         DATE          :=  SYSDATE;
  v_tomorrow      v_today%TYPE;
  v_basic_percent NUMBER(10)    :=  45;
  v_pf_percent    NUMBER(10)    :=  12;
  v_fname         VARCHAR2(15);
  v_emp_sal       NUMBER(10);
BEGIN
  SELECT first_name,
    salary
  INTO v_fname,
    v_emp_sal
  FROM employees
  WHERE employee_id=110;
  DBMS_OUTPUT.PUT_LINE('Hello ' || v_fname);
  DBMS_OUTPUT.PUT_LINE('YOUR SALARY IS : ' || v_emp_sal);
  DBMS_OUTPUT.PUT_LINE('YOUR CONTRIBUTION TOWARDS PF : ' || ((v_emp_sal * (v_basic_percent/100)) * (v_pf_percent/100)));
  /*
  DBMS_OUTPUT.PUT_LINE('TODAY IS' || v_today);
  v_tomorrow := v_today + 1;
  DBMS_OUTPUT.PUT_LINE('TOMORROW IS' || v_tomorrow);
  :b_pf_percent := 12;
  :b_basic_percent := 45;
  */
END;
```



