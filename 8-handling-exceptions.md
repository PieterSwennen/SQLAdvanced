# 8 Handling Exceptions

## Exception - Example

```sql
DECLARE
  v_lname VARCHAR2(15);
BEGIN
  SELECT last_name
  INTO v_lname
  FROM employees
  WHERE first_name = 'John';
  DBMS_OUTPUT.PUT_LINE('John''s last name is : ' || v_lname);
EXCEPTION
  WHEN TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE('Your select statement retrieved multiple rows. Consider using a cursor');
END;
```

## Non predefined error - Example

```sql
DECLARE
  e_insert_excep EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_insert_excep, -01400);
BEGIN
  INSERT INTO departments( department_id,
                              department_name
                            )
      VALUES(280, NULL);

EXCEPTION
  WHEN e_insert_excep THEN
    DBMS_OUTPUT.PUT_LINE('Insert operation failed');
    DBMS_OUTPUT.PUT_LINE(SQLERRM);

END;
```

## Trapping user-definded Exceptions - Example

```sql
DECLARE
  v_deptno                NUMBER        := 500;
  v_name                  VARCHAR2(20)  := 'Testing';
  e_invalid_department    EXCEPTION;

BEGIN
  UPDATE departments
  SET department_name = v_name
  WHERE department_id = v_deptno;

  IF SQL % NOTFOUND THEN
    RAISE e_invalid_department;
  END IF;
  COMMIT;  
EXCEPTION
  WHEN e_invalid_department
    THEN
      DBMS_OUTPUT.PUT_LINE('No such department id.');
END;
```

## Extra Exercise 1

> Schrijf een anoniem PL/SQL-blok om:
>
> * een nieuwe functie toe te kunnen voegen in de tabel JOBS met gegevens vanaf het toetsenbord
> * als de functie al bestaat moet er een aangepaste foutmelding verschijnen
> * test je programma en doe telkens een rollback

```sql
DECLARE
BEGIN
  INSERT INTO jobs
  VALUES('&jobid', '&jobtitle', &laagstesal, &hoogstesal);

  EXCEPTION
    WHEN dup_val_on_index THEN
      DBMS_OUTPUT.PUT_LINE('Deze functie bestaat al.');
ROLLBACK;
END;
```

> Schrijf een anoniem PL/SQL-blok om:
>
> * het min. en max. salary samen met de job\_title te tonen van 1 bepaalde functie die wordt ingegeven vanaf het toetsenbord. Toon dit als volgt: `Het laagste en hoogste salaris van de job ….. is ….. en ……` Er moet een gepaste foutmelding verschijnen als een onbestaande functie wordt ingegeven.
> * het min. en max. salary samen met het job\_id te tonen van een job\_id dat begint met SA. Toon dit als volgt : `Het laagste en hoogste salaris van de job … is …. En ….`    Zorg voor gepaste foutmelding als een onbestaand id wordt ingegeven of als een id meermaals voor komt.
> * test je programma op al deze mogelijke fouten

```sql
DECLARE
       v_jobtitel jobs.job_title%type;
       v_minsal   jobs.min_salary%type;
       v_maxsal   jobs.max_salary%type;
       v_jobid    jobs.job_id%type;
BEGIN
       SELECT job_title, min_salary,max_salary
       INTO v_jobtitel,v_minsal,v_maxsal
       FROM jobs
       WHERE job_title='&jobtitel';

       dbms_output.put_line('Het laagste en hoogste salaris van de job ' ||v_jobtitel||' is '|| v_minsal||' en '|| v_maxsal);

       SELECT job_id, min_salary, max_salary
       INTO v_jobid,v_minsal,v_maxsal
       FROM jobs
       WHERE job_id like 'SA%';

       dbms_output.put_line('Het laagste en hoogste salaris van de job ' ||v_jobid||' is '|| v_minsal||' en '|| v_maxsal);

EXCEPTION
       WHEN no_data_found
        THEN dbms_output.put_line('Deze functie bestaat niet.');
       WHEN too_many_rows
        THEN dbms_output.put_line('De functie die begint met SA komt meer dan 1 keer voor!');
END;
```

## Extra Exercise 2

> Schrijf een anoniem PL/SQL-blok om:
>
> * een nieuw land toe te kunnen voegen in de tabel COUNTRIES met gegevens vanaf het toetsenbord
> * als dit land al bestaat moet er een aangepaste foutmelding verschijnen
> * er moet ook een aangepaste foutmelding getoond worden als er een land wordt ingegeven met een onbestaand region\_id
> * verwijder de regio met het id 1 uit de tabel REGIONS en zorg voor gepaste opvang indien een fout optreedt
> * test je programma op deze mogelijke fouten en doe telkens een rollback

```sql
DECLARE
  e_onbekende_regio exception;
  pragma exception_init(e_onbekende_regio, -02291);
  e_del_child exception;
  pragma exception_init(e_del_child, -02292);
BEGIN
  INSERT into countries
    values('&landId', '&landnaam', &regio);

  DELETE FROM regions
  WHERE region_id = 1;

EXCEPTION
  WHEN dup_val_on_index THEN
    DBMS_OUTPUT.PUT_LINE('Dit land bestaat al.');
  WHEN e_onbekende_regio THEN
    DBMS_OUTPUT.PUT_LINE('Regio_id komt niet voor in  tabel REGIONS');
  WHEN e_del_child THEN
    DBMS_OUTPUT.PUT_LINE('Regio_id kan niet verwijderd worden wegens verwijzingen vanuit table countries');
ROLLBACK;
END;
```

## Extra Excercise 3

> A. Open hfst8\_oef1\_2.sql en pas deze oefening aan zodat het programma niet correct eindigt \(dus afgebroken wordt\) indien een fout optreedt. Bewaar als hfst8\_oef3\_1.sql

```sql
DECLARE
       v_jobtitel jobs.job_title%type;
       v_minsal   jobs.min_salary%type;
       v_maxsal   jobs.max_salary%type;
       v_jobid    jobs.job_id%type;
BEGIN
       SELECT job_title, min_salary,max_salary
       INTO v_jobtitel,v_minsal,v_maxsal
       FROM jobs
       WHERE job_title='&jobtitel';
       dbms_output.put_line('Het laagste en hoogste salaris van de job ' ||v_jobtitel||' is '|| v_minsal||' en '|| v_maxsal);

       SELECT job_id, min_salary, max_salary
       INTO v_jobid,v_minsal,v_maxsal
       FROM jobs
       WHERE job_id like 'SA%';
       dbms_output.put_line('Het laagste en hoogste salaris van de job ' ||v_jobid||' is '|| v_minsal||' en '|| v_maxsal);

EXCEPTION
       WHEN no_data_found
       THEN raise_application_error(-20001,'Deze functie bestaat niet.');

       WHEN too_many_rows
       THEN raise_application_error(-20002,'De functie die begint met SA komt meer dan 1 keer voor!');
END;
```

> B. Creëer een logtabel ERRORS met volgende attributen e\_user, e\_code, e\_date en e\_message.  
> Schrijf een anoniem PL/SQL-blok om:
>
> * in de tabel JOBS het maximum salary van alle managers te verhogen naar 20000
> * als er geen aanpassingen gebeuren in deze tabel dan moet dit gemeld worden
> * als er meer dan 5 aanpassingen zijn dan moet dit gemeld worden en mag de wijziging niet door gaan
> * schrijf alle niet opgevangen fouten naar de logtabel ERRORS
> * zorg ervoor dat je programma niet correct eindigt \(dus afgebroken wordt\) indien er een fout optreedt
> * test je programma op al deze mogelijke fouten
> * bewaar als hfst8\_oef3\_2.sql

```sql
CREATE TABLE errors
(e_user       VARCHAR2(10) CONSTRAINT e_pk PRIMARY KEY,
e_code        VARCHAR2(20) NOT NULL,
e_date        DATE NOT NULL,
e_message     VARCHAR2(100) NOT NULL);

DECLARE
       e_geen_update            exception;
       e_update_teveel_rijen    exception;
       error_code               NUMBER;
       error_message            VARCHAR2(100);
BEGIN
       UPDATE jobs
       SET max_salary = 20000
       WHERE job_title LIKE '%Manager%';
       IF sql%notfound THEN
                     raise e_geen_update;
       ELSIF sql%rowcount > 5 THEN
                     raise e_update_teveel_rijen;
       END IF;
EXCEPTION
       WHEN e_geen_update then
                     raise_application_error(-20003,'Er werden geen rijen geupdated!');
       WHEN e_update_teveel_rijen THEN
                     ROLLBACK;
                     raise_application_error(-20004,'Teveel rijen geupdated. Update wordt niet uitgevoerd!');
       WHEN others THEN
                     error_code := sqlcode;
                     error_message :=sqlerrm;
                     INSERT INTO errors
                     VALUES(user,error_code,SYSDATE,error_message);
END;
```



