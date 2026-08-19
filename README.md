# Socket Programming

A Python TCP client/server project that provides authenticated, role-based access to an SQLite database. Clients connect to the server, sign in with employee credentials, and can view or modify database tables only when their role and branch permit it.

The project includes a newer, database-backed implementation and a smaller earlier socket example kept for reference.

## Features

- TCP socket server that supports multiple simultaneous clients using threads.
- Username/password authentication against the `Credentials` table.
- Password verification with `bcrypt` hashes.
- Role- and branch-based permissions for viewing, inserting, and updating data.
- JSON payloads with a four-byte length prefix for structured client/server data.
- Session logs containing the user, IP address, timestamps, commands, and errors.
- Generic table operations rather than operations limited to one database table.

## Architecture

```text
main.py (interactive client)
        |
        | TCP connection, port 12345
        v
server1.py (threaded server)
        |
        v
server_functions.py (authentication, permissions, database operations)
        |
        v
database1.db (SQLite database)
```

When a client connects, the server asks it to authenticate. On successful authentication, the server determines the user's role and branch from `Credentials`. Each requested database operation is checked against the matching row in the `Roles` table before it is executed.

## Project files

| File | Purpose |
| --- | --- |
| `server1.py` | Starts the current threaded server and accepts client connections. |
| `main.py` | Current command-line client. Handles login and sends database commands. |
| `server_functions.py` | Server-side authentication, authorization, JSON helpers, CRUD-style operations, and logging. |
| `server_variables.py` | Shared server state: connected clients, lock, counter, and the active database name. |
| `database1.db` | Main SQLite database used by the current server. |
| `1.py` | One-time migration helper that bcrypt-hashes plaintext passwords in `database1.db`. |
| `server.py`, `client1.py` | Earlier/basic socket example, without the full database permission system. |
| `database.py`, `database.db`, `test.py` | Earlier database experiments and a small reporting script. |
| `database1.sqbpro` | SQLite Browser project file. |

## Requirements

- Python 3.8 or later
- The `bcrypt` package

Install the external dependency:

```bash
python -m pip install bcrypt
```

`socket`, `sqlite3`, `json`, `threading`, and `datetime` are provided by Python's standard library.

## Running the current application

### 1. Choose a bind address

Both `server1.py` and `main.py` currently use the hard-coded address `100.86.253.5` and port `12345`.

- To run the client and server on the same machine, change that address in both files to `127.0.0.1` (or `localhost`).
- To run between two machines, use the server machine's reachable LAN/VPN IP address in both files and make sure port `12345` is allowed through the firewall.

### 2. Start the server

From this directory, run:

```bash
python server1.py
```

Expected output:

```text
Server listening...
```

### 3. Start a client

In another terminal, from the same directory, run:

```bash
python main.py
```

Enter a username and password that exist in the `Credentials` table of `database1.db`. Passwords must be stored as bcrypt hashes for login to succeed.

## Client commands

After login, enter one of the following exact commands.

| Command | What it does | Required permission |
| --- | --- | --- |
| `get_table` | Retrieves all rows from a table. | Viewer access to that table. |
| `insert_row` | Adds a row to a chosen table from JSON input. | Editor access to that table. |
| `update_cell` | Updates a column in rows matched by JSON conditions. | Editor access to that table. |
| `add_D_Cred` | Adds an employee record to `Credentials`; the password is bcrypt-hashed first. | Editor access to `Credentials`. |
| `Disconnect` | Ends the client session. | None. |

### Example: view a table

```text
Enter anything [Type 'Disconnect' to exit]: get_table
Enter table name: Customer
```

### Example: insert a row

```text
Enter anything [Type 'Disconnect' to exit]: insert_row
Enter table name: Shipper
Enter row data as JSON: {"companyName":"Example Logistics","phone":"555-0100"}
```

### Example: update a row

```text
Enter anything [Type 'Disconnect' to exit]: update_cell
Enter table name: Customer
Column to update: city
New value: Mumbai
Enter conditions as JSON: {"entityId": 1}
```

Use valid JSON: object keys and text values must be enclosed in double quotes.

## Database structure

The current database, `database1.db`, contains these main tables:

- `Credentials` — employee identity, branch, role, login name, bcrypt password hash, and active/inactive status.
- `Roles` — permissions by role and branch. `Viewer` allows reads; `Editor` allows inserts and updates. `Tables` holds the permitted table names or `All tables`.
- `Logs` — connection/session history, commands, and captured errors.
- `Category`, `Customer`, `Product`, `Shipper`, and `Supplier` — business data tables.

## Password migration

If `database1.db` still has plaintext passwords, run the migration helper once:

```bash
python 1.py
```

It skips passwords that already look like bcrypt hashes and hashes the rest. Back up the database before using it because this updates stored password values.

## Notes and limitations

- `database1.db` is the active database for the newer server. `database.db` is used only by the older helper/example scripts.
- This is a learning project, not a production-ready secure service. In particular, it has no TLS encryption, hard-codes the server address, and dynamically builds SQL identifiers from client-supplied table/column names.
- The current server records command and error information in `Logs` when a client disconnects.
- `server.py`/`client1.py` use a different, older protocol and should be run together only with each other—not with `server1.py`/`main.py`.

## Suggested next improvements

1. Move the host, port, and database path into a configuration file or environment variables.
2. Validate table and column identifiers against an allow-list before building SQL.
3. Add TLS and a clearer request/response protocol for production use.
4. Add automated tests for authentication and permission rules.
5. Add database initialization and seed scripts so a new user can run the project without relying on the existing `.db` files.

## License

See [LICENSE](LICENSE).
