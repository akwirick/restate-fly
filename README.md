# Restate running on fly.io

This guid helps get restate running on fly.io.   Before you start, you'll need to pick a unique name for the fly app.

```zsh
export RESTATE_APP_NAME="my_app"
export RESTATE_APP_VOLUME="${RESTATE_APP_NAME}_data"
```


# Steps to configure
1. [Install flyctl](https://fly.io/docs/hands-on/install-flyctl/)
1. Login to docker - `echo $GITHUB_TOKEN | docker login -u $GITHUB_USER --password-stdin`
1. Pull the restate image `docker pull ghcr.io/restatedev/restate-dist:latest`
1. Create the deployment - `fly launch -i ghcr.io/restatedev/restate-dist:latest --name $RESTATE_APP_NAME`
1. Create an attached volume - `fly vol create $RESTATE_APP_DATA -a $RESTATE_APP_NAME`
1. Configure the deployment `fly.toml` with special attention to services and mounts

```toml
# Ingress (gRPC + Connect Protocol) - Acts as an API gateway for all services registered with Restate
[[services]]
  internal_port = 9090
  protocol = "tcp"
  
  # Configured as a gRPC service
  [[services.ports]]
      handlers = ["tls"]
      port = 9090
      tls_options = { "alpn" = ["h2"] }

# Storage (gRPC) - Exposes raw RocksDB read-only storage operations, used by the CLI
[[services]]
  internal_port = 9090
  protocol = "tcp"
  
  # Configured as a gRPC service
  [[services.ports]]
      handlers = ["tls"]
      port = 9090
      tls_options = { "alpn" = ["h2"] }

# Meta (REST) - Allows for CRUD operations on service metadata, eg for service registration
[[services]]
  internal_port = 8081
  protocol = "tcp"
  
  # Configured as a plain HTTP service
  [[services.ports]]
      handlers = ["http"]
      port = 8081
      force_https=true

[mounts]
  source="YOUR_VOLUME_NAME"
  destination="/target"
```  