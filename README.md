# tmp-secure-patch

This project enforces the use of `tmp@0.2.4` across all nested dependencies using npm's `overrides` field to eliminate known vulnerabilities in `tmp@0.0.33`.

## 🔐 Why

- Vulnerability: [GHSA-52f5-9888-hmc6](https://github.com/advisories/GHSA-52f5-9888-hmc6)
- Description: Allows arbitrary file/directory overwrite via symbolic link manipulation in insecure temp directory handling
- Affected: `tmp@<=0.2.3`
- Fixed: `tmp@0.2.4`

## 📦 How

This project uses the `overrides` field in `package.json`:

```json
"overrides": {
  "tmp": "0.2.4"
}
# Final push to trigger CI
