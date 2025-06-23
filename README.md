# FastAPI gRPC Bridge

A Python package that seamlessly adds gRPC support to FastAPI applications using decorators. It automatically generates Protocol Buffer definitions from Pydantic models and creates gRPC services that mirror your FastAPI routes.  
[![PyPI Version](https://img.shields.io/pypi/v/fastapi-grpc-bridge)](https://pypi.org/project/fastapi-grpc-bridge/)
[![Python Versions](https://img.shields.io/pypi/pyversions/fastapi-grpc-bridge)](https://pypi.org/project/fastapi-grpc-bridge/)
[![License: MIT](https://img.shields.io/pypi/l/fastapi-grpc-bridge)](https://github.com/abhishek17569/fastapi-grpc-bridge/blob/main/LICENSE)


## Features

- 🚀 **Easy Integration**: Add gRPC support to existing FastAPI apps with a simple decorator
- 🔄 **Auto Proto Generation**: Automatically generates `.proto` files from Pydantic models
- 📡 **Dual Protocol Support**: Serve both HTTP and gRPC from the same codebase
- 🔐 **Enterprise Security**: Full TLS and mTLS support with certificate management
- ⚡ **Auto-Start gRPC**: Automatically starts gRPC server when FastAPI starts (configurable)
- 🎯 **Type Safety**: Full type safety with Pydantic models
- 🔧 **FastAPI Router Support**: Works with FastAPI routers and sub-applications
- 🌐 **Async Support**: Full support for async/await functions
- 🛡️ **Flexible Configuration**: Support for environment variables, dictionaries, and objects

## Installation [![PyPI Version](https://img.shields.io/pypi/v/fastapi-grpc-bridge)](https://pypi.org/project/fastapi-grpc-bridge/)

```bash
pip install fastapi-grpc-bridge
```

## Quick Start

### 1. Basic Usage with Auto-Start (Recommended)

```python
from fastapi import FastAPI
from fastapi_grpc_bridge import grpc_route, add_grpc_support
from pydantic import BaseModel

app = FastAPI()

class HelloResponse(BaseModel):
    message: str

@app.get("/hello")
@grpc_route
async def say_hello(name: str) -> HelloResponse:
    return HelloResponse(message=f"Hello, {name}!")

# Add gRPC support - this will auto-start the gRPC server when FastAPI starts
add_grpc_support(app)

if __name__ == "__main__":
    import uvicorn
    # Both HTTP (port 8000) and gRPC (port 50051) will start automatically
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 2. Secure gRPC with TLS

```python
from fastapi import FastAPI
from fastapi_grpc_bridge import grpc_route, add_grpc_support, create_secure_grpc_config
from pydantic import BaseModel

app = FastAPI()

class HelloResponse(BaseModel):
    message: str

@app.get("/hello")
@grpc_route
async def say_hello(name: str) -> HelloResponse:
    return HelloResponse(message=f"Hello, {name}!")

# Create secure configuration
config = create_secure_grpc_config(
    cert_file="path/to/server.crt",
    key_file="path/to/server.key"
)

# Add secure gRPC support
add_grpc_support(app, config=config)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 3. Mutual TLS (mTLS) Configuration

```python
from fastapi_grpc_bridge import create_secure_grpc_config

# mTLS configuration
config = create_secure_grpc_config(
    cert_file="path/to/server.crt",
    key_file="path/to/server.key",
    ca_cert_file="path/to/ca.crt",
    enable_mtls=True,
    client_cert_required=True
)

add_grpc_support(app, config=config)
```

## Security Features

### 🔐 TLS/mTLS Support

The package provides enterprise-grade security with:

- **TLS Encryption**: Secure server-client communication
- **Mutual TLS (mTLS)**: Bidirectional certificate authentication
- **Certificate Management**: Built-in certificate validation and utilities
- **Flexible Configuration**: Multiple ways to configure security settings

### Security Configuration Methods

**1. Using Configuration Objects:**
```python
from fastapi_grpc_bridge import GRPCSecurityConfig, TLSConfig, MTLSConfig

config = GRPCSecurityConfig(
    insecure=False,
    tls=TLSConfig(
        enabled=True,
        cert_file="server.crt",
        key_file="server.key",
        ca_cert_file="ca.crt"
    ),
    mtls=MTLSConfig(
        enabled=True,
        client_cert_required=True,
        ca_cert_file="ca.crt"
    )
)
```

**2. Using Environment Variables:**
```bash
export GRPC_INSECURE=false
export GRPC_TLS_ENABLED=true
export GRPC_TLS_CERT_FILE=server.crt
export GRPC_TLS_KEY_FILE=server.key
export GRPC_TLS_CA_CERT_FILE=ca.crt
export GRPC_MTLS_ENABLED=true
export GRPC_MTLS_CLIENT_CERT_REQUIRED=true
```

```python
from fastapi_grpc_bridge import GRPCSecurityConfig

config = GRPCSecurityConfig.from_env()
add_grpc_support(app, config=config)
```

**3. Using Dictionaries:**
```python
config_dict = {
    "insecure": False,
    "tls": {
        "enabled": True,
        "cert_file": "server.crt",
        "key_file": "server.key",
        "ca_cert_file": "ca.crt"
    },
    "mtls": {
        "enabled": True,
        "client_cert_required": True,
        "ca_cert_file": "ca.crt"
    }
}

config = GRPCSecurityConfig.from_dict(config_dict)
```

### Client Credentials

Create secure client connections:

```python
from fastapi_grpc_bridge import create_client_credentials
import grpc

# Create client credentials
credentials = create_client_credentials(config)

# Use with gRPC client
channel = grpc.secure_channel('localhost:50051', credentials)
```

## Working with FastAPI Routers

```python
from fastapi import FastAPI, APIRouter
from fastapi_grpc_bridge import grpc_route, add_grpc_support
from pydantic import BaseModel

app = FastAPI()
api_router = APIRouter(prefix="/api")

class UserResponse(BaseModel):
    id: int
    name: str
    email: str

@api_router.get("/user")
@grpc_route
async def get_user(user_id: int) -> UserResponse:
    return UserResponse(
        id=user_id,
        name=f"User {user_id}",
        email=f"user{user_id}@example.com"
    )

app.include_router(api_router)
add_grpc_support(app)  # Auto-starts gRPC server

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Type Support

The package automatically maps Python types to Protocol Buffer types:

| Python Type | Protocol Buffer Type |
|-------------|---------------------|
| `str` | `string` |
| `int` | `int32` |
| `float` | `float` |
| `bool` | `bool` |
| `bytes` | `bytes` |
| `Optional[T]` | `T` (optional) |

## API Reference

### Core Functions

#### `@grpc_route`
Decorator to add gRPC support to FastAPI routes.

#### `add_grpc_support(app, config=None, auto_start_grpc=True)`
Add gRPC support to a FastAPI app or router.

**Parameters:**
- `app`: FastAPI app or APIRouter instance
- `config`: Security configuration (GRPCSecurityConfig, dict, or None for insecure)
- `auto_start_grpc`: Whether to automatically start gRPC server (default: True)

#### `create_secure_grpc_config(**kwargs)`
Convenience function to create secure gRPC configuration.

**Parameters:**
- `cert_file`: Server certificate file path
- `key_file`: Server private key file path  
- `ca_cert_file`: CA certificate file path (optional)
- `enable_mtls`: Enable mutual TLS (default: False)
- `client_cert_required`: Require client certificates for mTLS (default: True)
- `host`: Server host (default: "[::]")
- `port`: Server port (default: 50051)

### Security Classes

#### `GRPCSecurityConfig`
Main configuration class for gRPC security settings.

#### `TLSConfig`
Configuration for TLS settings.

#### `MTLSConfig`
Configuration for mutual TLS settings.

## Testing Your Services

### Test HTTP Service
```bash
curl "http://localhost:8000/hello?name=World"
curl "http://localhost:8000/api/user?user_id=123"
curl "http://localhost:8000/health"  # Shows gRPC routes status
```

### Test Secure gRPC Service
```python
import grpc
from fastapi_grpc_bridge import create_client_credentials

# Create secure channel
config = create_secure_grpc_config(ca_cert_file="ca.crt")
credentials = create_client_credentials(config)
channel = grpc.secure_channel('localhost:50051', credentials)

# Use the channel for gRPC calls
# ... (rest of gRPC client code)
```

### Test Insecure gRPC Service
```python
import grpc
import sys
sys.path.append('generated_protos')

import say_hello_pb2
import say_hello_pb2_grpc

def test_grpc_service():
    channel = grpc.insecure_channel('localhost:50051')
    stub = say_hello_pb2_grpc.Say_helloServiceStub(channel)
    
    request = say_hello_pb2.say_helloRequest(name="World")
    response = stub.Call(request)
    
    print(f"Response: {response.message}")
    channel.close()

if __name__ == "__main__":
    test_grpc_service()
```

## Generated Files

The package automatically generates:

- `.proto` files in the `generated_protos/` directory
- Python gRPC client and server stubs (`*_pb2.py` and `*_pb2_grpc.py`)

## Configuration

### Default Ports
- **HTTP**: 8000 (configurable via uvicorn)
- **gRPC**: 50051 (configurable via GRPCSecurityConfig)

### Security Modes
- **Insecure**: Default mode for development (no encryption)
- **TLS**: Server-side encryption with certificates
- **mTLS**: Mutual authentication with client and server certificates

### Auto-Start Behavior
- **Default**: gRPC server auto-starts when FastAPI starts
- **Disable**: Set `auto_start_grpc=False` in `add_grpc_support()`
- **Manual**: Use `start_grpc_server_standalone()` for manual control

## Documentation

- **Security Guide**: See `SECURITY_README.md` for comprehensive security documentation
- **Examples**: Check the `examples/` directory for working examples
- **API Reference**: Complete API documentation in the source code

## Requirements

- Python 3.7+
- FastAPI
- Pydantic  
- grpcio
- grpcio-tools

## License

This project is licensed under the MIT License - see the LICENSE file for details. 
