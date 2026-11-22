# JoobQ SDK for TypeScript / Node.js

**Background jobs and workflows for TypeScript and Node.js, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![npm](https://img.shields.io/npm/v/joobq)](https://www.npmjs.com/package/joobq) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
npm install joobq
```

A worker, as a complete program:

```ts
import { JoobQClient, JoobQWorker, defineJob } from 'joobq'

const welcomeEmail = defineJob({
  name: 'welcome.email',
  queue: 'default',
  perform: async (payload) => {
    console.log(`Welcome email sent to ${payload.to}`)
    return { sent_to: payload.to }
  },
})

const client = new JoobQClient(process.env.JOOBQ_GATEWAY_URL!, process.env.JOOBQ_SDK_TOKEN!)
const worker = new JoobQWorker(client, { workerId: 'welcome-worker-1', taskQueues: ['default'] })
worker.registerJob(welcomeEmail)
await worker.start()
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```ts
await client.enqueue(welcomeEmail, { to: 'ada@example.com' })
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `enqueueMany`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`createBatch`,
`getBatch`) and a shared sliding-window rate limit (`rateLimit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Node.js 22 or newer; no runtime dependencies (Node built-ins only).
- Install: `npm install joobq`.
- Use: `import { JoobQClient, JoobQWorker } from 'joobq'` (an ES module with its type declarations).
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).