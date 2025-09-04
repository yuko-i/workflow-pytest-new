
# non-interactive
ENV DEBIAN_FRONTEND=noninteractive

# 必須ツールの導入
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates curl gnupg lsb-release \
 && rm -rf /var/lib/apt/lists/*

# gcsfuse の APT レポジトリを追加（Ubuntu 22.04 jammy）
RUN set -eux; \
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.asc] https://packages.cloud.google.com/apt gcsfuse-$(lsb_release -c -s) main" \
      > /etc/apt/sources.list.d/gcsfuse.list; \
    curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
      | tee /usr/share/keyrings/cloud.google.asc >/dev/null

# gcsfuse をインストール（FUSE3 は依存で入ることが多いですが明示的に入れても可）
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcsfuse fuse3 \
 && rm -rf /var/lib/apt/lists/*
