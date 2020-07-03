FROM haproxy:2.1.7-alpine

RUN set -x && \
    apk add --no-cache vim

# Expose port 6443 for Kubernetes API
EXPOSE 6443

# Copy over the pre-defined haproxy.cfg
COPY files/haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg