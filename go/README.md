# JoobQ SDK for Go

**Background jobs and workflows for Go, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![pkg.go.dev](https://pkg.go.dev/badge/github.com/azutoolkit/joobq-sdk-go.svg)](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
go get github.com/azutoolkit/joobq-sdk-go
```

A worker, as a complete program:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"

	joobq "github.com/azutoolkit/joobq-sdk-go"
)

func main() {
	welcomeEmail := joobq.DefineJob("welcome.email", joobq.JobOptions{Queue: "default"},
		func(payload map[string]any, ec *joobq.ExecutionContext) (any, error) {
			fmt.Println("Welcome email sent to", payload["to"])
			return map[string]any{"sent_to": payload["to"]}, nil
		})

	client := joobq.NewClient(os.Getenv("JOOBQ_GATEWAY_URL"), os.Getenv("JOOBQ_SDK_TOKEN"))
	worker := joobq.NewWorker(client, "welcome-worker-1", []string{"default"})
	worker.RegisterJob(welcomeEmail)
	if err := worker.Run(context.Background()); err != nil {
		log.Fatal(err)
	}
}
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```go
jid, err := welcomeEmail.Enqueue(context.Background(), client, map[string]any{"to": "ada@example.com"}, joobq.EnqueueOptions{})
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `EnqueueMany`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`CreateBatch`,
`GetBatch`) and a shared sliding-window rate limit (`RateLimit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Go 1.22 or newer; standard library only.
- Install: `go get github.com/azutoolkit/joobq-sdk-go`.
- Use: the module `github.com/azutoolkit/joobq-sdk-go`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).