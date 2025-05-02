== Implementation

=== Project Setup

Technologies:
- Java 21
- Spring Boot (Reactive) with WebFlux
- Spring Data MongoDB Reactive (raw ingestion)
- Spring Data JPA (PostgreSQL with PostGIS)
- Spring Scheduler (cron-based background polling)
- OpenCSV (for efficient line-by-line CSV reading)
- GraphQL Java Kickstart (GraphQL endpoint)
- MapStruct and Lombok (DTO mapping and boilerplate reduction)

---


---

=== MongoDB (Raw Data Schema)

**Collection**: `raw_crime_data`

```json
{
  "crime_id": Number,
  "case_number": String,
  "date": ISODate,
  "block": String,
  "iucr": String,
  "primary_type": String,
  "description": String,
  "location_description": String,
  "arrest": Boolean,
  "domestic": Boolean,
  "beat": Number,
  "district": Number,
  "ward": Number,
  "community_area": Number,
  "fbi_code": String,
  "x_coordinate": Number,
  "y_coordinate": Number,
  "year": Number,
  "updated_on": ISODate,
  "latitude": Number,
  "longitude": Number,
  "location": {
    "type": "Point",
    "coordinates": [longitude, latitude]
  },
  "ingestion_metadata": {
    "ingested_at": ISODate,
    "source_file": String
  },
  "postgres_synced": Boolean
}

Indexes:

- { crime_id: 1 } (unique)

- { year: 1, primary_type: 1, arrest: 1 }

- { location: "2dsphere" }

- Full-text: { description: "text", block: "text", location_description: "text" }

- { postgres_synced: 1 }


=== PostgreSQL Normalized Schema

CREATE TABLE primary_types (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE descriptions (
    id SERIAL PRIMARY KEY,
    primary_type_id INT REFERENCES primary_types(id),
    text TEXT NOT NULL
);

CREATE TABLE location_descriptions (
    id SERIAL PRIMARY KEY,
    text TEXT UNIQUE NOT NULL
);

CREATE TABLE fbi_codes (
    id SERIAL PRIMARY KEY,
    code VARCHAR(10) UNIQUE NOT NULL
);

CREATE TABLE community_areas (
    id INT PRIMARY KEY,
    name TEXT
);

CREATE TABLE crime_data (
    id BIGINT PRIMARY KEY,
    case_number VARCHAR(20),
    date TIMESTAMP,
    block TEXT,
    iucr VARCHAR(10),
    primary_type_id INT REFERENCES primary_types(id),
    description_id INT REFERENCES descriptions(id),
    location_description_id INT REFERENCES location_descriptions(id),
    arrest BOOLEAN,
    domestic BOOLEAN,
    beat INT,
    district INT,
    ward INT,
    community_area_id INT REFERENCES community_areas(id),
    fbi_code_id INT REFERENCES fbi_codes(id),
    x_coordinate BIGINT,
    y_coordinate BIGINT,
    year INT,
    updated_on TIMESTAMP,
    latitude DOUBLE PRECISION,
    longitude DOUBLE PRECISION,
    location GEOGRAPHY(Point, 4326),
    ingested_at TIMESTAMPTZ DEFAULT NOW()
);

Indexes:

- Composite: (year, primary_type_id)

- Full-text: GIN (to_tsvector('english', block))

- Geo: GIST (location) using PostGIS

=== Ingestion Flow

Phase 1 – Raw Ingestion to MongoDB

Scheduled with Spring Scheduler (CrimeFilePoller).

Streams CSV using CsvCrimeFileReaderAdapter.

Skips rows where crime_id <= last_max_id (read from tracking table).

Inserts new records into raw_crime_data.

Updates last_max_id in a tracking collection/table.

Moves the file to /data/crimes/archive/crimes_<timestamp>.csv on success.

``` java
Files.move(
  Paths.get("/data/crimes/crimes.csv"),
  Paths.get("/data/crimes/archive/crimes_" + LocalDate.now() + ".csv"),
  StandardCopyOption.REPLACE_EXISTING
);
```
Phase 2 – Normalize and Insert into PostgreSQL

Queries MongoDB for postgres_synced = false.

Resolves FKs from lookup tables.

Inserts into crime_data using PostgresCrimeRepositoryAdapter.

Updates MongoDB documents: set postgres_synced = true.

=== API Layer

CrimeRestController: REST endpoints using Spring WebFlux.

CrimeGraphQLController: GraphQL endpoint using GraphQL Java Kickstart.

Exposes filters by year, primary type, arrest, etc.

Supports location-based search (PostgreSQL + PostGIS).

=== Scheduler Configuration (YAML)

```yaml
app:
  file-ingestion:
    path: /data/crimes
    filename: crimes.csv
    schedule: "0 0/15 * * * *"  # Every 15 minutes

spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/chicago-crimes
  datasource:
    url: jdbc:postgresql://localhost:5432/crimes
    username: crimes_user
    password: secret

```
This implementation ensures:

Separation of ingestion and normalization for maintainability and traceability.

Full recovery and retry support.

Clean, testable, and scalable system based on Hexagonal/Clean architecture.

Efficient querying via indexes and normalization.


