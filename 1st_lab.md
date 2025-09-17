Airport Database — ERD Documentation (text for README)

OVERVIEW
This repository contains an Entity-Relationship Diagram (ERD) and the related schema design for a simplified international airport system. The database manages airports, flights, airlines, gates, passengers, bookings, payments, boarding passes, luggage and checks (security and baggage). The text below identifies entities, attributes (with suggested data types), primary and foreign keys, uniqueness and required constraints, normalization notes (3NF), relationships with cardinalities, and SQL DDL examples you can paste into a README or use as a starting point.

1. ENTITIES & ATTRIBUTES (with suggested data types)
   Note: suggested types are PostgreSQL-style. Adjust types for other RDBMS if needed.

AIRPORT

* airport\_id (PK) : integer PRIMARY KEY GENERATED (serial / identity)
* name : varchar(255) NOT NULL
* country : varchar(100) NOT NULL
* state : varchar(100) NULL
* city : varchar(100) NULL
* year\_built : smallint NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

AIRLINE

* airline\_id (PK) : integer PRIMARY KEY
* code : varchar(10) NOT NULL UNIQUE
* name : varchar(255) NOT NULL
* country : varchar(100) NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

GATE

* gate\_id (PK) : integer PRIMARY KEY
* gate\_number : varchar(20) NOT NULL
* terminal : varchar(50) NULL
* airport\_id (FK → AIRPORT.airport\_id) : integer NOT NULL
* status : varchar(20) NULL   -- e.g. 'active','inactive'
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

FLIGHT

* flight\_id (PK) : integer PRIMARY KEY
* airline\_id (FK → AIRLINE.airline\_id) : integer NOT NULL
* departure\_airport\_id (FK → AIRPORT.airport\_id) : integer NOT NULL
* arrival\_airport\_id (FK → AIRPORT.airport\_id) : integer NOT NULL
* departure\_gate\_id (FK → GATE.gate\_id) : integer NULL
* arrival\_gate\_id (FK → GATE.gate\_id) : integer NULL
* scheduled\_departure : timestamptz NOT NULL
* scheduled\_arrival : timestamptz NOT NULL
* actual\_departure : timestamptz NULL
* actual\_arrival : timestamptz NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

PASSENGER

* passenger\_id (PK) : integer PRIMARY KEY
* first\_name : varchar(100) NOT NULL
* last\_name : varchar(100) NOT NULL
* gender : varchar(10) NULL
* date\_of\_birth : date NULL
* citizenship\_country : varchar(100) NULL
* residence\_country : varchar(100) NULL
* passport\_number : varchar(50) NOT NULL UNIQUE
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

BOOKING

* booking\_id (PK) : integer PRIMARY KEY
* booking\_reference : varchar(50) NOT NULL UNIQUE
* flight\_id (FK → FLIGHT.flight\_id) : integer NOT NULL
* passenger\_id (FK → PASSENGER.passenger\_id) : integer NOT NULL
* status : varchar(30) NOT NULL   -- e.g. 'confirmed','cancelled','hold'
* platform : varchar(50) NULL     -- e.g. 'website','agent'
* ticket\_price : numeric(10,2) NULL
* seat\_preference : varchar(20) NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

BOOKING\_CHANGE

* change\_id (PK) : integer PRIMARY KEY
* booking\_id (FK → BOOKING.booking\_id) : integer NOT NULL
* change\_type : varchar(50) NOT NULL   -- e.g. 'seat\_change','date\_change'
* change\_details : text NULL
* changed\_by : varchar(100) NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

PAYMENT

* payment\_id (PK) : integer PRIMARY KEY
* booking\_id (FK → BOOKING.booking\_id) : integer NOT NULL
* payment\_reference : varchar(100) NOT NULL UNIQUE
* payment\_status : varchar(30) NOT NULL   -- 'paid','pending','refunded'
* payment\_method : varchar(50) NULL
* payment\_date : timestamptz NULL
* amount : numeric(10,2) NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

BOARDING\_PASS

* pass\_id (PK) : integer PRIMARY KEY
* booking\_id (FK → BOOKING.booking\_id) : integer NOT NULL
* boarding\_pass\_number : varchar(100) NOT NULL UNIQUE
* seat : varchar(10) NULL
* boarding\_time : timestamptz NULL
* gate\_id (FK → GATE.gate\_id) : integer NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

LUGGAGE

* baggage\_id (PK) : integer PRIMARY KEY
* tag\_number : varchar(100) NOT NULL UNIQUE
* booking\_id (FK → BOOKING.booking\_id) : integer NOT NULL
* weight\_kg : numeric(6,2) NULL
* dimensions : varchar(100) NULL   -- e.g. "55x35x20 cm"
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

LUGGAGE\_CHECKING

* check\_id (PK) : integer PRIMARY KEY
* baggage\_id (FK → LUGGAGE.baggage\_id) : integer NOT NULL
* check\_result : varchar(50) NULL   -- e.g. 'ok','inspect','confiscated'
* checked\_by : varchar(100) NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()
  Note: passenger\_id was present in the diagram. To keep normalization (3NF), passenger info can be derived via luggage -> booking -> passenger. If you want faster direct access, you can add passenger\_id here as a FK, but that introduces redundancy (see normalization section).

SECURITY\_CHECK

* security\_id (PK) : integer PRIMARY KEY
* passenger\_id (FK → PASSENGER.passenger\_id) : integer NOT NULL
* check\_result : varchar(50) NULL
* checked\_by : varchar(100) NULL
* checked\_at : timestamptz NULL
* created\_at : timestamptz NOT NULL DEFAULT now()
* updated\_at : timestamptz NOT NULL DEFAULT now()

2. PRIMARY & FOREIGN KEYS, UNIQUE AND REQUIRED ATTRIBUTES

* Primary keys: all tables have primary keys as listed (e.g., airport\_id, flight\_id, passenger\_id, booking\_id, etc.)
* Foreign keys:

  * FLIGHT.airline\_id → AIRLINE.airline\_id
  * FLIGHT.departure\_airport\_id → AIRPORT.airport\_id
  * FLIGHT.arrival\_airport\_id → AIRPORT.airport\_id
  * FLIGHT.departure\_gate\_id, FLIGHT.arrival\_gate\_id → GATE.gate\_id (optional)
  * GATE.airport\_id → AIRPORT.airport\_id
  * BOOKING.flight\_id → FLIGHT.flight\_id
  * BOOKING.passenger\_id → PASSENGER.passenger\_id
  * BOOKING\_CHANGE.booking\_id → BOOKING.booking\_id
  * PAYMENT.booking\_id → BOOKING.booking\_id
  * BOARDING\_PASS.booking\_id → BOOKING.booking\_id
  * BOARDING\_PASS.gate\_id → GATE.gate\_id
  * LUGGAGE.booking\_id → BOOKING.booking\_id
  * LUGGAGE\_CHECKING.baggage\_id → LUGGAGE.baggage\_id
  * SECURITY\_CHECK.passenger\_id → PASSENGER.passenger\_id
* Unique attributes: passport\_number (PASSENGER), booking\_reference (BOOKING), payment\_reference (PAYMENT), boarding\_pass\_number (BOARDING\_PASS), tag\_number (LUGGAGE), airline.code (AIRLINE.code)
* Required (NOT NULL) attributes have been marked above where reasonable, e.g., PKs, booking\_reference, passenger linking fields; timestamps (created\_at) set default now()

4. RELATIONSHIPS & CARDINALITY

* AIRPORT 1 — \* FLIGHT (as departure): one airport can have many departing flights. Each flight departs from exactly one airport. Participation: airport (optional for airport record creation), flight (required).
* AIRPORT 1 — \* FLIGHT (as arrival): similarly for arrival.
* AIRPORT 1 — \* GATE: one airport has many gates; each gate belongs to one airport.
* AIRLINE 1 — \* FLIGHT: an airline operates many flights; each flight is operated by one airline.
* FLIGHT 1 — \* BOOKING: one flight can have many bookings (many passengers); each booking refers to one flight.
* PASSENGER 1 — \* BOOKING: a passenger can have many bookings; each booking is for exactly one passenger record.
* BOOKING 1 — 0..1 BOARDING\_PASS: a booking can have one boarding pass (or none if not issued yet).
* BOOKING 1 — \* PAYMENT: a booking can have multiple payments (instalments/refunds); each payment is linked to one booking.
* BOOKING 1 — \* LUGGAGE: a booking can have multiple baggage items; each baggage item belongs to one booking.
* LUGGAGE 1 — \* LUGGAGE\_CHECKING (or 1 — 0..\*): a single baggage tag may be checked multiple times (scans), or checked once depending on process.
* PASSENGER 1 — \* SECURITY\_CHECK: a passenger can have multiple security checks (different dates/flights), each security\_check links to one passenger.


5. ENTITY BOXES

AIRPORT

* PK: airport\_id (integer)
* name (varchar, required)
* country (varchar)
* state (varchar)
* city (varchar)
* year\_built (smallint)
* created\_at, updated\_at (timestamps)

6. CONNECTIONS (how to describe relationships in text)
   Write relations as lines like:
   FLIGHT.departure\_airport\_id (FK) → AIRPORT.airport\_id  (CARDINALITY: FLIGHT many → AIRPORT one)
   BOOKING.flight\_id (FK) → FLIGHT.flight\_id  (CARDINALITY: BOOKING many → FLIGHT one)
   BOOKING.passenger\_id (FK) → PASSENGER.passenger\_id  (CARDINALITY: BOOKING many → PASSENGER one)
   BOARDING\_PASS.booking\_id (FK) → BOOKING.booking\_id  (CARDINALITY: BOARDING\_PASS many? Usually one → BOOKING one)

7. BRIEF DESCRIPTIONS / ENTITY PURPOSE (short keys)

* AIRPORT: stores airports (location and metadata).
* AIRLINE: airline companies operating flights.
* GATE: physical gate points tied to airports.
* FLIGHT: scheduled flight instances (links to airline and airports).
* PASSENGER: passenger personal data.
* BOOKING: reservation for a passenger on a flight.
* BOOKING\_CHANGE: historical record of changes to bookings.
* PAYMENT: payment transactions for bookings.
* BOARDING\_PASS: issued boarding passes tied to bookings and gates.
* LUGGAGE: baggage registered for bookings.
* LUGGAGE\_CHECKING: results/history of baggage checks.
* SECURITY\_CHECK: security checks performed on passengers.