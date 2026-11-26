# Sorokit Core API Reference

Sorokit Core is a framework-independent TypeScript SDK for Stellar wallets,
accounts, transactions, and Soroban contracts.

Start with {@link createSorokitClient} to create a client. Browse its
{@link SorokitClient} interface for wallet, account, transaction, Soroban, and
network operations. Public methods return {@link SorokitResult} values.

```ts
import { createSorokitClient } from "sorokit-core";

const result = createSorokitClient({ network: "testnet" });
if (result.status === "ok") {
  const client = result.data;
  console.log(client.network.getConfig());
}
```

## Resource Ownership and Lifecycle

Applications need to know whether shared clients own connections, timers, or subscriptions so they can reuse and close them correctly:

1. **SorokitClient (`createSorokitClient`)**:
   - **Ownership**: The client instance is stateless and lightweight; it does *not* own persistent TCP sockets, worker threads, or background timers.
   - **Reuse**: Safe to share as a singleton throughout an application, or instantiate per-request in serverless/edge environments without resource leaks.
   - **Teardown**: Because no sockets or timers are held, no explicit `close()` or `dispose()` is needed on the client itself.

2. **HTTP Connection Pooling (`ConnectionPool`, `setConnectionPool`)**:
   - **Ownership**: The application owns instances of `ConnectionPool`.
   - **Reuse**: A single `ConnectionPool` should be shared across server instances via `setConnectionPool(pool)` to throttle concurrency and reuse socket connections.
   - **Teardown**: Call `pool.close()` upon application or test shutdown to clear keep-alive idle timers and abort pending requests.

3. **Endpoint Failover & Health Monitoring (`EndpointPool`, `configureEndpointFailover`)**:
   - **Ownership**: The application or caller owns the failover configuration.
   - **Lifecycle**: If `pool.startHealthMonitoring()` is invoked, it starts a periodic timer. Call the returned unregister function or `pool.stopHealthMonitoring()` to terminate the timer. For testing or global resets, invoke `resetEndpointPools()`.

4. **Subscriptions & Feeds (`subscribePrices`, `createWebSocketTransactionFeed`, `streamTransactionStatus`)**:
   - **Ownership**: The caller owns the active subscription or WebSocket feed.
   - **Lifecycle**: Methods return handles with `unsubscribe()`, `close()`, or `stop()`. Callers must invoke these during teardown (e.g. React `useEffect` cleanup or Vue `onUnmounted`) to avoid memory or socket leaks.

Use the search box or navigation to find functions, interfaces, and examples.

- [Getting started and SDK usage](https://github.com/TochukwuJustice/core#readme)
- [Task-oriented workflows](https://github.com/TochukwuJustice/core/blob/main/docs/workflows.md)
- [Architecture guide](https://github.com/TochukwuJustice/core/blob/main/docs/architecture.md)
- [Multi-endpoint failover & graceful degradation](https://github.com/TochukwuJustice/core/blob/main/docs/recipes/21-multi-endpoint-failover.md)

To regenerate this reference locally, run `npm ci --legacy-peer-deps` and
`npm run docs`, then open `docs/api/index.html`. Generated files are not committed.
