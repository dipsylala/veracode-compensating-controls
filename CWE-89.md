# CWE-89: Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')

## Overview

Occurs when user-supplied input is incorporated into an SQL query without proper parameterization, allowing an attacker to manipulate query structure, read unauthorized data, or modify the database.

> **Note on scope:** Parameterized queries, ORMs, and stored procedures with typed parameters are **fixes** — if correctly applied, SAST would not report the flaw and risk is reduced to zero. The controls below are for cases where string concatenation into SQL is unavoidable or where the scanner flags a code path it cannot fully trace, but the risk has been reduced to an acceptable level through other means.

---

## Compensating Controls

### 1. Allow-list for Non-Parameterisable Identifiers (Table / Column Names)

SQL does not support binding table names, column names, schema names, or `ORDER BY` targets as parameters — they must be interpolated as strings. Where user input influences these identifiers, concatenation into the query is unavoidable. The risk is mitigated by validating the value against a hard-coded allow-list of permitted identifiers before it is used in the query. Any value not on the list is rejected; user input never reaches the query as a freeform string.

**How to confirm:** Locate the validation step immediately before the flagged query construction. Confirm it compares the input value against a hard-coded, application-controlled set of permitted identifiers (e.g. an array of valid column names, a `switch`/`enum` mapping). The allow-list must be defined in code, not sourced from the database or user input. Any value not matched results in an error response and the query is never executed.

---

### 2. SQL Loaded from Configuration — Not Constructed from User Input

The SQL statement is defined in full in an external configuration file (e.g. XML mapper, `.properties` file, ORM mapping, query config) that is loaded at application startup. The only user-supplied values are bound as parameters to this pre-defined template. The query structure itself cannot be influenced by a user; the scanner flags the string-based query execution but cannot trace that the SQL originated from a trusted configuration source.

**How to confirm:** Identify where the flagged SQL string originates. Confirm it is read from a configuration file or resource (e.g. MyBatis mapper XML, `.sql` file, Spring `@Query`, properties file) that is deployed as part of the application and is not writable at runtime. Confirm that user input is bound only as typed parameters to the pre-defined template and is never concatenated into the SQL string loaded from config.

---

### 3. Value Is Not User-Controlled

The value being concatenated into the SQL query originates from a trusted internal source — a hard-coded constant, a server-generated ID, or a value derived entirely from application state — and cannot be influenced by an external attacker. The scanner identifies the concatenation pattern as a potential sink but cannot trace that the tainted source is not actually user-controlled.

**How to confirm:** Trace the full data flow from the flagged variable back to its origin. Confirm it is never derived from HTTP request parameters, headers, body, cookies, URL segments, or any user-supplied input. Document the source explicitly (e.g. "assigned as a hard-coded string constant", "set to `Thread.CurrentPrincipal.Identity.Name` after authentication", "generated as a server-side UUID").

---

### 4. Least-Privilege Database Account (Defence in Depth)

The application's database account holds only the minimum permissions required for normal operation. This does not prevent injection but substantially limits the impact if exploitation were to succeed — an attacker cannot read unrelated tables, drop objects, or execute operating system commands.

**How to confirm:** The database user (or role) has been audited and holds only the permissions required for this application (e.g. `SELECT`/`INSERT` on specific tables). It has no `DROP`, `CREATE`, or `EXECUTE` rights on sensitive objects and cannot execute `xp_cmdshell` or equivalent.

---

## Mitigation Text Templates

These are typically 'Mitigate by Design' in Veracode.

### Allow-list for non-parameterisable identifier

```text
The flagged code path constructs a SQL query using a string value for [table name / column name / ORDER BY target], which cannot be supplied as a bind parameter in SQL. Before the query is constructed, the value is validated against a hard-coded allow-list of permitted identifiers: [list the allowed values, e.g. "column names: 'name', 'created_date', 'status'"]. Any value not present in this list is rejected with [describe response, e.g. "a 400 Bad Request"] and the query is never executed. User input cannot introduce arbitrary SQL identifiers through this code path. The residual risk of SQL injection exploitation is acceptably low.
```

### SQL from configuration file

```text
The SQL statement executed on this code path is not constructed dynamically from user input — it is a pre-defined template loaded from [config file/resource, e.g. "the MyBatis mapper XML at resources/mappers/OrderMapper.xml" / "the application.properties query definition"]. The query structure is fixed at deployment time and cannot be modified by an attacker. User-supplied values are bound only as typed parameters to this template and are never concatenated into the SQL string. The scanner identifies the query execution call but cannot trace that the SQL itself originates from a trusted configuration source. The residual risk of SQL injection exploitation through this code path is acceptably low.
```

### Value not user-controlled

```text
The value concatenated into the SQL query is not derived from user input and originates from [describe source, e.g. "a hard-coded application constant" / "a server-generated UUID assigned during session initialisation"] and is under [describe security controls for the trusted source]. This value cannot directly be influenced by an external attacker. The scanner identifies the string concatenation pattern but cannot statically verify that the controls around the source.
```

### Allow-list + least-privilege (layered)

```text
The risk from this is lowered through two layered controls: (1) Allow-list validation — the [table/column] name used in the query is validated against a hard-coded set of permitted values [list them] before the query is constructed; any value not on this list is rejected; (2) Least-privilege database account — the application database user is restricted to [list permissions, e.g. "SELECT on the Reporting schema only"] with no DDL rights or access to other schemas, limiting the impact of any exploitation. The residual risk of successful SQL injection through this code path is acceptably low.
```
