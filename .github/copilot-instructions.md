<!-- Copilot instructions for this repository - concise, actionable guidance -->
# Repository intent

This repository demonstrates using the Azure Cosmos DB Linux emulator in GitHub Actions for four language samples: .NET, Java, Go and Python. Each language includes a small `UserOperations` implementation and corresponding integration tests that create a database and container against an emulator/service container.

**Quick goals for an AI coding agent**
- Implement or update language-specific `UserOperations` logic consistently across folders.
- Keep environment-variable-driven test setup intact: tests expect `COSMOSDB_CONNECTION_STRING`, `COSMOSDB_DATABASE_NAME`, `COSMOSDB_CONTAINER_NAME`.
- Preserve the pattern where tests create the DB/container and delete the DB in teardown.

**Key files to inspect**
- `dotnet-app/UserOperations.cs` and `dotnet-app/UserOperationsTest.cs` (NUnit)
- `java-app/src/main/java/com/demo/UserOperations.java` and `java-app/src/test/java/com/demo/UserOperationsTest.java` (JUnit 5)
- `go-app/user/user_operations.go` and `go-app/user/user_operations_test.go` (Go + testify)
- `python-app/user_operations.py` and `python-app/test_user_operations.py` (pytest)
- `README.md` — explains GitHub Actions emulator usage and required secrets/variables.

Architecture & patterns (what to follow)
- Multi-language samples share one integration-testing pattern: read `COSMOSDB_*` env vars, create DB/container if missing, run tests, then delete DB. Keep this lifecycle when modifying tests.
- Partition key is consistently `/id` across samples — avoid changing partition key without updating tests in all languages.
- Tests are integration-style and rely on a running Cosmos emulator or compatible service container. Locally set `COSMOSDB_CONNECTION_STRING` to the emulator connection string shown in `README.md`.

Build / test commands (examples)
- .NET (from repo root):
  - `dotnet test dotnet-app/dotnet-app.sln --configuration Debug`
- Java (Maven):
  - `mvn -f java-app/pom.xml test`
- Go:
  - `cd go-app && go test ./...`
- Python (inside `python-app`):
  - `python -m pip install -r python-app/requirements.txt` then `pytest -q python-app`

Connection string / emulator notes
- All code reads `COSMOSDB_CONNECTION_STRING` and the sample workflows rely on the Linux emulator container described in `README.md`.
- .NET and Java samples adjust the connection string (replace `http://` with `https://`) and configure the client to accept the emulator's certs (see `dotnet-app/UserOperationsTest.cs`). If you change transport/security handling, update the corresponding sample(s).

Editing guidelines for PRs
- When changing `User` schema, update all four language samples and their tests for parity.
- Preserve test lifecycle (create DB/container in setup, delete DB in teardown). Failing to delete will leave test artifacts in the emulator between runs.
- When adding dependencies, update per-language manifests: `dotnet-app.csproj`, `java-app/pom.xml`, `go-app/go.mod`, `python-app/requirements.txt`.

Examples of common edits
- Add a new property to the `User` model: update `User` class/struct in each language and the tests that assert fields. See `User` definitions in the four sample files listed above.
- If exposing helper functions for reuse, place them alongside sample code in the same language folder (no global libs in this repo).

If anything here is unclear or you want more detail (CI workflow specifics, emulator startup commands, or a PR checklist), tell me which section to expand. 
