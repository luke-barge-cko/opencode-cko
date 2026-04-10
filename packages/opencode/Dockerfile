FROM alpine AS base

# Disable the runtime transpiler cache by default inside Docker containers.
# On ephemeral containers, the cache is not useful
ARG BUN_RUNTIME_TRANSPILER_CACHE_PATH=0
ENV BUN_RUNTIME_TRANSPILER_CACHE_PATH=${BUN_RUNTIME_TRANSPILER_CACHE_PATH}
RUN apk add libgcc libstdc++ ripgrep

# Reduce outbound calls in containerized runs (override at `docker run` if needed).
# - models.dev: no startup/hourly refresh; bundled snapshot or mount + OPENCODE_MODELS_PATH.
# - skills.urls in config still fetches over HTTP — omit those entries in mounted config.
# - Config dependency install (Npm.install) and provider APIs have no dedicated OFF switch here.
# Also avoid at runtime (defaults are already safe): OPENCODE_EXPERIMENTAL, OPENCODE_ENABLE_EXA /
# OPENCODE_EXPERIMENTAL_EXA (Exa web tools), OPENCODE_AUTO_SHARE, OTEL_EXPORTER_OTLP_* .
ENV OPENCODE_DISABLE_MODELS_FETCH=1 \
    OPENCODE_PURE=1 \
    OPENCODE_DISABLE_AUTOUPDATE=1 \
    OPENCODE_DISABLE_LSP_DOWNLOAD=1 \
    OPENCODE_DISABLE_EXTERNAL_SKILLS=1 \
    OPENCODE_DISABLE_SHARE=1

FROM base AS build-amd64
COPY dist/opencode-linux-x64-baseline-musl/bin/opencode /usr/local/bin/opencode

FROM base AS build-arm64
COPY dist/opencode-linux-arm64-musl/bin/opencode /usr/local/bin/opencode

ARG TARGETARCH
FROM build-${TARGETARCH}
RUN opencode --version
ENTRYPOINT ["opencode"]
