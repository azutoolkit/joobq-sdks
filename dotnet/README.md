# JoobQ SDK for .NET

**Background jobs and workflows for .NET, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
dotnet add package JoobQ
```

A worker, as a complete program:

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using JoobQ;

var welcomeEmail = JobDefinition.Define("welcome.email", "default", 3, (payload, ctx) =>
{
    var to = payload?["to"]?.GetValue<string>();
    Console.WriteLine($"Welcome email sent to {to}");
    return Task.FromResult<object?>(new Dictionary<string, object?> { ["sent_to"] = to });
});

var client = new JoobQClient(
    Environment.GetEnvironmentVariable("JOOBQ_GATEWAY_URL")!,
    Environment.GetEnvironmentVariable("JOOBQ_SDK_TOKEN")!);
var worker = new JoobQWorker(client, "welcome-worker-1", new[] { "default" });
worker.Register(welcomeEmail);
await worker.RunAsync();
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```csharp
var jid = await client.EnqueueAsync(welcomeEmail, new Dictionary<string, object?> { ["to"] = "ada@example.com" });
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `EnqueueManyAsync`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`CreateBatchAsync`,
`GetBatchAsync`) and a shared sliding-window rate limit (`RateLimitAsync`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: .NET 8 or newer; no dependencies outside the base class library.
- Install: `dotnet add package JoobQ`.
- Use: the namespace `JoobQ`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).