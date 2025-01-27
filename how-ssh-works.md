
# How SSH Works

SSH (Secure Shell) was created by Finnish computer scientist **Tatu Ylönen** in 1995. It provides a secure way to connect to remote servers, replacing insecure tools like `rlogin`, `rsh`, and `TELNET`. These older tools sent data in plaintext, making them vulnerable to eavesdropping via packet-sniffing tools. SSH solves this issue by encrypting the communication, ensuring data security.

---

## Establishing an SSH Connection

1. **TCP Connection**:
   - The client establishes a TCP connection to the server on port **22** (default).
   - Standard TCP handshake (SYN-ACK-SYN) occurs.

2. **Protocol and Software Exchange**:
   - The client and server exchange their supported SSH protocol versions and software identifiers (e.g., `SSH-2.0-OpenSSH` or `Putty`).

3. **Negotiating Algorithms**:
   - Both parties agree on:
     - **Key exchange algorithm** (e.g., Diffie-Hellman, Elliptic Curve DH).
     - **Encryption algorithm** (e.g., AES).
     - **Message Authentication Code (MAC)** algorithm (e.g., HMAC).
     - **Compression algorithm** (e.g., Zlib).

### Key Exchange Process

- The server generates an ephemeral key pair using a large prime number and generator.
- The server sends its **ephemeral public key** to the client.
- The client generates its own ephemeral key pair, calculates the shared secret, and sends its **public key** to the server.
- Both parties independently compute the **shared secret**, which is used to derive:
  - **Symmetric encryption key**.
  - **MAC key**.
  - **Initialization vector (IV)**.

From this point forward, communication is encrypted using the symmetric key.

### Server Authentication

- The server signs a hash with its **private key** and sends it to the client.
- The client:
  1. Decrypts the hash using the server’s **public key** (stored in `known_hosts`).
  2. Generates a hash using shared data and verifies it against the server’s hash.

If the hashes match, the server’s authenticity is confirmed.

### Client Authentication

- The client sends an authentication request, including its **public key**.
- The server checks if the key exists in its `authorized_keys` file.
- If the key is valid, the server:
  1. Sends a random plaintext challenge encrypted with the client’s **public key**.
  2. Verifies the decrypted response from the client using the **public key**.

Once authentication is successful, the client gains remote access to the server.

---

## AWS EC2

### SSH in AWS EC2
When connecting to an EC2 instance via SSH, the initial process (TCP connection, key exchange, server authentication) remains the same. However, the key management process involves additional steps:

1. **Key Pair Generation**:
   - When creating an EC2 instance, AWS generates a **key pair**:
     - **Public key**: Stored in the instance’s `authorized_keys` file.
     - **Private key**: Provided to the user as a `.pem` file.

2. **Connecting to the Instance**:
   - The client initiates a connection, and the server presents its **host key**. The client stores this key in the `known_hosts` file after verifying it.

3. **Client Authentication**:
   - The client sends an authentication request containing metadata, such as the SSH key algorithm.
   - The server uses this metadata to identify the correct public key from its `authorized_keys` file.
   - The server sends a plaintext challenge to the client.
   - The client signs the challenge with its private key (`.pem` file) and sends the response.
   - The server verifies the signature using the client’s public key.

Once authentication succeeds, the user gains secure access to the EC2 instance.

### Key Management in EC2
- The private key provided by AWS must be kept secure. Unauthorized access to this file can compromise the EC2 instance.
- The server’s host keys are stored in `/etc/ssh/` (e.g., `ssh_host_rsa_key`).
- The client’s public keys are stored in the `~/.ssh/authorized_keys` file on the server.

---

## Example: Connecting to an EC2 Instance

1. **Key Pair Setup**:
   - AWS generates the key pair when launching an EC2 instance.
   - The `.pem` file is downloaded and stored locally.

2. **First Connection**:
   - The client runs:
     ```bash
     ssh -i my-key.pem ec2-user@<instance-public-ip>
     ```
   - The server’s host key is verified and added to the `known_hosts` file.

3. **Authentication**:
   - The client signs the server’s challenge with its private key (`my-key.pem`).
   - The server verifies the signature and grants access.

---

By combining strong encryption, key-based authentication, and secure key management, SSH ensures secure access to EC2 instances and other remote servers.
