# logger
slog handler with opentelemetry tracing support and init helper method

## usage

```go
package main

import (
    "context"
    "github.com/ttys3/logger"
    "github.com/ttys3/tracing-go"
    "go.opentelemetry.io/otel"
    "golang.org/x/exp/slog"
    "io"
)

func main() {
	h := logger.NewTracingHandler(logger.NewHandler("info", "json", "stderr"))
	slog.SetDefault(slog.New(h))

	ctx := context.Background()

	// set up a recording tracer, non-recording span will not get a trace_id
	tpShutdown, err := tracing.InitProvider(ctx, tracing.WithStdoutTrace())
	if err != nil {
		panic(err)
	}
	defer tpShutdown(context.Background())

	ctxWithSpan, newSpan := otel.Tracer("my-tracer-name").Start(ctx, "hello.Slog")
	defer newSpan.End()
	ctxLog := slog.With("foo", "bar").WithContext(ctxWithSpan)
	ctxLog.Info("hello world")
	ctxLog.With("foo", "bar").Error("have a nice day", io.ErrClosedPipe)
	ctxLog.Error("example error", io.ErrClosedPipe)
}
```

## issue

there's a bug in slog `With`,

we need wait https://go.dev/cl/459615 to be merged.