# 9 Triggers

## Extra voorbeeld 1A

>Doe het nodige om ervoor te zorgen dat de gebruiker student geen gegevens uit de tabel employees kan verwijderen en dat deze gebruiker geen aanpassingen kan doen aan het loon van de medewerkers. Toon in beide gevallen de melding “U heeft niet voldoende rechten om deze actie uit te voeren!”. Bewaar als d2_h9_oef1a.sql.

```sql
CREATE OR REPLACE trigger bdus_emp
  BEFORE DELETE OR UPDATE OF salary ON employees
BEGIN
  IF user = 'STUDENT'
    THEN raise_application_error (
         -20000,'U heeft niet voldoende rechten om deze actie uit te voeren!'
         );
  END IF;
END;
```

>Test met delete:

```sql
DELETE FROM employees WHERE last_name = 'Fox';
```

>Test met update:

```sql
UPDATE employees
SET salary = 15000
WHERE last_name = 'Higgins';
```
<div style="page-break-after: always;"></div>
## Extra voorbeeld 1B
>Schrijf een database trigger waardoor bijgehouden wordt wie, wanneer, welke bewerking(insert, update of delete) op de tabel job_history heeft uitgevoerd. Maak daarvoor de tabel log_history aan met de volgende kolommen: log_user, log_datum, log_tijd, log_actie. Bewaar als d2_h9_oef1b.sql.

```sql
CREATE TABLE log_history ( log_user VARCHAR2(30),
                           log_datum DATE,
                           log_tijd TIMESTAMP,
                           log_actie varchar2(15) )
```

```sql
CREATE OR REPLACE TRIGGER aiud_jhis
  AFTER INSERT OR UPDATE OR DELETE ON job_history

DECLARE
  v_actie varchar2(15) ;

BEGIN
  IF INSERTING
    THEN v_actie := 'INSERT';
  ELSIF UPDATING
    THEN v_actie := 'UPDATE';
  ELSE v_actie := 'DELETE';
  END IF;

INSERT INTO log_history
  VALUES(user, SYSDATE, systimestamp, v_actie);
END;
```

>Test met:

```sql
INSERT INTO job_history
VALUES(127,'14-JAN-99', SYSDATE, 'ST_CLERK', 50);
```

>Inhoud tabel log_history

```sql
SELECT * FROM log_history;
```
<div style="page-break-after: always;"></div>
## Extra voorbeeld 2
>Doe al het nodige om er voor te zorgen dat bij het wijzigen van het salaris van een of meerdere medewerkers de verhoging nooit meer mag zijn dan 5%(melding indien meer dan 5%: “Een loonsverhoging van meer dan 5 % is niet toegelaten” en verhoging mag niet doorgaan) en een verlaging resulteert in een foutmelding “Loonsverlaging kan niet!” en deze mag dus ook niet doorgaan. Bewaar als d2_h9_oef2.sql.

```sql
CREATE OR REPLACE TRIGGER aur_emp
  AFTER UPDATE of salary ON employees
  FOR EACH ROW
BEGIN
  If :new.salary < :old.salary
    THEN raise_application_error(
         -20000,
         'Loonsverlaging kan niet!'
         );
  ELSIF :new.salary > :old.salary*1.05
    THEN raise_application_error(
         -20000,
         'Een loonsverhoging van meer dan 5 % is niet toegelaten!'
         );

  END IF;
END;
```

>Test met update

```sql
UPDATE employees
SET salary = 5000
WHERE employee_id = 115;
```

> Je krijgt foutmelding omdat onze 1ste trigger op de tabel employees een statement trigger is en deze wordt uitgevoerd vóór de rijtrigger(zie hb p 9-24). Willen we de rijtrigger testen dan moeten we de 1ste trigger tijdelijk uitschakelen via ALTER TRIGGER bdus_emp DISABLE (‘ALTER TABLE employees DISABLE ALL TRIGGERS’ kan ook maar dan worden alle triggers gekoppeld aan de tabel employees tijdelijk uitgeschakeld). Vergeet dan niet na het testen de trigger terug in te schakelen via ALTER TRIGGER bdus_emp ENABLE of ALTER TABLE employees ENABLE ALL TRIGGERS.

<div style="page-break-after: always;"></div>
## Extra voorbeeld 3
>Pas de oplossing van oefening 2 zodanig aan dat deze trigger enkel werkt als het gaat om de medewerkers die aangeworven zijn vóór 01-01-1995. Bewaar als d2_h9_oef3.sql.

```sql
CREATE OR REPLACE TRIGGER aur_emp
  AFTER UPDATE of salary ON employees
  FOR EACH ROW
  WHEN (old.hire_date < to_date('01-01-1995','dd-mm-yyyy'))
BEGIN
  If :new.salary < :old.salary
    THEN raise_application_error(
         -20000,
         'Loonsverlaging kan niet!'
         );
  ELSIF :new.salary > :old.salary*1.05
    THEN raise_application_error(
         -20000,
         'Een loonsverhoging van meer dan 5 % is niet toegelaten!'
         );
  END IF;
END;
```

<div style="page-break-after: always;"></div>
## Extra voorbeeld 4
>Doe al het nodige om er voor te zorgen dat:
- bij het toevoegen of wijzigen in de tabel employees het job_id automatisch naar hoofdletters wordt geconverteerd en de naam en voornaam enkel begint met een hoofdletter
- bij het toevoegen bij de hire_date nooit een datum in het verleden kan worden ingegeven
- medewerkers die promoveren en een managersfunctie krijgen, automatisch een salarisverhoging krijgen van 5%. Het moet wel degelijk om een promotie gaan. Iemand die president of vice president was en manager wordt, krijgt geen verhoging.

```sql
CREATE OR REPLACE TRIGGER biur_emp
  BEFORE INSERT OR UPDDATE ON employees

FOR EACH ROW
BEGIN
    :new.job_id     := upper(:new.job_id);
    :new.last_name  := initcap(:new.last_name);
    :new.first_name := initcap(:new.first_name);

    IF INSERTING AND :new.hire_date < SYSDATE
      THEN RAISE_APPLICATION_ERROR (-20000,'Nieuwe aanwerfdatum kan niet in het verleden liggen!');
    END IF;

    IF (:new.job_id LIKE '%MAN' OR :new.job_id LIKE '%MGR')
        AND :old.job_id NOT IN ('AD_PRES','AD_VP')
      THEN :new.salary := :old.salary * 1.05;
    END IF;
END biur_emp;
```
> Test:

```sql
INSERT INTO employees
VALUES(207,'pieter','MAES','PMAES@pxl.be','650.124.3336',to_date('30-04-2015','dd-mm-yyyy'),'sa_rep',3000,null,148,80)
```
