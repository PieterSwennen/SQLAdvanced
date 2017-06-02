## Vraag 3
>Schrijf een procedure om het plannen van flitsacties te automatiseren.
Eerst wordt gecontroleerd of reeds in elke gemeente ooit een flitsactie gehouden werd.  

>Is dit niet zo, dan doe je het volgende:
* druk het aantal gemeenten af waar nog nooit een flitsactie plaatsvond
* de bedoeling is om in al deze gemeenten vrij snel een flitsactie te plannen (lees:
een record toe te voegen in de tabel flitsen). De eerste datum waarop een actie
doorgaat is de eerstvolgende zaterdag na 4 weken, en vanaf dan elke dag in een
andere gemeente tot alle gemeenten aan bod zijn gekomen. De gemeenten komen
aan bod in volgorde van hun postcode (laagste postcode eerst).  

>Is dit wel zo, dan doe je het volgende:
  * druk af dat alle gemeenten al ooit aan bod zijn gekomen
  * voor de gemeente waar het het langst geleden is dat er een flitsactie plaatsvond,
wordt een nieuwe gepland, nl op de eerstvolgende zaterdag na 4 weken.  
Als outputparameter stuur je het aantal toegevoegde records terug.
Maak hiervoor een bind variabele aan.

```sql
CREATE OR REPLACE PROCEDURE plan_flitsacties
( p_nieuwe_flitsacties  OUT NUMBER )
IS
  TYPE table_gemeente         
    IS TABLE OF gemeente%ROWTYPE INDEX BY PLS_INTEGER;
    
  v_geplande_datum            flitsen.wanneer%TYPE;
  v_gemeentes_niet_geflitst   table_gemeente;
  v_max_id                    NUMBER;
  v_postcode_langst           flitsen.waar%TYPE;
BEGIN
  SELECT g.* BULK COLLECT
  INTO v_gemeentes_niet_geflitst
  FROM gemeente g
  LEFT JOIN flitsen f
  ON f.WAAR = g.POSTCODE
  WHERE f.id IS NULL
  ORDER BY g.postcode;

  SELECT MAX(id)+1
  INTO v_max_id
  FROM flitsen;

  v_geplande_datum  := add_months(SYSDATE, 1);
  v_geplande_datum  := TRUNC(v_geplande_datum, 'iw') + 5;

  IF v_gemeentes_niet_geflitst.COUNT > 0
    THEN      
      DBMS_OUTPUT.PUT_LINE(
        'Aantal gemeente zonder flitsacties: '   
        ||  v_gemeentes_niet_geflitst.COUNT
        );
      DBMS_OUTPUT.PUT_LINE('Nieuw geplande flitsacties: ');
      DBMS_OUTPUT.NEW_LINE();

      FOR i in 1.. v_gemeentes_niet_geflitst.COUNT LOOP        
        INSERT INTO flitsen
        VALUES(v_max_id, 
               v_gemeentes_niet_geflitst(i).postcode,
               v_geplande_datum);
               
        v_geplande_datum := v_geplande_datum + 1;
        v_max_id := v_max_id + 1;

        DBMS_OUTPUT.PUT_LINE(
          v_gemeentes_niet_geflitst(i).gemeente 
          ||' '|| v_geplande_datum);
        DBMS_OUTPUT.PUT_LINE(
          v_gemeentes_niet_geflitst(i).postcode);
        DBMS_OUTPUT.NEW_LINE();

      END LOOP;
	p_nieuwe_flitsacties := v_gemeentes_niet_geflitst.COUNT;
  ELSE
      DBMS_OUTPUT.PUT_LINE('Alle gemeenten zijn reeds aan bod gekomen.');

      SELECT waar
      INTO v_postcode_langst
      FROM flitsen
      WHERE wanneer = ( SELECT MIN(wanneer)
                        FROM flitsen);

      INSERT INTO flitsen
      VALUES(v_max_id,
             v_postcode_langst,
             v_geplande_datum);

      DBMS_OUTPUT.PUT_LINE(
        'Nieuwe flitsactie voor: ' ||
	 v_postcode_langst || ' op '||
	 v_geplande_datum);
      p_nieuwe_flitsacties := SQL%ROWCOUNT;
  END IF;
ROLLBACK;
END;
```
>Gebruik:

```sql
DECLARE
  v_nieuwe_flitsacties   NUMBER;
BEGIN
  plan_flitsacties(v_nieuwe_flitsacties);
  DBMS_OUTPUT.PUT_LINE('Nieuwe flitsacties: '||v_nieuwe_flitsacties);
END;
```
<div style="page-break-after: always;"></div>
## Vraag 3 - Model Oplossing
```sql
CREATE OR REPLACE PROCDURE flitsplanning (p_toeg OUT NUMBER)
AS
	v_aantal        NUMBER(4);
	v_flitsdatum    DATE;
	v_postcode      gemeente.postcode%TYPE;
	v_minpostcode   gemeente.postcode%TYPE :=0;
	v_gemeente      gemeente.gemeente%TYPE;
BEGIN
  v_flitsdatum := sysdate + 28;
  v_flitsdatum := next_day(v_flitsdatum, 'sat');

  SELECT COUNT(*)
  INTO v_aantal
  FROM gemeente
  WHERE postcode
  NOT IN (SELECT waar
          FROM flitsen);

  IF v_aantal > 0
    THEN DBMS_OUTPUT.PUT_LINE(
					'Er zijn nog '||v_aantal||
					' gemeentes waar nooit geflitst werd! FLITSPLANNING:'
				 );

    FOR i IN 1 .. v_aantal LOOP
        SELECT postcode
        INTO v_postcode
        FROM gemeente
        WHERE postcode = (SELECT MIN(postcode)
                          FROM gemeente
                          WHERE postcode>v_minpostcode
                          AND postcode
                          NOT IN (SELECT waar
                                  FROM flitsen));

        v_minpostcode := v_postcode;

        INSERT INTO flitsen
        VALUES (
          flitsen_id_seq.nextval,
          v_postcode, 
          v_flitsdatum);

        v_flitsdatum := v_flitsdatum + 1;
      END LOOP;
      p_toeg := v_aantal;
    ELSE
		DBMS_OUTPUT.PUT_LINE('In elke gemeente werd al ooit geflitst. FLITSPLANNING:');

      SELECT waar
      INTO v_postcode
      FROM flitsen
      WHERE wanneer = (SELECT MIN(wanneer)
                       FROM flitsen);

      INSERT INTO flitsen
      VALUES (flitsen_id_seq.nextval,v_postcode, v_flitsdatum);
      p_toeg := sql%rowcount;
  END IF;
END;
```