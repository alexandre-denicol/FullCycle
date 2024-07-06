FROM alpine:latest as builder
WORKDIR /app
COPY fullcycle ./


FROM hello-world:latest
COPY --from=builder /app .
ENTRYPOINT ["./fullcycle"]