# JoobQ SDK for Java

**Background jobs and workflows for Java, on queues that workers in eight other languages share.**
Define a job, run a worker inside your own application, enqueue from any service, and inspect every job in the
JoobQUI console of your self-hosted server.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[Quickstart](https://www.joobq.io/docs/quickstart.html) · [Documentation](https://www.joobq.io/docs/develop/) · [The console](https://www.joobq.io/console) · [joobq.io](https://www.joobq.io)

## 30-second example

A worker, as a complete program:

```java
import io.joobq.JoobQClient;
import io.joobq.JoobQClient.JobDefinition;
import io.joobq.JoobQClient.JoobQWorker;
import java.util.List;
import java.util.Map;

public class WelcomeWorker {
    public static void main(String[] args) throws Exception {
        JobDefinition welcomeEmail = JobDefinition.of("welcome.email", "default", 3, (payload, ctx) -> {
            Object to = ((Map<?, ?>) payload).get("to");
            System.out.println("Welcome email sent to " + to);
            return Map.of("sent_to", to);
        });

        JoobQClient client = new JoobQClient(System.getenv("JOOBQ_GATEWAY_URL"), System.getenv("JOOBQ_SDK_TOKEN"));
        JoobQWorker worker = new JoobQWorker(client, "welcome-worker-1", List.of("default"));
        worker.register(welcomeEmail);
        worker.run();
    }
}
```

Enqueue the job from anywhere in your application, or from a service written in another language:

```java
String jid = client.enqueue(welcomeEmail, Map.of("to", "ada@example.com"), new JoobQClient.EnqueueOptions());
```

Both read the server's origin and its SDK token from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`. No server yet? The
[Quickstart](https://www.joobq.io/docs/quickstart.html) starts one with Docker Compose.

## What the SDK does

A thin worker and producer client for the JoobQ SDK gateway of a self-hosted JoobQUI server: register jobs and
workflow blueprints, run their tasks with retries, heartbeats and graceful shutdown, and enqueue jobs (up to 500 in one
call with `enqueueMany`) or start workflows. With JoobQ Pro on the server: batches with completion callbacks (`createBatch`,
`getBatch`) and a shared sliding-window rate limit (`rateLimit`).
JoobQUI version 0.8.0; an SDK talks to a server of its own version or later.

- Requirements: Java 17 or newer; JDK only (java.net.http).
- Install: the Maven coordinate `io.joobq:joobq:0.8.0` (Gradle: `implementation("io.joobq:joobq:0.8.0")`).
- Use: the class `io.joobq.JoobQClient`.
- Connect: the server's origin and its SDK token, read by the examples from `JOOBQ_GATEWAY_URL` and `JOOBQ_SDK_TOKEN`.
- Documentation: https://www.joobq.io/docs/develop/ (workers, jobs, workflows, the gateway contract).

## One queue, nine languages

A job enqueued from one language can be run by a worker in another. The other SDKs:
[Python](https://pypi.org/project/joobq/) · [TypeScript](https://www.npmjs.com/package/joobq) · [Ruby](https://rubygems.org/gems/joobq) · [Go](https://pkg.go.dev/github.com/azutoolkit/joobq-sdk-go) · [Rust](https://crates.io/crates/joobq) · [PHP](https://packagist.org/packages/azutoolkit/joobq) · [.NET](https://www.nuget.org/packages/JoobQ) · [Elixir](https://hex.pm/packages/joobq_ex).
Crystal applications use the JoobQ shards directly and share the same queues.

## License

MIT (see `LICENSE`).