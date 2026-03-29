FROM signoz/signoz-otel-collector:v0.144.2

ENV SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_DSN=tcp://clickhouse:9000
ENV SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_CLUSTER=cluster
ENV SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_REPLICATION=true
ENV SIGNOZ_OTEL_COLLECTOR_TIMEOUT=10m

# Run all migrations in sequence, then exit
ENTRYPOINT ["/bin/sh", "-c"]
CMD ["/signoz-otel-collector migrate bootstrap && /signoz-otel-collector migrate sync up && /signoz-otel-collector migrate async up"]
