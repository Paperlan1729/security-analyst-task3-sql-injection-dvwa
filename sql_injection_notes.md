# SQL Injection Notes — DVWA Low Security

**Author:** Dhrumit Asari  
**Environment:** Local DVWA (XAMPP / LAMP) — Security Level: Low  
**Date:** October 2026  
**Ethics:** Local lab only. No external systems tested.

---

## Module Used
SQL Injection (the classic user-ID lookup form).

## Payload Log

### Attempt 1 — Boolean-based authentication bypass style
**Input:** `1' OR '1'='1`  
**Observed Output:** Multiple user records returned (all rows matching the always-true condition).  
**Analysis:** The single quote closes the original string, `OR '1'='1` makes the WHERE clause true for every row, and the trailing quote balances the syntax. Classic demonstration of lack of input sanitisation.

### Attempt 2 — Comment-based variant
**Input:** `1' OR '1'='1' --`  
**Observed Output:** Same as above — all (or first set of) users displayed.  
**Analysis:** The `--` (SQL comment) causes the remainder of the original query to be ignored, providing a cleaner way to terminate the injected statement.

### Attempt 3 — UNION-based data extraction
**Input:** `1' UNION SELECT user, password FROM users --`  
**Observed Output:** Original row plus additional rows containing usernames and password hashes from the `users` table.  
**Analysis:** UNION requires the same number of columns as the original SELECT. On DVWA Low the original query returns two columns (first_name, last_name), so selecting `user, password` aligns correctly and appends the credential data to the result set.

### Additional Notes
- Error messages on Low security are verbose and helpful for enumeration.
- No prepared statements or escaping are present at this security level — intentional for training.
- Moving to Medium or High security changes the filtering and requires different techniques (see Task 9).

## Remediation Summary (Developer View)
1. Never concatenate user input into SQL.
2. Use prepared statements / parameterised queries exclusively.
3. Apply least-privilege database accounts.
4. Consider Web Application Firewall (WAF) rules as defence-in-depth, not as the primary control.
5. Validate and sanitise input for expected type/length even when using prepared statements.

## References for Further Learning
- PortSwigger Web Security Academy — SQL Injection
- OWASP SQL Injection Prevention Cheat Sheet
- DVWA official documentation
