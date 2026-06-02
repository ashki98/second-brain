# Wire Protocols - RESP

YT: https://www.youtube.com/watch?v=PtJl3jtmqgE

This summary outlines the fundamental concepts of wire protocols and the **Redis Serialization Protocol (RESP)**, based on the video "Wire Protocols, Why Are They Needed, and Redis' Wire Protocol - RESP" by Arpit Bhayani.

### I. Overview of Wire Protocols & RESP

A wire protocol defines how data is serialized and exchanged between a client and a server over a network so both sides can understand the communication.

- **Request-Response Model**: In Redis, both the request sent by the client and the response returned by the server are RESP-encoded.
- **Simple Purpose**: Redis uses RESP to support basic data types like integers, strings, and arrays with minimal complexity.

### II. General Formatting Rules

Every piece of data in RESP follows a consistent structure to ensure efficient parsing:

- **Special Character Start**: Each data type begins with a specific identifier character.
- **CRLF Termination**: Every data element and metadata line must end with `\r\n` (Carriage Return and Line Feed).

### III. RESP Data Types and Specifications

### 1. Simple Strings

Used for small, non-binary safe messages like "OK" or "PONG".

- **Identifier**: Starts with a `+`.
- **Format**: `+<string>\r\n`
- **Example**: `+PONG\r\n`.

### 2. Integers

Represents 64-bit signed integers.

- **Identifier**: Starts with a `:`.
- **Format**: `:<number>\r\n`
- **Example**: `:1729\r\n`.

### 3. Bulk Strings

Used for binary-safe strings, allowing them to contain any byte sequence, including newlines or null characters.

- **Identifier**: Starts with a `$`.
- **Binary Safety**: It is "binary safe" because it specifies the length (in bytes) upfront, so the parser knows exactly when the string ends even if it contains a `\r\n` character inside.
- **Format**: `$<length>\r\n<data>\r\n`
- **Example (PONG)**: `$4\r\nPONG\r\n`.
- **Special Cases**:
    - **Empty String**: `$0\r\n\r\n`.
    - **Null Value**: `$-1\r\n` (indicates a lack of data).

### 4. Arrays

Used to group multiple RESP data types. Most Redis commands are sent as an array of bulk strings.

- **Identifier**: Starts with a .
- **Format**: `<number_of_elements>\r\n<element_1><element_2>...`
- **Example (Mixed Types)**: An array containing the string "A", the integer 200, and the string "CAT".Plaintext
    
    # 
    
    - `3\r\n
    $1\r\n
    A\r\n
    :200\r\n
    $3\r\n
    CAT\r\n`
- **Special Cases**:
    - **Null Array**: `-1\r\n`.
    - **Empty Array**: `0\r\n`.

### 5. Errors

Used to transmit error messages from the server.

- **Identifier**: Starts with a .
- **Example**:  `Key not found\r\n`.

### IV. Design Goals: Why RESP instead of JSON?

Redis chose a custom protocol over standard formats like JSON for several critical performance reasons:

- **Human Readable**: Despite being optimized for machines, it remains easy for developers to debug manually.
- **Parsing Efficiency**: JSON parsing (handling quotes, colons, escaping) is CPU-intensive; RESP is designed to be parsed with extremely low overhead.
- **Prefix Length Benefits**: By stating the length or element count at the start (e.g., `$5\r\nhello`), the server can pre-allocate memory buffers and perform a single `read` call for exactly that many bytes, preventing unnecessary blocking.
- **Simplicity**: A simpler protocol leads to fewer implementation bugs and faster execution.

**Video Link**: [Wire Protocols, Why Are They Needed, and Redis' Wire Protocol - RESP | Redis Internals](http://www.youtube.com/watch?v=PtJl3jtmqgE)
