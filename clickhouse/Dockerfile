FROM clickhouse/clickhouse-server:25.5.6

# Download histogramQuantile UDF binary at build time
# (replaces the init-clickhouse container from docker-compose)
RUN apt-get update && apt-get install -y --no-install-recommends wget ca-certificates \
    && mkdir -p /var/lib/clickhouse/user_scripts \
    && ARCH=$(uname -m | sed 's/aarch64/arm64/' | sed 's/x86_64/amd64/') \
    && wget -O /tmp/histogram-quantile.tar.gz \
       "https://github.com/SigNoz/signoz/releases/download/histogram-quantile%2Fv0.0.1/histogram-quantile_linux_${ARCH}.tar.gz" \
    && tar -xzf /tmp/histogram-quantile.tar.gz -C /tmp \
    && mv /tmp/histogram-quantile /var/lib/clickhouse/user_scripts/histogramQuantile \
    && chmod +x /var/lib/clickhouse/user_scripts/histogramQuantile \
    && rm /tmp/histogram-quantile.tar.gz \
    && apt-get purge -y wget && apt-get autoremove -y && rm -rf /var/lib/apt/lists/*

# Bake in SigNoz ClickHouse configuration
COPY config.xml /etc/clickhouse-server/config.xml
COPY users.xml /etc/clickhouse-server/users.xml
COPY custom-function.xml /etc/clickhouse-server/custom-function.xml
COPY cluster.xml /etc/clickhouse-server/config.d/cluster.xml

ENV CLICKHOUSE_SKIP_USER_SETUP=1
