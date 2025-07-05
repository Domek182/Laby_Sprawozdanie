Modele konceptualny, logiczny, fizyczny
===============================================

:author:        - Piotr Domagała
                - Tomasz Oszczęda

Wprowadzenie
---------------
W tym rozdziale opisano projektowanie bazy danych dla biblioteki zawierającej informacje o jej klientach oraz książkach. Będzie to m.in. przejście przez modele konceptualny, logiczny i fizyczny oraz opis przechowywanych danych.

Model konceptualny 
---------------------
Encje

**1.użytkownicy:**

- PESEL 
- imię
- nazwisko
- adres zamieszkania 
- adres mailowy
- liczba wypożyczonych książek

**2. książki:**

- unikalny indeks 
- nazwa książki
- autor książki
- przez kogo została wypożyczona 
- czas wypożyczenia w dniach

Związki:

- przez kogo wypożyczona książka 
- ilość wypożyczonych książek przez użytkownika

Związki niepoprawne:

- nie ma określonej maksymalnej liczby wypożyczoncych książek więc, jeden użytkownik może wypożyczyć naraz wszyskie książki
- kilka osób może mieć ten sam adres mailowy

Przedstawienie za pomocą notacji Chena

.. image:: NotacjaChena.png
   :width: 668px
   :height: 891px
   :align: left

Model logiczny
-----------------
Aby zwiększyć poprawność danych stosuje się proces normalizacji. W przypdaku tej bazy danych została ona znormalizowana do stopnia 3, czyli spełnia następujące warunki:

- Każde pole w tabeli powinno być atomowe
- Wszystkie rekordy w tabeli muszą mieć tą samą strukturę 
- Brak powtarzających się kolumn zawierających podobne dane 
- Każdy atrybut który nie jest kluczem głównym musi być od niego w pełni zależny
- Atrybuty niekluczowe nie mogą zależeć od innych atrybutów niekluczowych

Przedstawienie bazy danych w notacji Barkera (tekstowo)

.. code-block:: none

 |========================|
 |      Uzytkownicy       |
 |------------------------|
 | PK  PESEL              |
 |     Imie (40)          |
 |     Nazwisko (40)      |
 |     AdresZamieszkania(40)|
 |     AdresMailowy(40)   |
 |     LiczbaWypozyczonychKsiazek(INT, max 15)|
 |========================|

  (0,N)                             (0,15)
 Uzytkownicy --------------------- Ksiazki

 |========================|
 |        Ksiazki         |
 |------------------------|
 | PK  ISBN               |
 |     NazwaKsiazki (40)  |
 |     AutorKsiazki (40)  |
 | FK  PESEL (opcjonalne) |
 |     CzasWypozyczenia (INT, max 60)|
 |========================|


Model fizyczny
-----------------

Implentacja dla PostgreSQL oraz SQLite

**PostgreSQL**

.. code-block:: sql
 
 CREATE TABLE IF NOT EXISTS użytkownicy (
         imie VARCHAR(40),
         nazwisko VARCHAR(40),
         adres_zamieszkania VARCHAR(40),
         PESEL CHAR(11) PRIMARY KEY,
         adres_mailowy VARCHAR(40),
         liczba_wypozyczonych_ksiazek SMALLINT CHECK (liczba_wypozyczonych_ksiazek BETWEEN 0 AND 15)
 );

   
 CREATE TABLE IF NOT EXISTS książki (
         nazwa_ksiazki VARCHAR(40),
         autor_ksiazki VARCHAR(40),
         indeks CHAR(13) PRIMARY KEY,
         pesel CHAR(11),
         czas_wypozyczenia SMALLINT CHECK (czas_wypozyczenia BETWEEN 0 AND 60),
         FOREIGN KEY (pesel) REFERENCES użytkownicy(PESEL) ON DELETE SET NULL
 );

**SQLite**

.. code-block:: sql

 CREATE TABLE IF NOT EXISTS użytkownicy (
         imie TEXT,
         nazwisko TEXT,
	 adres_zamieszkania TEXT,
	 PESEL TEXT PRIMARY KEY,
	 adres_mailowy TEXT,
	 liczba_wypozyczonych_ksiazek INTEGER CHECK (liczba_wypozyczonych_ksiazek BETWEEN 0 AND 15)
 );
 CREATE TABLE IF NOT EXISTS książki (
	 nazwa_ksiazki TEXT,
	 autor_ksiazki TEXT,
	 indeks TEXT PRIMARY KEY,
	 pesel TEXT,
	 czas_wypozyczenia INTEGER CHECK (czas_wypozyczenia BETWEEN 0 AND 60),
	 FOREIGN KEY (pesel) REFERENCES użytkownicy(PESEL) ON DELETE SET NULL
 );
