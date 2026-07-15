FROM oven/bun:1.3.12-debian AS builder

WORKDIR /src

COPY package.json bun.lock ./
RUN bun install --frozen-lockfile

COPY adapters/package.json adapters/bun.lock ./adapters/
RUN cd adapters && bun install --frozen-lockfile

COPY desktop/package.json desktop/bun.lock ./desktop/
RUN cd desktop && bun install --frozen-lockfile

COPY . .
RUN cd desktop && bun run build
RUN SIDECAR_TARGET_TRIPLE=x86_64-unknown-linux-gnu bun run ./desktop/scripts/build-sidecars.ts

FROM debian:12-slim

RUN apt-get update \
  && apt-get install -y --no-install-recommends bash ca-certificates curl git openssh-client procps ripgrep \
  && rm -rf /var/lib/apt/lists/* \
  && useradd --create-home --uid 10001 sscops \
  && mkdir -p /app /data/.claude /workspace \
  && chown -R sscops:sscops /app /data /workspace

COPY --from=builder --chown=sscops:sscops \
  /src/desktop/src-tauri/binaries/claude-sidecar-x86_64-unknown-linux-gnu \
  /app/claude-sidecar
COPY --from=builder --chown=sscops:sscops /src/desktop/dist /app/dist

USER sscops
WORKDIR /workspace

ENV HOME=/home/sscops \
  CLAUDE_CONFIG_DIR=/data/.claude \
  CLAUDE_APP_ROOT=/app \
  CLAUDE_CLI_PATH=/app/claude-sidecar \
  CLAUDE_H5_DIST_DIR=/app/dist

ENTRYPOINT ["/app/claude-sidecar"]
