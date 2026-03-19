# quarkus-tls-config-bug

Demonstrates that Quarkus TLS certificate configuration ignores profile-specific overrides in test mode, falling back to prod paths instead.

## The problem

When you define per-profile TLS key store paths in `application.properties`:

```properties
quarkus.http.tls-configuration-name=https
%test.quarkus.tls.https.key-store.pem.0.cert=server.crt
%test.quarkus.tls.https.key-store.pem.0.key=server.key
%prod.quarkus.tls.https.key-store.pem.0.cert=/app/certificates/tls.crt
%prod.quarkus.tls.https.key-store.pem.0.key=/app/certificates/tls.key
```

Quarkus should resolve the `%test` paths (`server.crt`, `server.key`) when running `@QuarkusTest`. Instead, it resolves the `%prod` paths and fails:

```
Caused by: java.io.UncheckedIOException: Unable to read file /app/certificates/tls.crt
    at io.quarkus.tls.runtime.config.TlsConfigUtils.read(TlsConfigUtils.java:52)
    ...
Caused by: java.nio.file.NoSuchFileException: /app/certificates/tls.crt
```

The test cert files (`server.crt`, `server.key`) are present in `src/main/resources/` and would load fine if the correct profile was being used.

## Reproducing

```bash
./gradlew clean test
```

The single `@QuarkusTest` (`GreetingResourceTest`) will fail because Quarkus can't start - it tries to load `/app/certificates/tls.crt` instead of `server.crt`.

## Requirements

* Java 25
* Quarkus 3.32.4
