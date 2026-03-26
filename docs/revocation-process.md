# Version Revocation Process

When a released version is found to be actively dangerous (supply chain compromise,
critical vulnerability, data loss bug), the revocation mechanism can force endpoints
off the bad version immediately.

## When to Use

- **Supply chain compromise** — binary may be malicious (use `shutdown`)
- **Critical security vulnerability** — active exploitation (use `downgrade` or `safe_mode`)
- **Data loss or corruption bug** — the binary causes damage while running (use `safe_mode`)
- **DDoS / resource exhaustion** — the binary causes harm to infrastructure (use `safe_mode`)

## Revocation Actions

| Action | Behavior | Severity |
|--------|----------|----------|
| `safe_mode` | Stops enforcement/monitoring, enters passive update-only mode | Medium |
| `shutdown` | Exits immediately, refuses to restart until manually updated | Critical |
| `downgrade` | Installs a specified safe version (bypasses rollback protection) | High |

## Two Delivery Mechanisms

### 1. Revocation List (standalone + defense-in-depth)

A signed `revocations.json` hosted in this repo. Endpoints check it on startup and
periodically.

**File**: `revocations.json` (repo root, served via GitHub Pages)

```json
{
  "schema_version": 1,
  "revoked_versions": [
    {
      "version": "1.3.0",
      "action": "safe_mode",
      "reason": "CVE-2026-XXXX: policy engine bypass allows unrestricted network access",
      "revoked_at": "2026-03-26T15:00:00Z",
      "safe_version": "1.2.5"
    }
  ],
  "signature": "<Base64 Ed25519 signature>"
}
```

### 2. Server-Pushed Recall (enterprise)

The SixWays Server includes a `recall` field in the heartbeat response. This reaches
enrolled endpoints immediately (within one heartbeat cycle).

```json
{
  "recall": {
    "affected_versions": ["1.3.0", "1.3.1"],
    "action": "downgrade",
    "reason": "CVE-2026-XXXX: policy engine bypass",
    "safe_version": "1.2.5",
    "manifest": { "...signed manifest for 1.2.5..." },
    "recall_id": "recall-2026-03-26-001",
    "signature": "<Ed25519 signature over canonical recall JSON>"
  }
}
```

## Issuing a Revocation

### Step 1: Create the revocation entry

Edit `revocations.json` in this repo. Add an entry to `revoked_versions`:

```json
{
  "version": "1.3.0",
  "action": "safe_mode",
  "reason": "Brief description of the issue",
  "revoked_at": "2026-03-26T15:00:00Z",
  "safe_version": "1.2.5"
}
```

### Step 2: Sign the revocation list

```bash
# From the endpoint repo
cargo xtask sign-revocations \
  --revocations-file ../releases/revocations.json \
  --key-file /path/to/signing_key.bin
```

This uses the **same artifact signing key** as release manifests. A compromised server
or GitHub account cannot forge a revocation without this key.

### Step 3: Commit and push

```bash
cd releases/
git add revocations.json
git commit -m "Revoke v1.3.0: CVE-2026-XXXX policy engine bypass"
git push
```

Endpoints will pick up the revocation on their next check (startup or periodic).

### Step 4: Server-side recall (enterprise)

For enrolled endpoints, also push the recall via the SixWays Server dashboard:

1. Navigate to **Admin > Releases > Recall**
2. Select affected versions
3. Choose action (safe_mode / shutdown / downgrade)
4. Enter reason
5. For downgrade: select safe version (must be a signed release)
6. Confirm

## Security Properties

- Both delivery mechanisms require the **artifact signing key** for valid signatures
- A compromised server alone cannot forge a recall
- A compromised GitHub account alone cannot forge a revocation (signature check fails)
- Recall data is cached to disk — survives daemon restarts
- The `downgrade` action is the only mechanism that bypasses rollback protection
- The `shutdown` action causes the daemon to exit on every startup attempt until a
  non-recalled version is installed manually
