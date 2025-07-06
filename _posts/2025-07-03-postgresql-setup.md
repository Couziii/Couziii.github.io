---
layout: post
title:  "Postgresql – Setup/Django"
date:   2025-07-03 10:00:00 +0000
categories: database, rdms, relational database
---


## 1. What is PostgreSQL?

PostgreSQL is an open-source relational database.

### Strengths:

#### ACID Compliance

| ACID Property        | Description                                                                                           |
|----------------------|-------------------------------------------------------------------------------------------------------|
| **Atomicity**         | A transaction includes every step, or nothing at all.                                                 |
| **Consistency**       | A transaction moves the database from one valid state to another, preserving all rules and constraints. <br>_E.g., can't add an order if `product_id` doesn't exist._ |
| **Isolation**         | Concurrent transactions don't interfere with each other. They behave as though run one after another. |
| **Durability**        | Once a transaction is committed, it persists—even in a crash or power failure.                        |

#### Other Strengths
> **Custom / Extensibility** – Supports custom data types, functions, and extensions like **PostGIS** for geospatial data.  
> **Concurrency** – Uses Multi-Version Concurrency Control (**MVCC**) for efficient parallel transactions.  
> **Analytics** – Advanced SQL querying features such as window functions, CTEs, and full-text search.

---

### Weaknesses:

> **Performance tuning complexity** – Requires tuning of parameters like `work_mem`, `shared_buffers`, and proper indexing.  
> **Write-heavy workloads** – May lag under massive, high-frequency write operations (e.g., logging, IoT) compared to NoSQL solutions.  
> **Complex replication/scaling** – Sharding and horizontal scaling can be more complex than some alternatives.

## 2. Setup PostgreSQL

### A. Installing Prerequisites

Install PostgreSQL along with optional and development tools:

- `postgresql-contrib` – *(Optional)* Additional PostgreSQL tools like `pg_stat_statements`.
- `libpq-dev`, `python3-dev` – PostgreSQL development libraries used by tools like **Django** and **psycopg2**.

```bash
$ sudo apt update
$ sudo apt install postgresql postgresql-contrib libpq-dev python3-dev
```

### B. Start Posgres shell as postgres user
```bash
$ sudo -u postgres psql
```

### C. Create user and database
```python
CREATE USER user WITH PASSWORD 'mypassword';
CREATE DATABASE mydb OWNER myuser;

-- Ensures consistent encoding across connections (important for Django)
ALTER ROLE myuser SET client_encoding TO 'utf8';


-- Prevents dirty reads in transactions (safe default)
ALTER ROLE myuser SET default_transaction_isolation TO 'read committed';

-- Ensures time-related fields are in UTC (important for servers/apps)
ALTER ROLE myuser SET timezone TO 'UTC';

-- Gives full access to "myuser" for all actions on the "mydb" database
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
```


## 3. Create Schemas and grant schema-level permissions
- Postgres creates a default public schema
```python
CREATE SCHEMA myschema AUTHORIZATION myuser;
GRANT USAGE ON SCHEMA myschema TO myuser;
GRANT ALL PRIVILEGES ON ALL TABlES IN SCHEMA myschema TO myuser;

-- Tells PostgreSQL to look in "myschema" first when accessing tables/functions
ALTER USER myuser SET search_path = myschma, "$user", public;
```

## 4. Connect with Django
### A. Install Python Adapter
- Installs the PostgreSQL driver for Python/Django (used in DATABASES settings)
```bash
$ pip install psycopg2-binary
```

### B. Configure settings.py
```python
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.postgresql',
    'NAME': 'mydb',
    'USER': 'myuser',
    'PASSWORD': 'mypassword',
    'HOST': 'localhost',
    'PORT': '5432',
  }
}
```

5. Connect with Node,js
### A. Install Python Adapter
- Installs the 'pg' library which allows you to connect and query PostgreSQL from Node.js
```bash
$ npm install pg
```

### B. Configure settings.py
```JavaScript
const { Client } = require('pg');

// Configure the database connection
const client = new Client({
  user: 'myuser',
  host: 'localhost',
  database: 'mydb',
  password: 'mypassword',
  port: 5432,
});

// Connect to the PostgreSQL database
await client.connect();

// Example table creation query
await client.query(`
  CREATE TABLE IF NOT EXISTS myschema.example (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    created TIMESTAMPTZ DEFAULT now()
  );
`);

// Close the connection once done
await client.end();

```

| Category             | Requirements                                        |
| -------------------- | --------------------------------------------------- |
| **DB Engine**        | PostgreSQL installed & running                      |
| **Superuser Access** | Via `postgres` for creating DB, roles, schemas      |
| **App user**         | Role (e.g., `myuser`) with password                 |
| **Database**         | Created with correct owner                          |
| **Schema**           | Default (`public`) or custom (`myschema`)           |
| **Grants**           | `GRANT USAGE` on schema, `ALL PRIVILEGES` on tables |
| **Search path**      | Set to your schema for convenient access            |
| **Drivers**          | `psycopg2-binary` for Django, `pg` for Node.js      |
| **Configuration**    | Proper in-app settings (credentials, host, port)    |
