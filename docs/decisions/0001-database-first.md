# ADR-0001: PostgreSQL from the first version

- Status: Accepted
- Date: 2026-10-23

## Problem
CehSplitter needs to store members, expenses and who shares each expense, and the first
version must provide complete core functionality. I had to decide whether the first version
should store data in memory and add a database later, or use PostgreSQL from the start.

## Options considered
- **In-memory storage first.** No database setup, and simple to test at the beginning.
- **PostgreSQL from day one.** Persistence is real from the first version, and the stack
  (EF Core, migrations, Docker Compose) is set up once.
- **In-memory for the domain logic only, then persistence.** Build and test the splitting and
  balance logic as plain code first, then add the database.

## Decision and why
Use PostgreSQL with EF Core and Docker Compose from the first version.
- Persistence is part of "complete core functionality". A version that loses its data on
  restart is not a finished feature.
- Constraints, transactions and `numeric` rounding behave differently in a real database
  than in memory. Meeting them early avoids finding them late.
- The first version needs only about three tables, so the schema is small and can change
  through migrations.

## Rejected alternatives
- **In-memory first.** My argument for it was easier testing at the start, no database
  setup, and tables I had not designed yet. I rejected it because:
  - The logic that most needs tests (splitting, balances) can be tested as plain functions
    without any storage, so in-memory storage is not needed for that.
  - In-memory storage would be throwaway code that I would rewrite against EF Core, and it
    would hide database behaviour (constraints, concurrency) until the second pass.
  - Schema uncertainty is what migrations are for. Changing the model later is an expected,
    documented part of the project history.
- **Domain logic first, then persistence.** A reasonable approach that I considered. I did
  not choose it because the database setup is small enough to do once at the start, and
  unit tests can already cover the domain logic separately.

## Trade-offs and limitations
- More setup before the first working screen (Docker, a connection string, a first migration).
- Early schema mistakes cost a migration instead of an edit.
- Tests that touch the database are slower and need the container running.
- Assumption, not measured: the setup takes roughly 30 minutes. I will replace this with the
  real time once it is done.

## What would make me revisit this
- If the database setup keeps blocking progress for more than a day or two, I would reconsider
  starting with a simpler store to unblock the domain work.
- If the data model turns out to be very different from the three-table plan, I would pause
  and redesign before adding more features.
