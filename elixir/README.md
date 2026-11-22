# JoobQ SDK for Elixir

**Background jobs and workflows for Elixir, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

A worker, as a complete program:

```elixir
Mix.install([{:joobq_ex, git: "https://github.com/azutoolkit/joobq-sdks.git", sparse: "elixir", tag: "v0.8.0"}])

defmodule WelcomeEmail do
  use JoobQ.Worker, queue: "default", job_type: "welcome.email"

  @impl JoobQ.Worker
  def perform(%JoobQ.Job{payload: %{"to" => to}}) do
    IO.puts("Welcome email sent to #{to}")
    {:ok, %{"sent_to" => to}}
  end
end

worker = [
  url: System.fetch_env!("JOOBQ_GATEWAY_URL"),
  token: System.fetch_env!("JOOBQ_SDK_TOKEN"),
  worker_id: "welcome-worker-1",
  queues: ["default"],
  workers: [WelcomeEmail]
]

{:ok, _supervisor} = Supervisor.start_link([{JoobQ, worker}], strategy: :one_for_one)
Process.sleep(:infinity)
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```elixir
WelcomeEmail.enqueue(%{"to" => "ada@example.com"})
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A client for the JoobQ SDK gateway of a self-hosted JoobQUI server: enqueue jobs now, later or at a given time, with
deadlines, idempotency keys and fields the gateway seals, one at a time (`JoobQ.enqueue/4`, `enqueue_async/4`) or up
to 500 in one call (`enqueue_many/2`), and run them in the application with `use JoobQ.Worker` and the instance's
`:queues` and `:workers` options (`JoobQ.Config`); register job schedules, use JoobQ Pro batches and rate limits, and
start and talk to workflow runs, and build workflows whose steps the worker runs (`JoobQ.Workflow`, the
instance's `:workflows` option).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Elixir 1.16 or newer; depends on `finch`, `jason` and `telemetry`.
- Install: from the repository until the first Hex release, `{:joobq_ex, git: "https://github.com/azutoolkit/joobq-sdks.git", sparse: "elixir", tag: "v0.8.0"}` in `mix.exs`; the Hex package `{:joobq_ex, "~> 0.8.0"}` follows the first release (the name is not published yet).
- Use: the module `JoobQ`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ).
Crystal applications use the JoobQ shards directly and share the same queues.

## Workers

```elixir
children = [
  {JoobQ,
   url: "https://joobqui.example.com",
   token: System.fetch_env!("JOOBQ_SDK_TOKEN"),
   queues: ["mail"],
   workers: [MyApp.Welcome],
   middleware: [MyApp.Timing]}
]
```

- **Jobs**: a module that does `use JoobQ.Worker, queue: "mail"` and implements `perform/1` (`JoobQ.Worker`); it
  answers `:ok`, `{:ok, output}` or `{:error, reason}`; `JoobQ.NonRetryable` skips the retries.
- **Context**: inside `perform/1`, what the handler enqueues, batches or starts carries the job's context under its
  own (`JoobQ.Context`); a process the handler spawns does not.
- **Middleware** (`JoobQ.Middleware`): `call(job, next, opts)` around every run, the first listed outermost; the
  job it gets has no lease token.
- **Stopping**: when the instance stops (the application shuts down, on SIGTERM too, or `Supervisor.stop/1`) the
  worker stops polling, runs what it holds and sends the outcomes within `:drain_timeout` (30 s by default), then
  deregisters, after a registration under way and within 5 s. At most a stop takes `:drain_timeout` + 1 s, plus a
  registration under way (a sixth of `:lease_ttl_ms` + 5 s) and 5 s for a connection, plus 5 s: 52 s with the
  defaults. A job not settled by then is left to its lease: the gateway queues it again when the lease expires,
  with one retry used. A process of the worker that crashes restarts alone, without a drain.
  That is longer than Kubernetes' default termination grace period (30 s): raise `terminationGracePeriodSeconds`
  to 60, or lower `:drain_timeout` (with 8 000 ms and the default lease, a stop takes at most 30 s).
- **Error reports**: a failed job sends its error's class, a cleaned message (valid UTF-8, no control characters,
  at most 4 000 code points; standard exceptions that quote the job's data send their class only) and, for a
  raise, an exit or a throw, up to 50 stack frames (module, function, arity, file and line, never the arguments). A
  message you raise yourself is sent as it is: keep secrets out of it.

## Schedules

`JoobQ.register_schedule("nightly", "mail", "digest", cron: "0 3 * * *")` registers a job schedule, or updates it:
it is idempotent by its name (`:every` takes `"30 seconds"`, `"5 minutes"`, ...; see `JoobQ.Schedules`). It fires on
the server; editing, pausing and deleting schedules are done in the console.

## Workflows (JoobQ Pro)

The joobq-pro engine conducts every run of a registered blueprint: it retries, waits, compensates and keeps the
run's state; a worker only runs steps. `JoobQ.Workflow` builds and registers workflows, `JoobQ.Workflows` starts runs
and talks to them.

```elixir
import JoobQ.Workflow

checkout =
  new("checkout", task_queue: "workflows")
  |> step("reserve", &Shop.reserve/2,
    input: %{sku: input("sku")},
    retry: retry(attempts: 3, non_retryable: [Shop.OutOfStock]),
    compensate: compensation(&Shop.release/2))
  |> parallel(fn wf -> wf |> step("charge", &Shop.charge/2) |> step("receipt", &Shop.receipt/2, after: ["charge"]) end)
  |> wait_for("approval", signal: "approve", timeout_ms: 60_000)
  |> query("total", output("charge", "amount"))

children = [{JoobQ, url: url, token: token, workflows: [checkout]}]
```

- **Steps are jobs**: a handler is `fn input, job -> ... end` (or `&Mod.fun/2`), answering what `perform/1`
  answers; it runs once per attempt inside the instance's middleware, with the job's context and its
  `[:joobq, :job, *]` span (`task_kind` `"activity"`, `"compensation"` or `"update"`). Its input is the `input:`
  spec with each reference (`input/1`, `output/2`, `ref/1`) replaced by the value it names, the run's input when
  there is none. A numeric segment indexes a list (`input("items.0")`); a negative one is refused, as is
  `ref("steps.<step>")` without `.output` (the engine reads that as the step's status, not its output). The worker sets no timer of its own: the engine enforces `timeout_ms` and `heartbeat_ms` (call
  `JoobQ.Job.heartbeat/1` more often than that). A task for a step no workflow in `:workflows` has fails without
  retries (`JoobQ.UnknownActivity`). A step still running when a stop's `:drain_timeout` ends is left to its
  lease: the gateway fails that attempt as `LeaseExpired`, and the step runs again only if it declares retries.
- **Registration**: when the worker starts, before it registers itself, it sends each blueprint; a refused one
  (4xx) is logged with the gateway's code and message and not sent again, one that met a 5xx is sent again with the
  next registration. The gateway versions a blueprint by its structure and references only: changing a literal or
  an input mapping of a step (what the handler gets around the references) changes neither its hash nor its
  version, so runs in flight pick the change up.
- **Building** raises `ArgumentError` naming the workflow, the node and the option, never a value; a step may only
  read steps declared before it. `inspect/1` of a workflow shows its name, version and task queue only. An
  update's `message:` is static text stored in the blueprint: never put data in it.
- **Not retried**: a raised exception whose module is listed in the step's `non_retryable:` (as `inspect/1`
  prints it, e.g. `Shop.OutOfStock`), also when the handler returns `{:error, exception}` (the error's class is the
  exception's module), and `JoobQ.NonRetryable`.
- **`:workflows`** takes `%JoobQ.Workflow{}` values (what `new/2` and the builder return), not modules; each is
  checked when the instance starts, and one whose blueprint cannot be made is refused with an `ArgumentError`.

From a producer: `JoobQ.start_workflow(name,
input, id: "order-1")`, `signal_workflow/4`, `update_workflow/4`, `query_workflow/3`,
`upsert_workflow_attributes/2`, and the reads `describe_workflow/2`, `workflow_result/2` and `workflow_history/2`
(`JoobQ.Workflows`). Each is sent once; a run id is one percent-encoded path segment and stays out of telemetry.


## Batches (JoobQ Pro)

`JoobQ.create_batch(jobs, on_complete: {"mail", "report"}, context: %{"tenant" => "t1"})` creates, fills and seals a
batch of 1 to 1000 jobs in one call; `JoobQ.get_batch(id)` reads its progress. The gateway enqueues the callbacks
(`:on_failure` at the first dead member, `:on_complete` when all are counted, `:on_success` then too when none is
dead), each exactly once; the SDK never does. See `JoobQ.Batches` for what a member and a callback may carry.

## Rate limits (JoobQ Pro)

`JoobQ.rate_limit("tenant-1", 10, 60_000)` reserves one of 10 slots a minute for the key `"tenant-1"` on the gateway,
in one step: `{:ok, %{"allowed" => true}}`, or `"allowed" => false` with `"retry_after_ms"`. It paces your own code
and never waits; without a Pro licence it answers `{:error, %JoobQ.Error{reason: :pro_required}}`.

## Telemetry

Spans (`:start`, then `:stop` or `:exception`, with `duration` in native units; see `JoobQ.Telemetry`):

| Event | Metadata | `:stop` adds |
|-------|----------|--------------|
| `[:joobq, :enqueue, *]` | `instance`, `queue`, `job_type` | `result`, `status`, `jid`, `reason` |
| `[:joobq, :schedule, *]` | `instance`, `schedule`, `queue`, `job_type` | `result`, `reason` |
| `[:joobq, :rate_limit, *]` | `instance`, `limit`, `period_ms` (never the key) | `result`, `allowed`, `reason` |
| `[:joobq, :batch, *]` | `instance`, `size` (`create_batch/2`; `get_batch/2` has only `:request`) | `result`, `reason`, `batch_id` |
| `[:joobq, :job, *]` | `instance`, `queue`, `job_type`, `jid`, `attempt`, `task_kind` | `outcome`, `retryable`, `error_class` |
| `[:joobq, :workflow, *]` | `instance`, `call` (`:start`, `:signal`, ...), `name` | `result`, `reason` |
| `[:joobq, :blueprint, *]` | `instance`, `workflow` | `result` (`:registered`, `:refused`, `:retry`) |
| `[:joobq, :request, *]` | `name`, `method`, `path` | `status` |

A workflow step's job span is a job's: it adds only `task_kind` (`"activity"`, `"compensation"`, `"update"`; `"job"`
for a job), and its `job_type` is the activity name (`"<workflow>/<step>"`).

`:exception` adds `kind` and `error_class` only. None of JoobQ's own events carries a payload, an output, a lease
token or the SDK token. Finch's own `[:finch, ...]` events carry the request (the token in its headers, the body):
do not forward those to a sink you do not trust.

## Raising variants

`enqueue!/4` and `enqueue_many!/2` raise the error instead of answering it; the other calls (schedules, rate limits,
batches, workflows) answer `{:ok, answer}` or `{:error, %JoobQ.Error{}}` only.

## Retries and limits

A producer call is sent once and never retried. An error with reason `:transport` (no answer) or `:bad_response`
(a 2xx answer that is not JSON) MAY mean the job was stored: the gateway can store it before the connection breaks,
and it does not deduplicate by jid. Give a job a `unique_key` to make sending it again safe.

- `:request_timeout` (60 000 ms by default) bounds each call, until the whole answer is in.
- Pooled connections idle for 5 seconds are closed, below the usual proxy keep-alive limits.
- Finch emits its own `[:finch, :request, :*]` telemetry events whose metadata carries the request, headers included: do
  not attach a handler to them that logs the metadata, or the token is logged.
  Their request also carries the URL, and a worker's own heartbeat call (`/sdk/tasks/<lease token>/heartbeat`, the
  gateway's only route for it) has a lease token in it; JoobQ's `[:joobq, :request]` events name that call by its
  pattern, `/tasks/:token/heartbeat`.
- A worker keeps its lease tokens in a public named ETS table (`<instance>.Leases`) that any process in the VM can
  read; `inspect/1` of a `JoobQ.Job` leaves out its lease token, payload and context, but
  `inspect(job, structs: false)` and Erlang's `~p` (`:io_lib.format/2`, some crash reports) print them: the language
  cannot prevent it, so do not log a job that way.
- The size of an answer is not capped yet (a follow-up); the gateway's own answers are small.
- `:url` must be `https://`; `http://` only for a loopback host or with `allow_insecure: true` (development only).

More about JoobQ: https://www.joobq.io

## License

MIT (see `LICENSE`).