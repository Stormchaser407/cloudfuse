# Stage 1: Builder (if cloudfuse is built from source, use this)
# For now assuming binary is pre-built
FROM ubuntu:22.04 as runtime

ENV DEBIAN_FRONTEND=noninteractive

# Install runtime dependencies only (no build tools)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      fuse3 \
      rsyslog \
      vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user (optional for security)
RUN useradd -m -u 1000 cloudfuse || true

# Create directories
RUN mkdir -p /usr/share/cloudfuse /mnt/blobfuse_mnt /tmp/blobfuse_temp && \
    chmod 777 /mnt/blobfuse_mnt && \
    chmod 777 /tmp/blobfuse_temp

# Configure FUSE
RUN echo "user_allow_other" >> /etc/fuse.conf

# Copy binary and config
COPY ./cloudfuse /usr/local/bin/cloudfuse
COPY ./config.yaml /usr/share/cloudfuse/
COPY ./11-cloudfuse.conf /etc/rsyslog.d/
COPY ./cloudfuse-logrotate /etc/logrotate.d/cloudfuse

# Create mount scripts
RUN echo "#!/bin/bash" > /usr/share/cloudfuse/cloudfuse-mount.sh && \
    echo "/sbin/rsyslogd" >> /usr/share/cloudfuse/cloudfuse-mount.sh && \
    echo "cloudfuse mount /mnt/blobfuse_mnt --config-file=/usr/share/cloudfuse/config.yaml --ignore-open-flags --foreground=true" >> /usr/share/cloudfuse/cloudfuse-mount.sh && \
    echo "trap 'cloudfuse unmount all' EXIT" >> /usr/share/cloudfuse/cloudfuse-mount.sh && \
    chmod +x /usr/share/cloudfuse/cloudfuse-mount.sh && \
    ln -s /usr/share/cloudfuse/cloudfuse-mount.sh /usr/local/bin/fuse

RUN echo "#!/bin/bash" > /usr/share/cloudfuse/cloudfuse-umount.sh && \
    echo "cloudfuse unmount all" >> /usr/share/cloudfuse/cloudfuse-umount.sh && \
    chmod +x /usr/share/cloudfuse/cloudfuse-umount.sh && \
    ln -s /usr/share/cloudfuse/cloudfuse-umount.sh /usr/local/bin/unfuse

ENTRYPOINT ["/usr/share/cloudfuse/cloudfuse-mount.sh"]
