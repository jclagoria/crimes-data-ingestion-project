== Requirements

*Must Have*
- Periodic background job to detect and read a CSV file of crime incidents.
- Detect and ingest only new data rows (append-only) from the source CSV.
- Store raw data in a database for historical reference.
- Expose reactive and non-blocking REST and GraphQL endpoints for querying data.
- Apply Hexagonal Architecture and SOLID design principles throughout.
- Handle large file size efficiently (1.8GB+).
- Store filtered/query-ready data in a relational database (e.g., PostgreSQL).

*Should Have*
- Store raw unfiltered data in a document database (e.g., MongoDB).
- Background job scheduling using Spring Scheduler.
- Efficient deduplication and tracking mechanism to detect new vs existing rows.
- Full-text search across all fields in the initial MVP.

*Could Have*
- Real-time metrics for ingestion status.
- Retry and error logging for ingestion failures.
- Configuration-driven file path and scheduling intervals.

*Won’t Have*
- Manual data editing in the application UI.

== Method

=== Architecture Overview

The application uses Clean/Hexagonal Architecture to separate domain logic from infrastructure. The core handles ingestion, transformation, and filtering of crime records, while adapters provide file reading, data storage, and API interfaces.

Two databases are used:
- **MongoDB** for raw, unprocessed ingestion — acts as a persistent ingestion log.
- **PostgreSQL** for normalized, query-optimized data — used by APIs for reporting.

=== Ingestion Workflow

The ingestion process is split into two phases for robustness and replayability:

1. **Raw Ingestion Phase (MongoDB)**:
   - On schedule, read the CSV file line by line.
   - Skip rows where `id <= last_max_id` (tracked in a state collection or table).
   - Insert new records into MongoDB (collection: `raw_crime_data`).
   - Update `last_max_id` in state tracking.
   - On successful ingestion, move the file to `/archive/` folder with timestamped name.

2. **Normalization & Insertion Phase (PostgreSQL)**:
   - Query MongoDB for records where `postgres_synced = false`.
   - Normalize fields into lookup tables (e.g., `primary_types`, `descriptions`, `fbi_codes`).
   - Insert normalized data into the `crime_data` table.
   - Mark those MongoDB documents as `postgres_synced = true`.

=== PlantUML: Ingestion Flow

[plantuml, ingestion-flow, png]
----
@startuml
start
:Scheduler Triggers;
:Open CSV File;
repeat
  :Read Line;
  if (ID > last_max_id?) then (yes)
    :Insert into MongoDB;
  else (no)
  endif
repeat while (more lines?)
:Update last_max_id;
if (Success) then (yes)
  :Move file to /archive/;
endif
:Query unsynced records from MongoDB;
:Normalize + Insert into PostgreSQL;
:Mark records as postgres_synced;
stop
@enduml
----

=== Clean Architecture Mapping

- **Application Layer**
  - `IngestRawCrimeDataUseCase`
  - `SyncNormalizedDataUseCase`
- **Adapters**
  - `CsvCrimeFileReaderAdapter`
  - `MongoCrimeRepositoryAdapter`
  - `PostgresCrimeRepositoryAdapter`
- **Ports**
  - `FileReaderPort`
  - `RawCrimePersistencePort`
  - `CleanCrimePersistencePort`

