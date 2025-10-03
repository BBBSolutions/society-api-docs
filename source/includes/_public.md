# Public API

**Endpoint URL**

`API_ENDPOINT_PRODUCTION = https://domain.com`

`API_ENDPOINT_STAGING = https://testnet.domain.com`

## Request Structure

All public API requests must be sent to:

**`API_ENDPOINT/api`**

For example:

- Production: `https://domain.com/api`
- Staging: `https://testnet.domain.com/api`

### Request Format

| Component    | Description                    |
| ------------ | ------------------------------ |
| URL          | `API_ENDPOINT/api`             |
| Method       | `POST`                         |
| Content-Type | `application/json`             |
| Body         | JSON-RPC 2.0 formatted request |

---

# Data API
