# JoobQ SDK for PHP

**Background jobs and workflows for PHP, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![Packagist](https://img.shields.io/packagist/v/azutoolkit/joobq)](https://packagist.org/packages/azutoolkit/joobq) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

```sh
composer require azutoolkit/joobq
```

A worker, as a complete program:

```php
<?php
declare(strict_types=1);

require_once 'JoobQClient.php';

use JoobQ\ExecutionContext;
use JoobQ\Job;
use JoobQ\JoobQClient;
use JoobQ\JoobQWorker;

final class WelcomeEmail extends Job
{
    public static function jobName(): string
    {
        return 'welcome.email';
    }

    public static function perform(array $payload, ExecutionContext $ctx): mixed
    {
        echo "Welcome email sent to {$payload['to']}\n";
        return ['sent_to' => $payload['to']];
    }
}

$client = new JoobQClient((string)getenv('JOOBQ_GATEWAY_URL'), (string)getenv('JOOBQ_SDK_TOKEN'));
$worker = new JoobQWorker($client, 'welcome-worker-1', ['default']);
$worker->registerJob(WelcomeEmail::class);
$worker->run();
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```php
$client->enqueue(WelcomeEmail::class, ['to' => 'ada@example.com']);
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `enqueueMany`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`createBatch`,
`getBatch`) and a shared sliding-window rate limit (`rateLimit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: PHP 8.1 or newer with the `curl` and `json` extensions; `pcntl` for graceful shutdown.
- Install: `composer require azutoolkit/joobq`.
- Use: the namespace `JoobQ`, loaded by Composer's autoloader (`require 'vendor/autoload.php'`).
- Run: boot your application, then start `JoobQWorker` headless in a process of its own; handlers use the application's
  own services. With `pcntl` and `posix` loaded the worker always forks a pool (whatever `concurrency` is), each pool
  process a copy of the application made at the fork: open connections lazily or in `onProcessStart: fn () => ...`,
  which each pool process calls once before it takes tasks (like Resque's `after_fork`). When it throws, that process
  logs `joobq: onProcessStart failed: <message>` and ends without taking a task; its replacement starts after a pause
  that doubles from 0.25 s to 5 s, and after 5 such processes in a row `run()` throws. Without `pcntl` or `posix` the
  worker runs one task at a time in its own process and does not call `onProcessStart`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [Java](https://central.sonatype.com/artifact/io.joobq/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).