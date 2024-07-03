#
# Dockerfile to create a Compute Server Node
# Define the build stage
FROM ubuntu:20.04 as build
ARG GRB_VERSION=11.0.3
ARG GRB_SHORT_VERSION=11.0
ARG TARGETPLATFORM

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
       ca-certificates \
       wget \
    && update-ca-certificates

WORKDIR /opt

COPY gurobi_server${GRB_VERSION}_armlinux64.tar.gz .
COPY gurobi_server${GRB_VERSION}_linux64.tar.gz .

# Install dependencies and download the appropriate Gurobi package
RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        export GRB_PLATFORM="armlinux64"; \
      else \
        export GRB_PLATFORM="linux64"; \
    fi \
    && tar -xvf gurobi_server${GRB_VERSION}_${GRB_PLATFORM}*.tar.gz \
    && rm *.tar.gz \
    && mv gurobi_server*  gurobi_server\
    && mv gurobi_server/${GRB_PLATFORM}  gurobi_server/linux
# After the file renaming, a clean image is built

# Define the final stage
FROM ubuntu:20.04 AS package

ARG GRB_VERSION=11.0.3
ARG TARGETPLATFORM

LABEL vendor="Gurobi"
LABEL version=${GRB_VERSION}

# Update the system and install necessary packages
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
       ca-certificates \
       p7zip-full \
       zip \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# Copy files from the build stage
COPY --from=build /opt/gurobi_server/linux/bin/grb_wlsproxy ./gurobi_server/linux/bin/grb_wlsproxy
COPY --from=build /opt/gurobi_server/linux/bin/grb_wlsproxy.cnf ./gurobi_server/linux/bin/grb_wlsproxy.cnf
#COPY --from=build /opt/gurobi_server/linux/EULA.pdf ./gurobi_server/linux/EULA.pdf


# Set environment variables and permissions
ENV GRB_HOME=/opt/gurobi_server/linux
ENV PATH $GRB_HOME/bin:$PATH
WORKDIR $GRB_HOME/bin

# Changes group permissions to run as a non-root user for better security
RUN chgrp -R 0 /opt/gurobi_server/linux/bin && \
    chmod -R g=u /opt/gurobi_server/linux/bin

# User to run the container
USER 1001

# Expose command line
ENTRYPOINT ["grb_wlsproxy", "--port=61099", "--console-ts"]
EXPOSE 61099
