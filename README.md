# Students API

A REST API for managing student records, written in Go with SQLite for storage. The project follows a layered layout (handlers, storage, config) and keeps the storage layer behind an interface so the database can be swapped without touching the HTTP code.

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Reference](#api-reference)
- [Error Handling](#error-handling)
- [Dependencies](#dependencies)
- [Roadmap](#roadmap)
- [Author](#author)

## Overview

The service exposes CRUD endpoints for a `student` resource. Requests are validated before they reach the database, configuration is loaded from a YAML file, and the server shuts down gracefully on `SIGINT`/`SIGTERM`, letting in-flight requests finish.

## Tech Stack

| Area          | Choice                         |
| ------------- | ------------------------------ |
| Language      | Go 1.21+                       |
| HTTP          | `net/http` (standard library)  |
| Database      | SQLite                         |
| Config        | YAML via `cleanenv`            |
| Validation    | `go-playground/validator`      |
| Logging       | `log/slog` (standard library)  |

## Project Structure

```
students-api/
├── cmd/
│   └── students-api/
│       └── main.go            # Entry point: wiring, server start, graceful shutdown
├── config/
│   └── local.yaml             # Local configuration
├── internal/
│   ├── config/                # Config struct and loader
│   ├── http/
│   │   └── handlers/
│   │       └── student/       # Student HTTP handlers
│   ├── storage/
│   │   ├── storage.go         # Storage interface
│   │   └── sqlite/            # SQLite implementation
│   ├── types/                 # Domain models
│   └── utils/
│       └── response/          # JSON response and error helpers
├── storage/                   # SQLite database file (created at runtime)
├── go.mod
└── go.sum
```

> Adjust the tree above if your folder names differ.

## Getting Started

### Prerequisites

- Go 1.21 or newer
- A C compiler (GCC or Clang). `mattn/go-sqlite3` uses CGO, so `CGO_ENABLED=1` is required.
  - Windows: install [MSYS2/MinGW-w64](https://www.msys2.org/) or TDM-GCC
  - macOS: `xcode-select --install`
  - Linux: `sudo apt install build-essential`

### Installation

```bash
git clone https://github.com/ujjwalsh-2209/students-api.git
cd students-api
go mod download
```

### Run

```bash
go run cmd/students-api/main.go -config config/local.yaml
```

The server listens on the address defined in the config (default `localhost:8082`).

### Build

```bash
go build -o bin/students-api ./cmd/students-api
./bin/students-api -config config/local.yaml
```

## Configuration

Configuration is read from a YAML file passed with the `-config` flag. The `CONFIG_PATH` environment variable can be used as an alternative.

```yaml
env: "dev"
storage_path: "storage/storage.db"
http_server:
  address: "localhost:8082"
```

| Key                   | Description                              |
| --------------------- | ---------------------------------------- |
| `env`                 | Runtime environment (`dev`, `prod`)      |
| `storage_path`        | Path to the SQLite database file         |
| `http_server.address` | Host and port the server binds to        |

## API Reference

Base URL: `http://localhost:8082/api`

| Method | Endpoint          | Description            |
| ------ | ----------------- | ---------------------- |
| POST   | `/students`       | Create a student       |
| GET    | `/students`       | List all students      |
| GET    | `/students/{id}`  | Retrieve a student     |
| PUT    | `/students/{id}`  | Update a student       |
| DELETE | `/students/{id}`  | Delete a student       |

### Create a student

```http
POST /api/students
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 20
}
```

`201 Created`

```json
{ "id": 1 }
```

### Get a student

```http
GET /api/students/1
```

`200 OK`

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "age": 20
}
```

### List students

```http
GET /api/students
```

`200 OK`

```json
[
  { "id": 1, "name": "John Doe", "email": "john@example.com", "age": 20 }
]
```

### Update a student

```http
PUT /api/students/1
Content-Type: application/json

{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "age": 21
}
```

### Delete a student

```http
DELETE /api/students/1
```

### Quick test with curl

```bash
curl -X POST http://localhost:8082/api/students \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com","age":20}'

curl http://localhost:8082/api/students
```

## Error Handling

Errors are returned as JSON with a consistent shape:

```json
{
  "status": "Error",
  "error": "field validation for 'Name' failed on the 'required' tag"
}
```

| Status | Meaning                                  |
| ------ | ---------------------------------------- |
| 400    | Malformed body or failed validation      |
| 404    | Student not found                        |
| 500    | Unexpected server or database error      |

## Dependencies

Only three third-party modules are used. Everything else comes from the Go standard library.

| Package | Purpose | Docs / Source |
| ------- | ------- | ------------- |
| `github.com/mattn/go-sqlite3` | SQLite driver for `database/sql` (CGO) | [pkg.go.dev](https://pkg.go.dev/github.com/mattn/go-sqlite3) · [GitHub](https://github.com/mattn/go-sqlite3) |
| `github.com/ilyakaznacheev/cleanenv` | Loads config from YAML and environment variables | [pkg.go.dev](https://pkg.go.dev/github.com/ilyakaznacheev/cleanenv) · [GitHub](https://github.com/ilyakaznacheev/cleanenv) |
| `github.com/go-playground/validator/v10` | Struct-level request validation | [pkg.go.dev](https://pkg.go.dev/github.com/go-playground/validator/v10) · [GitHub](https://github.com/go-playground/validator) |

Install them individually if needed:

```bash
go get github.com/mattn/go-sqlite3
go get github.com/ilyakaznacheev/cleanenv
go get github.com/go-playground/validator/v10
```

Or fetch everything declared in `go.mod`:

```bash
go mod tidy
```

## Roadmap

- [ ] Pagination for the list endpoint
- [ ] Filtering by name or email
- [ ] Unit and integration tests
- [ ] Dockerfile
- [ ] OpenAPI specification

## Author

Ujjwal — [github.com/ujjwalsh-2209](https://github.com/ujjwalsh-2209)
