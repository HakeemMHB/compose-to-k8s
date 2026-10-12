# 2. Use FastAPI as the web framework

Date: 2026-10-12

## Status

Accepted

## Context

The application is written in Python and needs:

- server-rendered HTML pages: a submission form and a list of reports;
- strict validation of user input: required fields, length limits, a closed
  list of severities;
- two technical endpoints: one reporting that the process is alive, one
  reporting that both databases respond;
- to run in containers.

## Considered Options

- **Flask**: minimal and flexible, but input validation and API documentation
  must be added through extensions.
- **Django**: complete (ORM, admin, authentication), but most of it would go
  unused in a two-page application, at the cost of more structure to learn.
- **FastAPI**

## Decision Outcome

Chosen option: **FastAPI**, because:

- input validation is built in through Pydantic models, so invalid
  submissions are rejected before they reach the databases;
- HTML pages can be rendered server-side with Jinja2 templates, while the
  health endpoints are plain routes;
- it automatically generates an OpenAPI description of the routes;
- it is widely used to serve machine learning models, which makes it directly
  relevant to MLOps work.

### Consequences

- **Good:** routing, validation and documentation come from a single
  framework, with little code.
- **Good:** the skills transfer directly to MLOps.
- **Bad:** no built-in ORM, admin or authentication. Database access
  (SQLAlchemy or psycopg) and security features such as CSRF protection must
  be added explicitly.
- **Bad:** a smaller ecosystem for server-rendered pages than Django.
