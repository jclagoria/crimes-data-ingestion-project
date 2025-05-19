
# Chicago Crime Ingestion System

This project ingests and processes public crime data from the City of Chicago, sourced from the Chicago Police Department's CLEAR system. It efficiently handles large CSV files (>1.8GB), detects newly appended rows, stores raw data in MongoDB, and normalizes data into PostgreSQL for querying.

## Features

- ✅ Scheduled background file ingestion (Spring Scheduler)
- ✅ Two-phase ingestion: Raw (MongoDB) → Normalized (PostgreSQL)
- ✅ Reactive REST + GraphQL APIs with filtering and full-text search
- ✅ Full Clean Architecture + Hexagonal layering
- ✅ Geospatial querying support via PostGIS
- ✅ Resilient: Supports retries, archiving, and state tracking

## Architecture Overview

```
[Scheduler] --> [CSV Reader Adapter] --> [Raw MongoDB]
                                        --> [Normalizer] --> [PostgreSQL]
                                                          --> [REST / GraphQL APIs]
```

## Tech Stack

- Java 21
- Spring Boot (WebFlux, Data MongoDB Reactive, Data R2DBC)
- MongoDB for raw ingestion
- PPostgreSQL (with R2DBC) + PostGIS for normalized data
- OpenCSV (CSV streaming)
- GraphQL Java Kickstart

## Project Structure

```
src/main/java/com/example/crime
├── application       # Use cases (ingest, sync)
├── domain            # Models and value objects
├── interface         # API (REST, GraphQL), Ports
├── infrastructure    # Adapters (CSV, Mongo, Postgres)
└── scheduler         # File poller
```

## Configuration

```yaml
# application.yml
app:
  file-ingestion:
    path: /data/crimes
    filename: crimes.csv
    schedule: "0 0/15 * * * *"  # Every 15 minutes

spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/crimes
    username: crimes_user
    password: secret

  data:
    mongodb:
      uri: mongodb://localhost:27017/chicago-crimes
```

## Setup

1. Clone the repository
2. Ensure MongoDB and PostgreSQL (with PostGIS) are running
3. Configure `application.yml` with correct paths and DB settings
4. Build and run the application

```bash
./mvnw spring-boot:run
```

## Endpoints

- REST: `GET /crimes?year=2023&primaryType=THEFT`
- GraphQL: `/graphql` → Query by type, year, geolocation
- File watcher: polls for new files every 15 minutes

## Development Tips

- Use Docker for MongoDB and PostgreSQL locally
- Enable PostGIS: `CREATE EXTENSION postgis;`
- Use Testcontainers for integration testing
- Archive ingested files under `/data/crimes/archive/`

## License

MIT License. Data sourced from City of Chicago open data portal.
