FROM signoz/signoz-otel-collector:v0.144.2

# Bake in collector and manager configs
COPY otel-collector-config.yaml /etc/otel-collector-config.yaml
COPY otel-collector-opamp-config.yaml /etc/manager-config.yaml

ENV OTEL_RESOURCE_ATTRIBUTES=host.name=signoz-host,os.type=linux
ENV LOW_CARDINAL_EXCEPTION_GROUPING=false
ENV SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_DSN=tcp://clickhouse:9000
ENV SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_CLUSTER=cluster
ENV SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_REPLICATION=true
ENV SIGNOZ_OTEL_COLLECTOR_TIMEOUT=10m

EXPOSE 4317 4318

# Run migration sync check, then start collector with configs
ENTRYPOINT ["/bin/sh", "-c"]
CMD ["/signoz-otel-collector migrate sync check && /signoz-otel-collector --config=/etc/otel-collector-config.yaml --manager-config=/etc/manager-config.yaml --copy-path=/var/tmp/collector-config.yaml"]
