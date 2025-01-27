Understanding SSH: The Secure Gateway to Remote Servers

The History and Evolution of SSH

SSH (Secure Shell) was created by Finnish computer scientist Tatu Ylönen in 1995. It emerged as a solution to the glaring security flaws of earlier tools like rlogin, rsh, and the infamous TELNET. Back then, these tools transmitted data, including sensitive information like passwords, in plain text. This made them vulnerable to packet-sniffing attacks where attackers could easily intercept this information.

SSH revolutionized remote connectivity by introducing encryption, ensuring that all communication between a client and server is secure.

How SSH Works

Step 1: Establishing the Connection

The client initiates a TCP connection to the server on port 22 (default SSH port).

The usual TCP handshake (SYN, SYN-ACK, ACK) is completed.

Step 2: Protocol and Software Negotiation

The client and server exchange information about the supported SSH protocol versions and software, such as OpenSSH, PuTTY, or others.

Step 3: Agreeing on Algorithms

The client and server agree on the algorithms to be used for the session, including:

Key Exchange Algorithm: Examples include Diffie-Hellman or Elliptic Curve Diffie-Hellman (ECDH).

Encryption Algorithm: Examples include AES (Advanced Encryption Standard).

MAC (Message Authentication Code) Algorithm: Examples include HMAC (Hash-based Message Authentication Code).

Compression Algorithm: Examples include Zlib for data compression.

Step 4: Key Exchange (The Magic Behind Secure Communication)

The goal of the key exchange is to create a shared secret without directly transmitting it over the network. Here’s how it works:

Server Key Generation:

The server generates an ephemeral key pair using a large prime number and a generator.

The server sends its ephemeral public key to the client.

Client Key Generation:

The client generates its own ephemeral key pair.

The client calculates the shared secret and sends its ephemeral public key to the server.

Shared Secret Creation:

The server calculates the shared secret using its private key and the client’s public key.

The client and server now both possess the same shared secret without ever explicitly exchanging it.

Session Key Generation:

Using the shared secret, both parties derive the symmetric encryption key, MAC key, and an initialization vector (IV) for encrypted communication.

Step 5: Server Authentication

To ensure the server is genuine:

The server sends a hash signed with its private key.

The hash is generated using information known to both client and server.

The client uses the server’s public key (stored in the known_hosts file) to decrypt the hash.

The client generates its own hash using the same data and compares it to the server’s hash.

If the hashes match, the server’s authenticity is verified.

Step 6: Client Authentication

To verify the client’s identity:

The client sends an authentication request, including its public key.

The server checks if this public key exists in the authorized_keys file.

If the key is found:

The server generates a random challenge (plaintext) and encrypts it with the client’s public key.

The client receives the challenge, decrypts it using its private key (e.g., a .pem file), and sends the response back.

The server verifies the response by comparing it to the original challenge. If they match, the client is authenticated.

Step 7: Encrypted Communication

Once authenticated, all further communication is encrypted using the symmetric session key established during the key exchange process.

AWS EC2

When working with AWS EC2 instances, SSH remains the primary method for connecting securely to the server. The process has some specifics:

Key Pair Setup

During Instance Creation:

AWS generates a key pair for the instance:

The public key is stored on the server in the ~/.ssh/authorized_keys file.

The private key is provided as a .pem file to the user for secure storage.

Server’s Host Key:

The server also generates its own key pair (e.g., ssh_host_rsa_key) and stores the private key in /etc/ssh/. The corresponding public key is used during server authentication.

SSH Connection Steps for EC2

Establishing TCP and SSH Handshake:

The client initiates a TCP connection on port 22, followed by the usual SSH handshake.

The server sends its host key (public key) during the handshake.

If connecting for the first time, the client prompts the user to trust the server. On acceptance, the server’s public key is added to the client’s known_hosts file.

Server Authentication:

The server signs a hash using its private host key (/etc/ssh/ssh_host_rsa_key).

The client uses the server’s public key (received during the initial handshake) to verify the signature and authenticate the server.

Client Authentication:

The client sends an authentication request with metadata such as the preferred SSH key algorithm (e.g., RSA or ECDSA).

The server uses this metadata to select the appropriate public key from the authorized_keys file.

The server generates a random plaintext challenge and encrypts it with the client’s public key.

The client decrypts the challenge using the private key from the .pem file and sends the signed response back to the server.

The server verifies the response using the client’s public key. If successful, the client is authenticated.

Secure Communication:

After both server and client authentication, the session key (established during key exchange) is used to encrypt all further communication.

By following this process, AWS ensures that your EC2 instance connections are secure and only accessible to authorized users.


