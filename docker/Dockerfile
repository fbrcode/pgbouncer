# Use the base bitnami/pgbouncer image
FROM bitnami/pgbouncer:1.23.1

# Set shell to bash
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install additional tools
USER root
RUN apt-get update
RUN apt-get install -y iputils-ping
RUN apt-get install -y postgresql-client

# Switch back to the default non-root user for security
USER 1001

# Expose the default pgbouncer port
EXPOSE 6432

# Entrypoint remains the same (default in bitnami/pgbouncer)
