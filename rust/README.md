# JoobQ SDK for Rust

**Background jobs and workflows for Rust, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![crates.io](https://img.shields.io/crates/v/joobq)](https://crates.io/crates/joobq) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
cargo add joobq
```

A worker, as a complete program:

```rust
use joobq::{json, ExecutionContext, JobDef, JoobQClient, Value, Worker};
use std::env;

fn main() {
    let welcome_email = JobDef::new("welcome.email", "default", |payload: Value, _ctx: &ExecutionContext| {
        println!("Welcome email sent to {}", payload["to"].as_str().unwrap_or_default());
        Ok(json!({ "sent_to": payload["to"] }))
    });

    let gateway_url = env::var("JOOBQ_GATEWAY_URL").expect("JOOBQ_GATEWAY_URL is not set");
    let token = env::var("JOOBQ_SDK_TOKEN").expect("JOOBQ_SDK_TOKEN is not set");
    let client = JoobQClient::new(gateway_url, token);
    let worker = Worker::new(client, "welcome-worker-1", vec!["default".to_string()]).register_job(welcome_email);
    if let Err(error) = worker.run() {
        eprintln!("{error}");
    }
}
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```rust
let jid = client.enqueue(&welcome_email, json!({ "to": "ada@example.com" }), joobq::EnqueueOptions::default())?;
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `enqueue_many`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`create_batch`,
`get_batch`) and a shared sliding-window rate limit (`rate_limit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Rust 2021 edition; ureq (rustls), serde, serde_json and signal-hook.
- Install: `cargo add joobq`.
- Use: the crate `joobq`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).