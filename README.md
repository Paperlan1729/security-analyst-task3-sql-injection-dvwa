# Security Analyst Task 3: SQL Injection on DVWA (Low Security)

**Author:** Dhrumit Asari  
**GitHub:** [Paperlan1729](https://github.com/Paperlan1729)  
**Track:** Security Analyst (Practical Task)

---

## What is SQL Injection?

SQL Injection (SQLi) is a web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database. It typically occurs when user-supplied input is concatenated directly into an SQL statement without proper sanitization or parameterization. An attacker can modify the intended query logic to bypass authentication, extract sensitive data, modify or delete records, or in severe cases execute operating-system commands.

## Why the Classic Payload Works

On DVWA Low security, the backend PHP code constructs a query similar to:

```sql
SELECT * FROM users WHERE user_id = '$_GET["id"]'
```

When the input `' OR '1'='1` is supplied, the query becomes:

```sql
SELECT * FROM users WHERE user_id = '' OR '1'='1'
```

Because `'1'='1'` is always true, the WHERE clause evaluates to true for every row and the application returns all user records (or the first matching set, depending on the exact code path).

## Data Exposed

Typical results on DVWA Low:
- User IDs
- First names / surnames
- In some modules, password hashes or additional columns depending on the exact query

## How a Developer Would Fix This

**Use parameterised queries (prepared statements).**  
Never concatenate user input into SQL strings.

### PHP (mysqli) example
```php
$stmt = $mysqli->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $id);
$stmt->execute();
$result = $stmt->get_result();
```

### PHP (PDO) example
```php
$stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = :id");
$stmt->execute(['id' => $id]);
$rows = $stmt->fetchAll();
```

Prepared statements ensure the database engine treats the input strictly as data, not as executable SQL.

---

## Setup Documentation (Local Lab Only)

1. Install XAMPP (Windows) or LAMP stack (Linux).
2. Download DVWA from the official GitHub repository.
3. Place files in the web root (`htdocs` / `/var/www/html`).
4. Create the MySQL database and user as instructed by DVWA.
5. Copy `config/config.inc.php.dist` to `config.inc.php` and set database credentials.
6. Browse to `http://localhost/DVWA`, log in with default credentials (`admin` / `password`), and set the security level to **Low**.

**Ethics:** This exercise must be performed only on DVWA running locally on your own machine. Never attempt SQL injection against any real website or service.

---

## Injection Attempts Performed

See detailed payload log in [sql_injection_notes.md](sql_injection_notes.md).

| # | Payload | Result |
|---|---------|--------|
| 1 | `' OR '1'='1` | All users returned |
| 2 | `' OR '1'='1' --` | All users returned (comment out remainder of query) |
| 3 | `' UNION SELECT user, password FROM users --` | Usernames and password hashes extracted (UNION-based) |

Screenshots of each attempt should be placed in the `screenshots/` directory.

---

## Files

- `sql_injection_notes.md` — Full payload log and analysis
- `README.md` — This file
- `screenshots/` — Capture of each injection attempt and output

---

*Completed by Dhrumit Asari — Security Analyst Track. All testing performed exclusively on local DVWA instance.*
