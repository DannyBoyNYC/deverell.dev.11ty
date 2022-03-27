---
title: Postgres
date: "2022-03-27T22:12:03.284Z"
description: "Postgres notes."
navTitle: Postgres
tags: posts
---

# Postgres

{{description}}

Create: psql -h localhost -U devereld -d umami -f sql/schema.postgresql.sql
String: const conString = "postgres://admin:unami@localhost:5432/unami";
postgresql://api_user:dd2345@localhost:5432/books_api

### References

- https://www.sqlservercentral.com/articles/getting-started-with-postgresql-on-macos
- https://www.postgresql.org/docs/14/ddl-generated-columns.html

```
postgres=# \du
```
