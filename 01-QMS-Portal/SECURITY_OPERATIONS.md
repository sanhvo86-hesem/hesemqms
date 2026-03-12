# HESEM QMS Security Operations

This file documents quick operational commands for post-deploy checks and emergency user reset.

## 1) Run Post-Deploy Health Check

Run from `01-QMS-Portal/scripts`:

```bash
php postdeploy_healthcheck.php
```

Optional API status check:

```bash
php postdeploy_healthcheck.php --url="https://qms.hesem.com.vn/01-QMS-Portal/api.php"
```

## 2) Emergency Reset All User Passwords + MFA

Dry run first:

```bash
php security_reset_all_users.php
```

Apply (all users):

```bash
php security_reset_all_users.php --apply
```

Apply only active users and exclude selected accounts:

```bash
php security_reset_all_users.php --apply --scope=active --exclude=sanh.vo,mai.tran
```

Write credential CSV to custom path:

```bash
php security_reset_all_users.php --apply --out="/secure/path/reset_credentials.csv"
```

## 3) Notes

- `--apply` creates backup file `users.json.bak_YYYYMMDD_HHMMSS` before writing.
- Reset action forces:
  - New temp password hash.
  - `mfa.enabled=false` (users re-enroll MFA at next login).
  - `must_change_password=true`.
- Remove credential CSV after secure handover.
- If `QMS_DATA_DIR` is set, scripts use it. Otherwise they auto-prefer private data dir outside web root.

