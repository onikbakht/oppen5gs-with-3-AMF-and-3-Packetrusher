# syntax=docker/dockerfile:1

# Global build-time arguments
ARG OPEN5GS_VERSION="v2.7.2"
ARG UBUNTU_VERSION="jammy"

FROM base-open5gs:${OPEN5GS_VERSION} AS builder
FROM ubuntu:${UBUNTU_VERSION}

# Target container architecture
ARG TARGETARCH

# Build-time arguments
ARG DEBIAN_FRONTEND="noninteractive"
ARG YQ_VERSION="v4.30.6"

# Install dependencies
RUN apt-get update && apt-get install libssl-dev libtalloc-dev libtins-dev \
    libyaml-dev iproute2 iptables wget libmicrohttpd-dev -y

# Install yq
RUN wget --quiet https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_${TARGETARCH} -O /usr/bin/yq && chmod +x /usr/bin/yq

# Copy executable, default config, and libraries from builder
COPY --from=builder /open5gs/install/bin/open5gs-upfd /usr/local/bin/open5gs-upfd
COPY --from=builder /open5gs/install/etc/open5gs/upf.yaml /etc/open5gs/default/upf.yaml
COPY --from=builder /open5gs/install/lib/*/libogsproto.so.2 /open5gs/install/lib/*/libogscore.so.2 \
/open5gs/install/lib/*/libogsapp.so.2 /open5gs/install/lib/*/libogspfcp.so.2 /open5gs/install/lib/*/libogsgtp.so.2 \
/open5gs/install/lib/*/libogstun.so.2 /open5gs/install/lib/*/libogsipfw.so.2 /open5gs/install/lib/*/libogsmetrics.so.2 \
/open5gs/install/lib/*/libprom.so /usr/local/lib/

# Copy helper_functions and entrypoint scripts
COPY ./helper_functions.sh /usr/local/bin/helper_functions.sh
COPY ./entrypoint.sh /usr/local/bin/entrypoint.sh

# Ensure scripts are executable
RUN chmod +x /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/helper_functions.sh

# Configure dynamically linked libraries
RUN ldconfig

# Create directory to store logs
RUN mkdir -p /var/log/open5gs/

# Set absolute path for entrypoint
ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]

# Provide arguments to entrypoint
CMD [ "-c", "/etc/open5gs/default/upf.yaml" ]
