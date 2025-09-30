-- 1. Airport
CREATE TABLE airport (
  airport_id   INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  airport_name VARCHAR(255) NOT NULL,
  state        VARCHAR(255),
  country      VARCHAR(255),
  city         VARCHAR(255),
  created_at   TIMESTAMP NOT NULL DEFAULT now(),
  updated_at   TIMESTAMP
);

-- 2. Airline
CREATE TABLE airline (
  airline_id   INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  airline_code VARCHAR(10)  UNIQUE NOT NULL,
  airline_name VARCHAR(255) NOT NULL,
  country      VARCHAR(255) NOT NULL,
  created_at   TIMESTAMP NOT NULL DEFAULT now(),
  updated_at   TIMESTAMP
);

-- 3. Passenger
CREATE TABLE passenger (
  passenger_id           INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  first_name             VARCHAR(50)  NOT NULL,
  last_name              VARCHAR(50)  NOT NULL,
  gender                 VARCHAR(10),
  date_of_birth          DATE         NOT NULL,
  country_of_citizenship VARCHAR(100) NOT NULL,
  country_of_residence   VARCHAR(100),
  passport_number        VARCHAR(50)  UNIQUE NOT NULL,
  created_at             TIMESTAMP    NOT NULL DEFAULT now(),
  updated_at             TIMESTAMP
);

-- 4. Flight
CREATE TABLE flight (
  flight_id                 INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  departing_gate            VARCHAR(50),
  arriving_gate             VARCHAR(50),
  created_at                TIMESTAMP NOT NULL DEFAULT now(),
  updated_at                TIMESTAMP,
  airline_id                INT NOT NULL REFERENCES airline(airline_id),
  departure_airport_id      INT NOT NULL REFERENCES airport(airport_id),
  arrival_airport_id        INT NOT NULL REFERENCES airport(airport_id),
  scheduled_departure_time  TIMESTAMP NOT NULL,
  scheduled_arrival_time    TIMESTAMP NOT NULL,
  actual_departure_time     TIMESTAMP,
  actual_arrival_time       TIMESTAMP
);

-- 5. Booking
CREATE TABLE booking (
  booking_id     INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  flight_id      INT NOT NULL REFERENCES flight(flight_id),
  status         VARCHAR(50) NOT NULL,
  booking_platform VARCHAR(50),
  created_at     TIMESTAMP NOT NULL DEFAULT now(),
  updated_at     TIMESTAMP,
  ticket_price   INT NOT NULL,
  passenger_id   INT NOT NULL REFERENCES passenger(passenger_id)
);

-- 6. Baggage
CREATE TABLE baggage (
  baggage_id  INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  weight_kg   INT NOT NULL,
  created_at  TIMESTAMP NOT NULL DEFAULT now(),
  updated_at  TIMESTAMP,
  booking_id  INT NOT NULL REFERENCES booking(booking_id)
);

-- 7. Boarding Pass
CREATE TABLE boarding_pass (
  boarding_pass_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  seat             VARCHAR(10) NOT NULL,
  boarding_time    TIMESTAMP   NOT NULL,
  created_at       TIMESTAMP   NOT NULL DEFAULT now(),
  updated_at       TIMESTAMP,
  booking_id       INT NOT NULL REFERENCES booking(booking_id)
);

-- 8. Booking Change
CREATE TABLE booking_change (
  booking_change_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  booking_id        INT NOT NULL REFERENCES booking(booking_id)
);

-- 9. Baggage Check
CREATE TABLE baggage_check (
  baggage_check_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  check_result     VARCHAR(50) NOT NULL,
  created_at       TIMESTAMP NOT NULL DEFAULT now(),
  updated_at       TIMESTAMP,
  booking_id       INT NOT NULL REFERENCES booking(booking_id),
  passenger_id     INT NOT NULL REFERENCES passenger(passenger_id),
  baggage_id       INT NOT NULL REFERENCES baggage(baggage_id)
);

-- 10. Security Check
CREATE TABLE security_check (
  security_check_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  check_result      VARCHAR(100),
  created_at        TIMESTAMP NOT NULL DEFAULT now(),
  updated_at        TIMESTAMP,
  passenger_id      INT NOT NULL REFERENCES passenger(passenger_id)
);
