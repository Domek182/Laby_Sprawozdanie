ANALIZA BAZY DANYCH I OPTYMALIZACJA ZAPYTAŃ CLAAI01
====================================================

:author:        - Piotr Domagała
                - Tomasz Oszczęda

Analiza normalizacji 
------------------------
1NF – Pierwsza postać normalna:

- Wszystkie atrybuty są atomowe (niepodzielne).
- Brak powtarzających się grup danych.
Obie tabele spełniają 1NF — wszystkie kolumny zawierają pojedyncze, niepodzielne wartości.

2NF – Druga postać normalna:

- Spełnia 1NF.
- Każdy atrybut niekluczowy jest w pełni funkcjonalnie zależny od całego klucza głównego.
Obie tabele mają pojedyncze klucze główne (`PESEL` i `indeks`), więc nie ma problemu z częściową zależnością. 2NF jest spełniona.

3NF – Trzecia postać normalna:

- Spełnia 2NF.
- Żaden atrybut niekluczowy nie jest przechodnio zależny od klucza głównego.

Analiza tabeli `użytkownicy`:

- `PESEL` → `imie`, `nazwisko`, `adres_zamieszkania`, `adres_mailowy`, `liczba_wypozyczonych_ksiazek`
- Wszystkie atrybuty zależą bezpośrednio od klucza głównego.
- Brak przechodnich zależności.
Spełnia 3NF.

Analiza tabeli `książki`:

- `indeks` → `nazwa_ksiazki`, `autor_ksiazki`, `pesel`, `czas_wypozyczenia`
- `pesel` jest kluczem obcym, ale nie ma przechodnich zależności (np. `autor_ksiazki` nie zależy od `pesel`).
Spełnia 3NF.

Model danych jest w trzeciej postaci normalnej (3NF).

Potencjalne problemy wydajnościowe
-----------------------------------
Problemy:

- Brak indeksów poza kluczami głównymi. Częste wyszukiwanie po kolumnach takich jak indeks w tabeli książki lub PESEL w użytkownicy może prowadzić do spadku wydajności.
- Brak indeksów może prowadzić do pełnych skanów tabel (full table scan), co spowalnia zapytania przy dużej liczbie rekordów.

Strategie optymalizacji
------------------------
Rodzaje strategi:

- Indeksy

.. code-block:: sql
 
 CREATE INDEX idx_ksiazki_indeks    ON książki(indeks);
 CREATE INDEX idx_uzytkownicy_pesel     ON użytkownicy(PESEL);

- W przypadku naszej bazy danych partycjonowanie tabel nie przyniesie korzyści.

-  Widoki materializowane \-> \ jeśli często sprawdzamy, które książki są wypożyczone, a które nie to możemy utworzyć widok materializowany

.. code-block:: sql

 CREATE MATERIALIZED VIEW widok_status_ksiazek AS
 SELECT
     k.indeks,
     k.nazwa_ksiazki,
     k.autor_ksiazki,
     CASE
         WHEN k.pesel IS NULL THEN 'Dostępna'
         ELSE 'Wypożyczona'
     END AS status,
     u.imie,
     u.nazwisko
 FROM książki k
 LEFT JOIN użytkownicy u ON k.pesel = u.PESEL;

4. Optymalizacja zapytań SQL z użyciem EXPLAIN i indeksów:

- Unikanie SELECT \*\, wybieranie konkretnych kolumn.
- Używanie EXPLAIN ANALYZE (PostgreSQL).

EXPLAIN ANALYZE:

- Wykonuje zapytanie.
- Zwraca szczegółowy plan wykonania (tzw. execution plan).
- Pokazuje rzeczywiste czasy wykonania każdego kroku.
- Pokazuje liczbę wierszy przetworzonych na każdym etapie.
- Umożliwia identyfikację wąskich gardeł i nieefektywnych operacji.
- Używanie EXPLAIN QUERY PLAN (SQLite) 

EXPLAIN QUERY PLAN:

- Pokazuje, czy zapytanie użyje indeksu, czy wykona pełny skan tabeli.
- Informuje, czy zostanie użyta tymczasowa struktura danych (np. do GROUP BY lub ORDER BY).
- Pomaga zidentyfikować wąskie gardła w zapytaniu.
- Umożliwia optymalizację zapytań bez potrzeby ich uruchamiania.

Chcemy wykonać zapytanie

.. code-block:: sql

 SELECT u.imie, u.nazwisko, COUNT(*) AS cnt
 FROM książki k
 JOIN użytkownicy u ON k.pesel = u.PESEL
 WHERE k.czas_wypozyczenia BETWEEN 1 AND 50
 GROUP BY u.imie, u.nazwisko;
W celu dowiedzenia się jak przyspieszyć zapytanie używamy  

.. code-block:: none
 EXPLAIN ANALYZE/ EXPLAIN QUERY PLAN

W wyniku użycia tych narzędzi diagnostycznych dowiadujemy się, że PostgreSQL/SQLite wykonują pełny skan tabeli książki.  Aby temu zapobiec, tworzymy indeks wspierający filtrację po kolumnie czas_wypozyczenia:

.. code-block:: sql

 CREATE INDEX idx_ksiazki_czas_wypozyczenia ON książki(czas_wypozyczenia);

Dzięki temu silnik bazy danych może szybciej odnaleźć pasujące wiersze, co znacząco redukuje czas wykonania zapytania.
