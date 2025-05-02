== Milestones

[cols="1,3", options="header"]
|===
| Milestone | Description

| M1 - Project Bootstrapping
| - Initialize Spring Boot project with WebFlux, MongoDB, PostgreSQL, Scheduler.
  - Setup basic directory structure based on Clean Architecture.
  - Configure CI/CD pipeline (if applicable).

| M2 - MongoDB Raw Ingestion Pipeline
| - Implement `CsvCrimeFileReaderAdapter` with OpenCSV.
  - Implement `IngestRawCrimeDataUseCase`.
  - Save new records to MongoDB and track `last_max_id`.
  - Implement file move logic to `/archive`.

| M3 - PostgreSQL Normalization
| - Implement lookup table initialization (e.g., primary types, FBI codes).
  - Implement `SyncNormalizedDataUseCase` for transforming and inserting clean data.
  - Maintain `postgres_synced` flag in MongoDB.

| M4 - Reactive REST & GraphQL APIs
| - Implement `CrimeRestController` and `CrimeGraphQLController`.
  - Add filtering (by type, date, arrest, geo).
  - Validate reactive performance.

| M5 - Search Optimization & Indexes
| - Apply full-text and geospatial indexes.
  - Test query performance and tuning.
  - Add advanced filters like full-text search, bounding box, etc.

| M6 - Testing & Hardening
| - Add unit + integration tests (with Testcontainers for DBs).
  - Validate large file handling (~1.8GB).
  - Handle ingestion failures and retries.

| M7 - Production Deployment
| - Set up database instances (MongoDB + PostgreSQL with PostGIS).
  - Deploy service and schedule file polling.
  - Monitor ingestion, API availability, and log errors.

|===
