# Traefik Hub (Experimental)

## Overview

Once the Traefik Hub Experimental feature is enabled in Traefik,
Traefik and its local agent communicate together.
This agent can:

* get the Traefik metrics to display them in the Traefik Hub UI
* secure the Traefik routers
* provide ACME certificates to Traefik
* transfer requests from the SaaS Platform to Traefik (and then avoid the users to expose directly their infrastructure on the internet)

!!! warning "Traefik Hub EntryPoint"

    When the Traefik Hub feature is enabled, Traefik exposes some services meant for the Traefik Hub Agent on a dedicated entryPoint (on port `9900` by default).
    Given their sensitive nature, those services should not be publicly exposed.
    Also this dedicated entryPoint, regardless of how it is created (default, or user-defined), should not be used by anything other than the Hub Agent.

!!! important "Learn More About Traefik Hub"

    This section is intended only as a brief overview for Traefik users who are not familiar with Traefik Hub.
    To explore all that Traefik Hub has to offer, please consult the [Traefik Hub Documentation](https://doc.traefik.io/traefik-hub).

!!! Note "Prerequisites"

    * Traefik Hub is compatible with Traefik Proxy 2.7 or later.
    * The Traefik Hub Agent must be installed to connect to the Traefik Hub platform.
    * Activate this feature in the experimental section of the static configuration.

!!! example "Minimal Static Configuration to Activate Traefik Hub"

    ```yaml tab="File (YAML)"
    experimental:
      hub: true

    hub:
      tls:
        insecure: true

    metrics:
        prometheus: {}
    ```

    ```toml tab="File (TOML)"
    [experimental]
      hub = true

    [hub]
      [hub.tls]
        insecure = true

    [metrics]
      [metrics.prometheus]
    ```

    ```bash tab="CLI"
    --experimental.hub
    --hub.tls.insecure=true
    --metrics.prometheus=true
    ```

## Configuration

### `entryPoint`

_Optional, Default="traefik-hub"_

Defines the entryPoint that exposes data for Traefik Hub Agent.

!!! info

    * If no entryPoint is defined, a default `traefik-hub` entryPoint is created (on port `9900`).
    * If defined, the value must match an existing entryPoint name.
    * This dedicated Traefik Hub entryPoint should not be used by anything other than Traefik Hub.

```yaml tab="File (YAML)"
entryPoints:
  hub-ep: ":8000"

hub:
  entryPoint: "hub-ep"
```

```toml tab="File (TOML)"
[entryPoints.hub-ep]
  address = ":8000"

[hub]
  entryPoint = "hub-ep"
```

```bash tab="CLI"
--entrypoints.hub-ep.address=:8000
--hub.entrypoint=hub-ep
```

### `tls`

_Required, Default=None_

This section allows configuring mutual TLS connection between Traefik Proxy and the Traefik Hub Agent.
The key and the certificate are the credentials for Traefik Proxy as a TLS client.
The certificate authority authenticates the Traefik Hub Agent certificate.

!!! note "Certificate Domain"

    The certificate must be valid for the `proxy.traefik` domain.

!!! note "Certificates Definition"

    Certificates can be defined either by their content or their path.

!!! note "Insecure Mode"

    The `insecure` option is mutually exclusive with any other option.

```yaml tab="File (YAML)"
hub:
  tls:
    ca: /path/to/ca.pem
    cert: /path/to/cert.pem
    key: /path/to/key.pem
```

```toml tab="File (TOML)"
[hub.tls]
  ca= "/path/to/ca.pem"
  cert= "/path/to/cert.pem"
  key= "/path/to/key.pem"
```

```bash tab="CLI"
--hub.tls.ca=/path/to/ca.pem
--hub.tls.cert=/path/to/cert.pem
--hub.tls.key=/path/to/key.pem
```

### `tls.ca`

The certificate authority authenticates the Traefik Hub Agent certificate.

```yaml tab="File (YAML)"
hub:
  tls:
    ca: |-
      -----BEGIN CERTIFICATE-----
      MIIBcjCCARegAwIBAgIQaewCzGdRz5iNnjAiEoO5AzAKBggqhkjOPQQDAjASMRAw
      DgYDVQQKEwdBY21lIENvMCAXDTIyMDMyMTE2MTY0NFoYDzIxMjIwMjI1MTYxNjQ0
      WjASMRAwDgYDVQQKEwdBY21lIENvMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
      ZaKYPj2G8Hnmju6jbHt+vODwKqNDVQMH5nxhtAgSUZS61mLWwZvvUhIYLNPwHz8a
      x8C7+cuihEC6Tzvn8DeGeKNNMEswDgYDVR0PAQH/BAQDAgeAMBMGA1UdJQQMMAoG
      CCsGAQUFBwMBMAwGA1UdEwEB/wQCMAAwFgYDVR0RBA8wDYILZXhhbXBsZS5jb20w
      CgYIKoZIzj0EAwIDSQAwRgIhAO8sucDGY+JOrNgQg1a9ZqqYvbxPFnYsSZr7F/vz
      aUX2AiEAilZ+M5eX4RiMFc3nlm9qVs1LZhV3dZW/u80/mPQ/oaY=
      -----END CERTIFICATE-----
```

```toml tab="File (TOML)"
[hub.tls]
  ca = """-----BEGIN CERTIFICATE-----
MIIBcjCCARegAwIBAgIQaewCzGdRz5iNnjAiEoO5AzAKBggqhkjOPQQDAjASMRAw
DgYDVQQKEwdBY21lIENvMCAXDTIyMDMyMTE2MTY0NFoYDzIxMjIwMjI1MTYxNjQ0
WjASMRAwDgYDVQQKEwdBY21lIENvMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
ZaKYPj2G8Hnmju6jbHt+vODwKqNDVQMH5nxhtAgSUZS61mLWwZvvUhIYLNPwHz8a
x8C7+cuihEC6Tzvn8DeGeKNNMEswDgYDVR0PAQH/BAQDAgeAMBMGA1UdJQQMMAoG
CCsGAQUFBwMBMAwGA1UdEwEB/wQCMAAwFgYDVR0RBA8wDYILZXhhbXBsZS5jb20w
CgYIKoZIzj0EAwIDSQAwRgIhAO8sucDGY+JOrNgQg1a9ZqqYvbxPFnYsSZr7F/vz
aUX2AiEAilZ+M5eX4RiMFc3nlm9qVs1LZhV3dZW/u80/mPQ/oaY=
-----END CERTIFICATE-----"""
```

```bash tab="CLI"
--hub.tls.ca=-----BEGIN CERTIFICATE-----
MIIBcjCCARegAwIBAgIQaewCzGdRz5iNnjAiEoO5AzAKBggqhkjOPQQDAjASMRAw
DgYDVQQKEwdBY21lIENvMCAXDTIyMDMyMTE2MTY0NFoYDzIxMjIwMjI1MTYxNjQ0
WjASMRAwDgYDVQQKEwdBY21lIENvMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
ZaKYPj2G8Hnmju6jbHt+vODwKqNDVQMH5nxhtAgSUZS61mLWwZvvUhIYLNPwHz8a
x8C7+cuihEC6Tzvn8DeGeKNNMEswDgYDVR0PAQH/BAQDAgeAMBMGA1UdJQQMMAoG
CCsGAQUFBwMBMAwGA1UdEwEB/wQCMAAwFgYDVR0RBA8wDYILZXhhbXBsZS5jb20w
CgYIKoZIzj0EAwIDSQAwRgIhAO8sucDGY+JOrNgQg1a9ZqqYvbxPFnYsSZr7F/vz
aUX2AiEAilZ+M5eX4RiMFc3nlm9qVs1LZhV3dZW/u80/mPQ/oaY=
-----END CERTIFICATE-----
```

### `tls.cert`

The TLS certificate for Traefik Proxy as a TLS client.

!!! note "Certificate Domain"

    The certificate must be valid for the `proxy.traefik` domain.

```yaml tab="File (YAML)"
hub:
  tls:
    ca: |-
      -----BEGIN CERTIFICATE-----
      MIIBcjCCARegAwIBAgIQaewCzGdRz5iNnjAiEoO5AzAKBggqhkjOPQQDAjASMRAw
      DgYDVQQKEwdBY21lIENvMCAXDTIyMDMyMTE2MTY0NFoYDzIxMjIwMjI1MTYxNjQ0
      WjASMRAwDgYDVQQKEwdBY21lIENvMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
      ZaKYPj2G8Hnmju6jbHt+vODwKqNDVQMH5nxhtAgSUZS61mLWwZvvUhIYLNPwHz8a
      x8C7+cuihEC6Tzvn8DeGeKNNMEswDgYDVR0PAQH/BAQDAgeAMBMGA1UdJQQMMAoG
      CCsGAQUFBwMBMAwGA1UdEwEB/wQCMAAwFgYDVR0RBA8wDYILZXhhbXBsZS5jb20w
      CgYIKoZIzj0EAwIDSQAwRgIhAO8sucDGY+JOrNgQg1a9ZqqYvbxPFnYsSZr7F/vz
      aUX2AiEAilZ+M5eX4RiMFc3nlm9qVs1LZhV3dZW/u80/mPQ/oaY=
      -----END CERTIFICATE-----
```

```toml tab="File (TOML)"
[hub.tls]
  cert = """-----BEGIN CERTIFICATE-----
MIIBcjCCARegAwIBAgIQaewCzGdRz5iNnjAiEoO5AzAKBggqhkjOPQQDAjASMRAw
DgYDVQQKEwdBY21lIENvMCAXDTIyMDMyMTE2MTY0NFoYDzIxMjIwMjI1MTYxNjQ0
WjASMRAwDgYDVQQKEwdBY21lIENvMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
ZaKYPj2G8Hnmju6jbHt+vODwKqNDVQMH5nxhtAgSUZS61mLWwZvvUhIYLNPwHz8a
x8C7+cuihEC6Tzvn8DeGeKNNMEswDgYDVR0PAQH/BAQDAgeAMBMGA1UdJQQMMAoG
CCsGAQUFBwMBMAwGA1UdEwEB/wQCMAAwFgYDVR0RBA8wDYILZXhhbXBsZS5jb20w
CgYIKoZIzj0EAwIDSQAwRgIhAO8sucDGY+JOrNgQg1a9ZqqYvbxPFnYsSZr7F/vz
aUX2AiEAilZ+M5eX4RiMFc3nlm9qVs1LZhV3dZW/u80/mPQ/oaY=
-----END CERTIFICATE-----"""
```

```bash tab="CLI"
--hub.tls.cert=-----BEGIN CERTIFICATE-----
MIIBcjCCARegAwIBAgIQaewCzGdRz5iNnjAiEoO5AzAKBggqhkjOPQQDAjASMRAw
DgYDVQQKEwdBY21lIENvMCAXDTIyMDMyMTE2MTY0NFoYDzIxMjIwMjI1MTYxNjQ0
WjASMRAwDgYDVQQKEwdBY21lIENvMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
ZaKYPj2G8Hnmju6jbHt+vODwKqNDVQMH5nxhtAgSUZS61mLWwZvvUhIYLNPwHz8a
x8C7+cuihEC6Tzvn8DeGeKNNMEswDgYDVR0PAQH/BAQDAgeAMBMGA1UdJQQMMAoG
CCsGAQUFBwMBMAwGA1UdEwEB/wQCMAAwFgYDVR0RBA8wDYILZXhhbXBsZS5jb20w
CgYIKoZIzj0EAwIDSQAwRgIhAO8sucDGY+JOrNgQg1a9ZqqYvbxPFnYsSZr7F/vz
aUX2AiEAilZ+M5eX4RiMFc3nlm9qVs1LZhV3dZW/u80/mPQ/oaY=
-----END CERTIFICATE-----
```

### `tls.key`

The TLS key for Traefik Proxy as a TLS client.

```yaml tab="File (YAML)"
hub:
  tls:
    key: |-
      -----BEGIN PRIVATE KEY-----
      MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgm+XJ3LVrTbbirJea
      O+Crj2ADVsVHjMuiyd72VE3lgxihRANCAARlopg+PYbweeaO7qNse3684PAqo0NV
      AwfmfGG0CBJRlLrWYtbBm+9SEhgs0/AfPxrHwLv5y6KEQLpPO+fwN4Z4
      -----END PRIVATE KEY-----
```

```toml tab="File (TOML)"
[hub.tls]
  key = """-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgm+XJ3LVrTbbirJea
O+Crj2ADVsVHjMuiyd72VE3lgxihRANCAARlopg+PYbweeaO7qNse3684PAqo0NV
AwfmfGG0CBJRlLrWYtbBm+9SEhgs0/AfPxrHwLv5y6KEQLpPO+fwN4Z4
-----END PRIVATE KEY-----"""
```

```bash tab="CLI"
--hub.tls.key=-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQgm+XJ3LVrTbbirJea
O+Crj2ADVsVHjMuiyd72VE3lgxihRANCAARlopg+PYbweeaO7qNse3684PAqo0NV
AwfmfGG0CBJRlLrWYtbBm+9SEhgs0/AfPxrHwLv5y6KEQLpPO+fwN4Z4
-----END PRIVATE KEY-----
```

### `tls.insecure`

_Optional, Default=false_

Enables an insecure TLS connection that uses default credentials,
and which has no peer authentication between Traefik Proxy and the Traefik Hub Agent.
The `insecure` option is mutually exclusive with any other option.

!!! warning "Security Consideration"

    Do not use this setup in production.
    This option implies sensitive data can be exposed to potential malicious third-party programs.

```yaml tab="File (YAML)"
hub:
  insecure: true
```

```toml tab="File (TOML)"
[hub]
  insecure = true
```

```bash tab="CLI"
--hub.insecure=true
```
