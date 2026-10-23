
| Role / Group | Row Visibility | Sensitive Column Masking (amount, email, name) |
| :--- | :--- | :--- |
| `meridian_admins` / **Owners** | All rows globally | Unmasked (Full Visibility) |
| `meridian_analysts_us` | US destination rows only | Partial Masking (`amount`: show rounded int, `email`: show domain) |
| `meridian_analysts_global` | All rows globally | Full Redaction (`NULL` or `'[REDACTED]'`) |
