# FastAPI gRPC Bridge - Security Features

This document covers the security features available in fastapi-grpc-bridge, including TLS and mutual TLS (mTLS) support.

## Overview

The fastapi-grpc-bridge package now supports secure gRPC connections with:

- **TLS (Transport Layer Security)**: Encrypts communication between client and server
- **mTLS (Mutual TLS)**: Provides bidirectional authentication with client certificates
- **Flexible Configuration**: Environment variables, dictionaries, or configuration objects
- **Certificate Management**: Built-in utilities for certificate handling and validation
- **Backward Compatibility**: Existing insecure configurations continue to work

## Quick Start

### Basic TLS Setup

```python
from fastapi import FastAPI
from fastapi_grpc_bridge import add_grpc_support, create_secure_grpc_config

app = FastAPI()

# Create secure configuration
config = create_secure_grpc_config(
    cert_file="certs/server.crt",
    key_file="certs/server.key",
    host="0.0.0.0",
    port=50051
)

# Add gRPC support with TLS
add_grpc_support(app, config=config)
```

### mTLS Setup

```python
from fastapi_grpc_bridge import GRPCSecurityConfig, TLSConfig, MTLSConfig

# Configure mTLS
mtls_config = GRPCSecurityConfig(
    host="0.0.0.0",
    port=50051,
    insecure=False,
    tls=TLSConfig(
        enabled=True,
        cert_file="certs/server.crt",
        key_file="certs/server.key"
    ),
    mtls=MTLSConfig(
        enabled=True,
        ca_cert_file="certs/ca.crt",
        client_cert_required=True
    )
)

add_grpc_support(app, config=mtls_config)
```

## Configuration Options

### GRPCSecurityConfig

The main configuration class that controls all security settings.

```python
from fastapi_grpc_bridge import GRPCSecurityConfig, TLSConfig, MTLSConfig

config = GRPCSecurityConfig(
    host="[::]",              # Host to bind to
    port=50051,               # Port to bind to
    max_workers=10,           # Number of worker threads
    insecure=True,            # Enable/disable insecure mode
    tls=TLSConfig(...),       # TLS configuration
    mtls=MTLSConfig(...)      # mTLS configuration
)
```

### TLSConfig

Configuration for TLS (one-way authentication).

```python
tls_config = TLSConfig(
    enabled=False,            # Enable TLS
    cert_file=None,           # Path to server certificate
    key_file=None,            # Path to server private key
    ca_cert_file=None,        # Path to CA certificate (optional)
    server_name=None          # Server name for SNI (optional)
)
```

### MTLSConfig

Configuration for mutual TLS (bidirectional authentication).

```python
mtls_config = MTLSConfig(
    enabled=False,            # Enable mTLS
    client_cert_required=True, # Require client certificates
    ca_cert_file=None,        # Path to CA certificate
    client_cert_file=None,    # Path to client certificate (for clients)
    client_key_file=None,     # Path to client private key (for clients)
    verify_client_cert=True   # Verify client certificates
)
```

## Configuration Methods

### 1. Configuration Objects

```python
from fastapi_grpc_bridge import GRPCSecurityConfig, TLSConfig

config = GRPCSecurityConfig(
    insecure=False,
    tls=TLSConfig(
        enabled=True,
        cert_file="server.crt",
        key_file="server.key"
    )
)

add_grpc_support(app, config=config)
```

### 2. Dictionary Configuration

```python
config_dict = {
    "host": "0.0.0.0",
    "port": 50051,
    "insecure": False,
    "tls": {
        "enabled": True,
        "cert_file": "certs/server.crt",
        "key_file": "certs/server.key"
    },
    "mtls": {
        "enabled": True,
        "ca_cert_file": "certs/ca.crt",
        "client_cert_required": True
    }
}

add_grpc_support(app, config_dict=config_dict)
```

### 3. Environment Variables

Set these environment variables:

```bash
# Server Configuration
export GRPC_HOST="0.0.0.0"
export GRPC_PORT="50051"
export GRPC_INSECURE="false"
export GRPC_MAX_WORKERS="10"

# TLS Configuration
export GRPC_TLS_ENABLED="true"
export GRPC_TLS_CERT_FILE="certs/server.crt"
export GRPC_TLS_KEY_FILE="certs/server.key"
export GRPC_TLS_CA_CERT_FILE="certs/ca.crt"

# mTLS Configuration
export GRPC_MTLS_ENABLED="true"
export GRPC_MTLS_CA_CERT_FILE="certs/ca.crt"
export GRPC_MTLS_CLIENT_CERT_REQUIRED="true"
```

Then use:

```python
from fastapi_grpc_bridge import GRPCSecurityConfig

config = GRPCSecurityConfig.from_env()
add_grpc_support(app, config=config)
```

## Standalone Server

### Basic Standalone Server

```python
from fastapi_grpc_bridge import start_grpc_server_standalone

# Start with configuration object
start_grpc_server_standalone(config=config)

# Start with dictionary
start_grpc_server_standalone(config_dict=config_dict)

# Start with environment variables
start_grpc_server_standalone(from_env=True)
```

### Convenience Functions

```python
from fastapi_grpc_bridge import (
    start_grpc_server_with_tls,
    start_grpc_server_with_mtls
)

# Start with TLS
start_grpc_server_with_tls(
    cert_file="server.crt",
    key_file="server.key",
    host="0.0.0.0",
    port=50051
)

# Start with mTLS
start_grpc_server_with_mtls(
    cert_file="server.crt",
    key_file="server.key",
    ca_cert_file="ca.crt",
    client_cert_required=True
)
```

## Certificate Management

### Certificate Validation

```python
from fastapi_grpc_bridge import validate_certificates

# Validate configuration
issues = validate_certificates(config)
if issues:
    for issue in issues:
        print(f"Certificate issue: {issue}")
```

### Generate Self-Signed Certificates

For development and testing:

```python
from fastapi_grpc_bridge import generate_self_signed_cert

# Generate certificates
cert_info = generate_self_signed_cert(
    cert_dir="./dev_certs",
    server_name="localhost",
    days_valid=365
)

print(f"Certificate: {cert_info['cert_file']}")
print(f"Private Key: {cert_info['key_file']}")
```

**Note**: Requires `cryptography` library: `pip install cryptography`

### Client Credentials

For secure client connections:

```python
from fastapi_grpc_bridge import create_client_credentials
import grpc

# Create client credentials
credentials = create_client_credentials(config)

# Use with gRPC client
channel = grpc.secure_channel('localhost:50051', credentials)
```

## Security Best Practices

### 1. Certificate Management

- **Production**: Use certificates from a trusted Certificate Authority (CA)
- **Development**: Use self-signed certificates or internal CA
- **File Permissions**: Restrict access to private key files (600 or 400)
- **Certificate Rotation**: Implement regular certificate renewal

### 2. mTLS Configuration

- **Client Certificates**: Always require client certificates in production
- **CA Validation**: Use proper CA certificates for client verification
- **Certificate Revocation**: Implement certificate revocation checking

### 3. Network Security

- **Firewall**: Restrict access to gRPC ports
- **Network Segmentation**: Use VPNs or private networks
- **Monitoring**: Log security events and certificate expiration

### 4. Configuration Security

- **Environment Variables**: Use secure storage for sensitive configuration
- **File Permissions**: Protect configuration files
- **Secrets Management**: Use dedicated secrets management systems

## Troubleshooting

### Common Issues

1. **Certificate not found**
   ```
   FileNotFoundError: Certificate file not found: server.crt
   ```
   - Check file paths and permissions
   - Ensure certificates exist and are readable

2. **Invalid certificate format**
   - Ensure certificates are in PEM format
   - Check for proper certificate chain

3. **Client connection refused**
   - Verify server is running with correct configuration
   - Check firewall settings
   - Validate client certificates for mTLS

4. **SSL handshake failed**
   - Check certificate validity and expiration
   - Verify CA certificate configuration
   - Ensure hostname matches certificate

### Debug Mode

Enable detailed logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Certificate Information

Check certificate details:

```bash
# View certificate information
openssl x509 -in server.crt -text -noout

# Verify certificate chain
openssl verify -CAfile ca.crt server.crt
```

## Migration Guide

### From Insecure to TLS

1. **Generate or obtain certificates**
2. **Update configuration**:
   ```python
   # Before
   add_grpc_support(app)
   
   # After
   config = create_secure_grpc_config(
       cert_file="server.crt",
       key_file="server.key"
   )
   add_grpc_support(app, config=config)
   ```

3. **Update clients** to use secure connections

### From TLS to mTLS

1. **Set up Certificate Authority**
2. **Generate client certificates**
3. **Update server configuration**:
   ```python
   # Add mTLS to existing TLS config
   config.mtls = MTLSConfig(
       enabled=True,
       ca_cert_file="ca.crt",
       client_cert_required=True
   )
   ```

4. **Update clients** with client certificates

## Examples

See `examples/secure_grpc_examples.py` for comprehensive examples of all security features.

## Support

For issues related to security features:

1. Check certificate validity and paths
2. Verify configuration settings
3. Review logs for detailed error messages
4. Consult the troubleshooting section above

## License

This security functionality is part of the fastapi-grpc-bridge package and follows the same licensing terms. 