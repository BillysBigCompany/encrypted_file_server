# Billy's Encrypted File Server (BEFS) Architecture

## Overview

Billy's Encrypted File Server (BEFS) is a high-performance, end-to-end encrypted file storage system built in Rust. The system provides secure storage and retrieval of files through a chunk-based architecture, supporting both local filesystem and cloud storage backends, with strong security guarantees and operational excellence.

## System Architecture

### High-Level Components

BEFS consists of three main components working together:

1. **File Server** (`src/`) - Core storage and retrieval engine (this repository)
2. **Authentication Service** (`auth/`) - Token-based authentication and authorization  
3. **CLI Client** (`cli/`) - Client-side encryption, compression, and file management

### Communication Protocols

- **WebTransport** (Port 9999) - Primary protocol for modern, multiplexed connections
- **HTTP API** (Port 9998) - Compatibility layer for legacy clients
- **Internal TCP API** (Port 9990) - Administrative operations and inter-service communication

## Core Architecture Components

### 1. Storage Layer

#### Chunk Storage (`chunk_db/`)
Files are broken into chunks and stored using an abstract storage interface:

- **`ChunkDB` Trait**: Provides storage backend abstraction
- **`FSChunkDB`**: Local filesystem storage for development/testing
- **`S3ChunkDB`**: S3-compatible cloud storage for production (Tigris on Fly.io)
- **Path Organization**: `/{user_id}/{chunk_id}` for user isolation
- **Garbage Collection**: Automated cleanup of orphaned chunks

#### Metadata Storage (`meta_db.rs`)
PostgreSQL database with connection pooling for metadata management:

**Core Tables:**
- `chunks` - Chunk metadata with encrypted/legacy support
- `file_metadata` - Client-encrypted file metadata
- `storage_caps` - User storage quotas and suspension flags
- `tls_certs` - Automated certificate management
- `queued_actions` - Deferred administrative operations

### 2. Security Layer

#### Authentication & Authorization (`auth.rs`)
- **Biscuit Tokens**: Cryptographic tokens with fine-grained permissions
- **Permission Types**: Read, Write, Delete, Query, Usage, Payment, Settings
- **User Isolation**: All operations scoped to authenticated user ID
- **Suspension System**: Granular per-operation user suspension

#### Token Management (`tokens.rs`)
- **Revocation Service Integration**: Real-time token validity checking
- **In-Memory Cache**: Fast revocation lookup with concurrent access
- **External Synchronization**: Configurable central authority

#### TLS Certificate Management (`tls.rs`)
- **ACME Integration**: Automated Let's Encrypt certificate provisioning
- **Database Storage**: Certificates with expiration tracking
- **Auto-renewal**: 60-day rotation cycle
- **HTTP-01 Challenge**: Automated domain validation

### 3. Administrative Layer

#### Action Queue System (`action.rs`)
Deferred execution system for administrative operations:

- **Action Types**: File deletion, user suspensions
- **Lifecycle**: Pending → Executing → Executed/Deleted
- **Background Processing**: Periodic execution with jitter
- **Status Tracking**: Full audit trail of administrative actions

#### Internal API (`internal.rs`)
Encrypted administrative communication channel:

- **XChaCha20Poly1305 Encryption**: All messages encrypted
- **Operations**: User management, storage quotas, suspension controls
- **Service Integration**: Designed for external admin system integration

### 4. Application Layer (`main.rs`)

The main application orchestrates all components:

- **Multi-Service Initialization**: Concurrent service startup
- **OpenTelemetry Integration**: Distributed tracing with Honeycomb
- **Background Services**: Token revocation, garbage collection, action processing
- **Graceful Shutdown**: Clean resource cleanup

## Data Flow

### File Upload Process
1. Client encrypts and compresses file chunks
2. WebTransport connection established with Biscuit token authentication
3. Chunks stored via `ChunkDB` implementation (FS or S3)
4. Encrypted metadata stored in PostgreSQL
5. Storage quota validation and enforcement

### File Download Process
1. Client authenticates with Biscuit token
2. Metadata retrieved and decrypted client-side
3. Chunks retrieved from storage backend
4. Client reconstructs, decompresses, and decrypts file

### Administrative Operations
1. External admin system sends encrypted request to Internal API
2. Action queued in database for deferred execution
3. Background processor executes action with proper error handling
4. Status updated for audit trail

## Security Model

### End-to-End Encryption
- **Client-Side Encryption**: All file content encrypted before transmission
- **Metadata Encryption**: File metadata encrypted with user keys
- **Zero-Knowledge**: Server cannot decrypt user data

### Access Control
- **Cryptographic Authentication**: Biscuit tokens with embedded permissions
- **User Isolation**: Strict per-user data segregation
- **Fine-Grained Permissions**: Operation-level access control
- **Token Revocation**: Real-time token invalidation capability

### Operational Security
- **TLS Everywhere**: All communications encrypted in transit
- **Certificate Automation**: Automated certificate lifecycle management
- **Audit Logging**: Comprehensive operation logging
- **Suspension Controls**: Granular user access restriction

## Deployment Architecture

### Production (Fly.io)
- **Container Deployment**: Docker-based deployment on Fly.io
- **Cloud Storage**: Tigris S3-compatible storage for chunks
- **PostgreSQL**: Managed database service
- **TLS Termination**: Automated Let's Encrypt certificates

### Development
- **Local Storage**: Filesystem-based chunk storage
- **Local Database**: Local PostgreSQL instance
- **Self-Signed Certificates**: Generated localhost certificates

### Feature Flags
- **`prod` Feature**: Production-specific optimizations
- **`s3` Feature**: S3 storage backend compilation
- **Environment-Specific**: Different builds for different environments

## Scalability Considerations

### Horizontal Scaling
- **Stateless Design**: Server instances can be scaled horizontally
- **Shared Database**: PostgreSQL handles concurrent access
- **Load Balancing**: WebTransport and HTTP traffic distribution

### Storage Scaling
- **Abstract Storage**: Easy migration between storage backends
- **S3 Compatibility**: Leverages cloud storage scalability
- **Chunk-Based**: Efficient handling of large files

### Performance Optimizations
- **Connection Pooling**: Database connection reuse
- **Async Processing**: Non-blocking I/O throughout
- **Background Tasks**: Deferred processing for expensive operations
- **Caching**: In-memory token revocation cache

## Dependencies

### Core Runtime
- **Tokio**: Async runtime and I/O framework
- **SQLx**: Type-safe async PostgreSQL driver
- **wtransport**: WebTransport protocol implementation

### Security
- **biscuit-auth**: Cryptographic authorization tokens
- **instant-acme**: Let's Encrypt ACME client
- **ring**: Cryptographic primitives

### Observability
- **OpenTelemetry**: Distributed tracing and metrics
- **tracing**: Structured logging framework

## Future Considerations

### Planned Enhancements
- Enhanced storage quota management
- Additional storage backend implementations
- Improved client-side caching strategies
- Advanced file versioning capabilities

### Monitoring & Observability
- Comprehensive health checks
- Performance metrics collection
- Storage usage analytics
- Security event monitoring

This architecture provides a robust foundation for secure, scalable file storage with strong encryption guarantees and operational excellence.