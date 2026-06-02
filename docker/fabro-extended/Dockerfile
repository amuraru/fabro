# Extended Fabro runtime image.
#
# A thin overlay on the upstream STABLE Fabro release image that adds the CLI
# toolchain Fabro's in-container ("local") sandbox needs. Most importantly it
# adds `bash`: the local sandbox shells out via `Command::new("bash")`
# (lib/crates/fabro-sandbox/src/local.rs), but the stock Fabro image is
# Alpine/BusyBox and ships no bash, so agent shell commands fail (ENOENT /
# exit 127). Installing via apk puts bash at /usr/bin/bash with its libs in
# /usr/lib — standard paths — so the sandbox finds and loads it with no
# PATH/LD_LIBRARY_PATH wiring. Useful wherever the local sandbox runs inside the
# server container (e.g. Kubernetes, where the Docker/Daytona sandboxes aren't
# available).
#
# FABRO_VERSION is injected by CI with the latest stable Fabro release.
ARG FABRO_VERSION=0.244.0
FROM ghcr.io/fabro-sh/fabro:${FABRO_VERSION}

USER root

# Pin the public Alpine repos for this build (the base image's hardened repo can
# require auth). The alpine signing keys are already present in the base. git +
# busybox net tools (nc/wget/ping/nslookup/ip/...) already exist in the base;
# this layer adds the gaps: bash, curl, jq, dig (bind-tools). Extend the list as
# workflows need more (e.g. coreutils findutils gawk python3 nodejs npm).
RUN printf '%s\n' \
      https://dl-cdn.alpinelinux.org/alpine/v3.23/main \
      https://dl-cdn.alpinelinux.org/alpine/v3.23/community \
      > /etc/apk/repositories \
 && apk add --no-cache \
      bash \
      curl \
      jq \
      bind-tools \
      git \
      openssh-client \
      ca-certificates \
 && bash --version | head -1 \
 && command -v curl jq dig

# ENTRYPOINT / CMD / USER are inherited from the base Fabro image (its entrypoint
# drops privileges; the deploying platform may set its own runAsUser). Do NOT
# override them — this image must behave exactly like the base, plus tools.
