# next-serverless

Next.js example: server-side transaction signing with a secret key, batch operations, and a Vercel/Lambda-style serverless handler.

```
API route (server) — load account → build tx → sign with secret key → submit
Client page (browser) — trigger action → poll result → show status
```

## Patterns covered

- Server-side signing with `signTransactionOffline` (no browser wallet needed)
- Batch account fetches with `client.account.getAccountsBatch`
- Building a payment on the server and submitting from an API route
- Vercel/Lambda handler pattern: one-shot client, sign, submit, return result
- Separating server-only code (secret keys) from browser code

## Files

| File | Purpose |
|---|---|
| `src/sorokitServer.ts` | Server-side helpers: client factory, sign-and-submit, batch fetch |
| `src/pages/api/payment.ts` | Next.js API route: receives POST, builds, signs, and submits |
| `src/pages/api/balances.ts` | API route: batch balance fetch for multiple public keys |
| `src/pages/index.tsx` | Client page: triggers the payment and displays the result |
| `src/lambda.ts` | Drop-in AWS Lambda / Vercel Edge handler (same logic, different runtime) |

## Setup

```bash
npm install sorokit-core @creit.tech/stellar-wallets-kit next react react-dom
```

Set your secret key as an environment variable — never hard-code it:

```bash
# .env.local
SIGNING_SECRET_KEY=S...
STELLAR_NETWORK=testnet
```

## Security note

`signTransactionOffline` accepts a raw secret key. Keep it in an environment
variable and never expose it to the browser. The API routes in this example are
server-only (`pages/api/`) and Next.js never bundles them into the client.

## Client Resource Ownership & Reuse

`createSorokitClient` produces a lightweight facade with no background sockets or keep-alive timers. In serverless and edge environments:
- **Client Reuse**: You can instantiate a client once globally or create it per request with zero overhead.
- **Connection Pools**: If using a custom `ConnectionPool`, the application owns its lifecycle and should call `pool.close()` when gracefully terminating the worker or container.
- **Failover Monitors**: If `EndpointPool.startHealthMonitoring()` is used, remember to invoke the returned unregister callback or `pool.stopHealthMonitoring()` so timers do not prevent clean process exit.

## Running type-checks

```bash
npm run typecheck:examples
```
