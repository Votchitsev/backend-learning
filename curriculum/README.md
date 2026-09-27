# Backend Engineering Curriculum

## Goal

Develop practical backend engineering fundamentals while preserving
and extending existing TypeScript engineering experience.

Target stack:

Node.js → NestJS → PostgreSQL → Prisma → Docker

The goal is not simply to learn framework APIs.

The student should understand the underlying runtime, HTTP,
database, concurrency, architecture, and deployment concepts well
enough to reason about backend systems independently.

## Student background

The student is an experienced frontend and React Native developer.

Existing knowledge includes:

- JavaScript
- TypeScript
- React
- React Native
- Git
- HTTP from a client-side perspective
- General software development practices

Do not spend curriculum time teaching basic JavaScript or
TypeScript unless a specific knowledge gap is identified.

---

# Learning principles

Theory is primarily studied through official documentation.

Each lesson follows this cycle:

1. Study assigned documentation.
2. Complete practical exercises independently.
3. Submit the solution for code review.
4. Fix identified problems independently.
5. Complete a knowledge assessment.
6. Continue only when the important concepts are understood.

Passing tests does not automatically mean that a topic is mastered.

The curriculum defines WHAT should be learned.

Lesson materials define HOW a particular lesson is studied.

Do not change the curriculum automatically based on generated
lesson content.

---

# Phase 1 — Node.js Fundamentals

Goal:

Understand what happens underneath backend frameworks before
introducing NestJS.

## Lesson 01 — Node.js Runtime and Event Loop

Topics:

- Node.js runtime
- V8
- Call stack
- Event loop
- Event loop phases at a practical level
- Microtasks
- Promise callbacks
- `process.nextTick`
- Timers
- Asynchronous I/O
- libuv
- Thread pool
- Blocking vs non-blocking operations
- CPU-bound work

Expected outcome:

The student can explain how Node.js executes JavaScript and
coordinates asynchronous work.

The student can predict execution order in common asynchronous
examples and explain why CPU-intensive synchronous work can block
unrelated requests.

Practice should include:

- Execution-order prediction.
- Event loop experiments.
- Blocking behavior experiments.
- Comparison of CPU-bound and asynchronous work.

---

## Lesson 02 — Asynchronous I/O

Topics:

- Asynchronous I/O
- Promises
- `async` / `await`
- Sequential vs concurrent asynchronous operations
- `Promise.all`
- Error propagation
- File system operations
- Blocking filesystem APIs
- Non-blocking filesystem APIs

Expected outcome:

The student can reason about asynchronous execution rather than
simply using `async` and `await` syntactically.

The student understands when independent operations can execute
concurrently and can handle asynchronous failures correctly.

Practice should include:

- File operations.
- Sequential execution.
- Concurrent execution.
- Error handling.
- Measuring execution behavior.

---

## Lesson 03 — HTTP with Node.js

Topics:

- HTTP request / response model
- HTTP methods
- Status codes
- Headers
- URL
- Query parameters
- Request body
- JSON
- Basic routing
- Error responses
- `node:http`

Expected outcome:

The student understands how an HTTP server works without relying
on a web framework.

The student can implement a small REST-style API using Node.js
standard APIs.

Practice should include an API with operations similar to:

- `GET /users`
- `GET /users/:id`
- `POST /users`
- `PATCH /users/:id`
- `DELETE /users/:id`

No Express, NestJS, or other HTTP framework should be used.

---

## Lesson 04 — Streams, Buffer and Backpressure

Topics:

- Buffer
- Readable streams
- Writable streams
- Transform streams
- `pipe`
- `pipeline`
- Backpressure
- Memory implications of large data processing

Expected outcome:

The student understands why streaming exists and can process data
without loading an entire large resource into memory.

Practice should include:

- Reading large files.
- Streaming transformations.
- Comparing buffered and streaming approaches.
- Observing memory behavior.

This lesson concludes the Node.js fundamentals phase.

A short Node.js fundamentals challenge should verify knowledge from
Lessons 01–04.

---

# Phase 2 — NestJS Fundamentals

Goal:

Learn backend application architecture using NestJS while
understanding what the framework abstracts.

## Lesson 05 — NestJS Architecture

Topics:

- Application bootstrap
- Modules
- Controllers
- Providers
- Services
- Basic request handling

Expected outcome:

The student understands the basic NestJS application structure and
can implement a small feature using modules, controllers, and
providers.

---

## Lesson 06 — Dependency Injection

Topics:

- Dependency Injection
- IoC
- NestJS container
- Providers
- Injection tokens
- Custom providers
- Provider scopes
- Dependency graph

Expected outcome:

The student understands why dependency injection exists and can
explain how NestJS creates and supplies dependencies.

The student should understand structures such as:

Controller → Service → Repository

without treating dependency injection as framework magic.

---

## Lesson 07 — DTO and Validation

Topics:

- DTO
- Input validation
- ValidationPipe
- Transformation
- Validation errors
- HTTP error responses

Expected outcome:

The student can define and enforce an API contract at the
application boundary.

---

## Lesson 08 — NestJS Request Lifecycle

Topics:

- Middleware
- Guards
- Interceptors
- Pipes
- Controllers
- Exception filters
- Request lifecycle ordering

Expected outcome:

The student can explain how a request travels through a NestJS
application and select the appropriate mechanism for common
cross-cutting concerns.

---

## Lesson 09 — Configuration and Application Structure

Topics:

- Environment variables
- Configuration
- ConfigModule
- Logging
- Feature modules
- Module boundaries
- Application structure

Expected outcome:

The student can organize a growing NestJS application without
putting unrelated responsibilities into the same module or service.

---

## Lesson 10 — NestJS Fundamentals Challenge

No major new concepts.

The student receives a feature specification and independently
decides:

- Modules.
- Controllers.
- Providers.
- DTOs.
- Validation.
- Error handling.

Expected outcome:

Demonstrate practical understanding of Lessons 05–09.

---

# Phase 3 — PostgreSQL Fundamentals

Goal:

Develop strong relational database fundamentals before relying on
an ORM.

ORM usage is intentionally postponed until after this phase.

## Lesson 11 — Relational Data Modeling

Topics:

- Database
- Schema
- Table
- Row
- Column
- PostgreSQL data types
- Primary keys
- Foreign keys
- UNIQUE
- NOT NULL
- CHECK
- Relationships
- One-to-one
- One-to-many
- Many-to-many

Expected outcome:

The student can translate basic business requirements into a
relational data model with appropriate constraints.

---

## Lesson 12 — SQL CRUD

Topics:

- SELECT
- INSERT
- UPDATE
- DELETE
- WHERE
- ORDER BY
- LIMIT
- Basic expressions

Expected outcome:

The student can manipulate and query relational data using SQL
without an ORM.

---

## Lesson 13 — JOIN

Topics:

- INNER JOIN
- LEFT JOIN
- Joining multiple tables
- Relationship traversal
- Missing related data

Expected outcome:

The student can retrieve data across relational structures and
explain the semantic difference between common JOIN types.

---

## Lesson 14 — Aggregation

Topics:

- COUNT
- SUM
- AVG
- MIN / MAX
- GROUP BY
- HAVING

Expected outcome:

The student can answer analytical business questions using SQL.

Example problem:

Calculate expenses grouped by category and month.

---

## Lesson 15 — Indexes and Query Execution

Topics:

- Index purpose
- B-tree indexes
- Selectivity
- Single-column indexes
- Composite indexes
- Index column order
- Query planner
- EXPLAIN
- EXPLAIN ANALYZE
- Cost of indexes

Expected outcome:

The student can reason about whether an index may help a particular
query and verify assumptions using PostgreSQL query plans.

The student should not learn the rule:

"indexes make queries faster"

without understanding their trade-offs.

---

## Lesson 16 — Transactions

Topics:

- ACID
- BEGIN
- COMMIT
- ROLLBACK
- Atomicity
- Transaction boundaries
- Failure handling

Expected outcome:

The student recognizes operations that must be atomic and can
implement them using database transactions.

A money transfer between accounts should be used as one practical
scenario.

---

## Lesson 17 — Concurrency and Locking

Topics:

- Concurrent transactions
- Race conditions
- Lost updates
- Transaction isolation
- Row-level locking
- `SELECT ... FOR UPDATE`
- Deadlocks

Expected outcome:

The student understands that individually correct database
operations can produce incorrect results when executed
concurrently.

The student can reason about common concurrency failures and
appropriate database mechanisms.

---

## Lesson 18 — PostgreSQL Challenge

No major new concepts.

The challenge should combine:

- Data modeling.
- Constraints.
- SQL.
- JOIN.
- Aggregation.
- Indexes.
- Transactions.
- Concurrency reasoning.

Expected outcome:

Demonstrate practical understanding of PostgreSQL fundamentals
without relying on an ORM.

---

# Phase 4 — NestJS + PostgreSQL

Goal:

Combine application architecture and relational database knowledge
into a realistic backend service.

## Lesson 19 — Prisma Fundamentals

Topics:

- ORM purpose
- Prisma schema
- Models
- Relations
- Prisma Client
- CRUD
- Migrations
- Generated SQL awareness

Expected outcome:

The student can use an ORM without treating the database as an
implementation detail.

Previously learned SQL and PostgreSQL concepts remain authoritative.

---

## Lesson 20 — Data Access and Transactions

Topics:

- Data access layer
- Repository responsibility
- Service responsibility
- ORM queries
- Relations
- Transactions
- Business logic boundaries

Expected outcome:

The student can separate application business logic from data
access concerns and knows when transaction boundaries belong in
the application workflow.

---

## Lesson 21 — Authentication and Authorization

Topics:

- Password hashing
- Authentication
- Authorization
- JWT
- Access tokens
- Refresh tokens
- Guards
- Roles / permissions
- Basic authentication security

Expected outcome:

The student can implement and explain a basic authentication flow
and distinguish authentication from authorization.

Example API:

- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `GET /profile`

---

## Lesson 22 — Production-style API Endpoint

Topics:

- Filtering
- Pagination
- Validation
- Data access
- Query design
- Index implications
- Error handling

Example:

`GET /transactions?from=&to=&category=&page=`

Expected outcome:

Combine NestJS, PostgreSQL, Prisma, validation, query design, and
API design in one realistic feature.

---

# Phase 5 — Docker Fundamentals

Goal:

Understand how the backend application and its infrastructure can
be packaged and run reproducibly.

## Lesson 23 — Containers and Images

Topics:

- Container
- Image
- Dockerfile
- Layers
- Build context
- Ports
- Environment variables
- Container lifecycle

Expected outcome:

The student can containerize the NestJS application and explain
the difference between an image and a running container.

---

## Lesson 24 — Docker Compose, Networking and Persistence

Topics:

- Docker Compose
- Services
- Docker networks
- Service discovery
- Volumes
- PostgreSQL persistence
- Environment configuration
- Application startup
- Migrations

Expected outcome:

The complete backend environment can be started using a command
such as:

`docker compose up`

The student understands how the NestJS application communicates
with PostgreSQL and why database data survives container
recreation when configured correctly.

---

# Final Project

After completing the fundamentals, build a backend service without
step-by-step implementation instructions.

Suggested domain:

Personal finance management.

Core model:

User
↓
Accounts
↓
Transactions
↓
Categories

Possible functionality:

- Registration and authentication.
- Accounts.
- Income and expenses.
- Categories.
- Transfers between accounts.
- Transaction filtering.
- Pagination.
- Statistics.
- Data validation.
- Transactional operations.

Required stack:

- TypeScript
- Node.js
- NestJS
- PostgreSQL
- Prisma
- Docker

AI coding agents may be used during the final project unless a
specific task is explicitly designated as independent practice.

The student must still be able to explain and defend important
technical decisions.

---

# Completion criteria

A lesson is not complete simply because its implementation works.

Progress is tracked independently for:

- Documentation studied.
- Practice completed.
- Code review completed.
- Knowledge demonstrated.

A lesson is considered mastered only after the student demonstrates
understanding of its central concepts.

See `PROGRESS.md` for current state.