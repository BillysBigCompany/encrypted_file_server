# Billy's Encrypted File Server

## Architecture

Billy's Encrypted File Server (BEFS) is a modern, high-performance encrypted file storage system built in Rust with three main components:

### Core Components
- **`file_server`** (this repository) - The main storage engine handling file operations, chunk management, and user storage quotas
- **`auth`** - Authentication service using [Biscuit tokens](https://www.biscuitsec.org/) for cryptographic authorization with fine-grained permissions
- **`cli`** - Client-side application handling file encryption, compression, and chunk reconstruction

### Communication Protocols
- **WebTransport** (primary) - Modern, multiplexed protocol for efficient file operations
- **HTTP API** - Compatibility layer for legacy clients and admin operations
- **Internal TCP API** - Encrypted administrative interface for service management

### Storage Architecture
The system uses a chunk-based approach where files are split into chunks for efficient storage and transfer:
- **Dual Storage Backend**: Local filesystem (development) and S3-compatible storage (production)
- **PostgreSQL Metadata**: Encrypted file metadata and chunk information
- **User Isolation**: All data strictly segregated by authenticated user ID

### Security Features
- **End-to-End Encryption**: Files encrypted client-side before transmission
- **Cryptographic Authentication**: Biscuit tokens with embedded permissions
- **Automated TLS**: Let's Encrypt certificate management with auto-renewal
- **Storage Quotas**: Per-user storage limits with enforcement
- **Token Revocation**: Real-time token invalidation system

For detailed architectural information, see [ARCHITECTURE.md](ARCHITECTURE.md).
 
# Dependencies
I make heavy use of the [nix package manager](https://nixos.org/download). So if you want to work on the project, I recommend installing that. Alternatively, if you just want to run the server, you can use Docker.

## Usage
Remember that there are two parts two the server, `auth` and `file_server`. While you can run both components individually, you probably want to be running both ;).

### Building the server
Enter into either the `auth/` directory, or the root directory, and just run `nix build`

You can also use Docker by running `docker buildx build . -t billys_file_server`

### Running the server
Before running the server, you need to create an empty `data.db` file in the root directory, as well as in `auth/`. 

Enter into either the `auth/` directory, or the root directory, and just run `nix run`

As for Docker users, if you already built the Docker image and tagged it with `billys_file_server`, you can run it using `docker run -it billys_file_server`

### Running the CLI
1. `nix develop` (to get all the dependencies necessary for the CLI)
2. `cd cli/`
3. `sqlx db create` (initializes the CLI's database)
4. `cargo run`
