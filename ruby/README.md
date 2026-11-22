# JoobQ SDK for Ruby

**Background jobs and workflows for Ruby, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![RubyGems](https://img.shields.io/gem/v/joobq)](https://rubygems.org/gems/joobq) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
gem install joobq
```

A worker, as a complete program:

```ruby
require "joobq"

class WelcomeEmail
  include JoobQ::Job
  job_name "welcome.email"
  queue "default"

  def self.perform(payload, ctx)
    puts "Welcome email sent to #{payload["to"]}"
    { "sent_to" => payload["to"] }
  end
end

client = JoobQ::Client.new(gateway_url: ENV.fetch("JOOBQ_GATEWAY_URL"), token: ENV.fetch("JOOBQ_SDK_TOKEN"))
worker = JoobQ::Worker.new(client: client, worker_id: "welcome-worker-1", task_queues: ["default"])
worker.register_job(WelcomeEmail)
worker.start
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```ruby
WelcomeEmail.enqueue(client, { "to" => "ada@example.com" })
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `enqueue_many`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`create_batch`,
`get_batch`) and a shared sliding-window rate limit (`rate_limit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Ruby 3.0 or newer; standard library only.
- Install: `gem install joobq`, or `gem "joobq"` in a Gemfile.
- Use: `require "joobq"` (the module `JoobQ`: `JoobQ::Client`, `JoobQ::Worker`).
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).