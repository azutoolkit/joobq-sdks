# JoobQ SDK for Python

**Background jobs and workflows for Python, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![PyPI](https://img.shields.io/pypi/v/joobq)](https://pypi.org/project/joobq/) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
pip install joobq
```

A worker, as a complete program:

```python
import os

from joobq import JoobQClient, JoobQWorker, job


@job("welcome.email", queue="default")
def welcome_email(payload, ctx):
    print(f"Welcome email sent to {payload['to']}")
    return {"sent_to": payload["to"]}


client = JoobQClient(os.environ["JOOBQ_GATEWAY_URL"], os.environ["JOOBQ_SDK_TOKEN"])
worker = JoobQWorker(client, "welcome-worker-1", ["default"])
worker.register_job(welcome_email)
worker.start()
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```python
welcome_email.enqueue(client, {"to": "ada@example.com"})
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `enqueue_many`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`create_batch`,
`get_batch`) and a shared sliding-window rate limit (`rate_limit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Python 3.9 or newer; standard library only.
- Install: `pip install joobq`.
- Use: `from joobq import JoobQClient, JoobQWorker, Job`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).