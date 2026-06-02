# JWT

Here's a breakdown of the JWT structure and how each component is formed, based on the video:

- **Header:** This is the first part of the JWT, located before the first period. It is Base64 encoded and specifies the algorithm used for encoding/decoding and the token type.
- **Payload (Data):** The payload, or data section, is situated between the two periods. It contains the actual data, such as the user ID (subject - `sub`), name, issue time (`IAT`), and potentially an expiration date (`exp`). This section is also Base64 encoded.
- **Signature:** The signature is created by taking the Base64 encoded header and payload, combining them with a period, and then hashing this combined string using the algorithm specified in the header and a secret key. This signature is essential for verifying that the JWT has not been tampered with.

In essence, the header and payload are Base64 encoded, and the signature is a hash of those two components combined with a secret key.
