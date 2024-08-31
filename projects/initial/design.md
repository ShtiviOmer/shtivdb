# Requirements
In the initial phase the database should be operational database.
The intention for this database is rational (at least at the start).
It should be fast.


## Architecture
The database will be broken down to the following components:
* Connection: Will handle the connection itself, authentication, etc'
* SQL parsing: Will parse the SQL into a well defined instructions.
* Transaction management: support for ACID
* Caching: Data caching
* Storage: Handle data storing

### Connection
First phase the database will use TCP/IP.
The database will bind a specific IP/Port (can be set using configuration)

Upon receiving a connection request, the server will spawn a new asynchronous task (using Tokio) to handle the entire lifecycle of that connection. This ensures that each connection is managed independently, allowing the server to handle multiple connections concurrently without blocking.

<b>Connection lifecycle</b>

Connection Acceptance: When a connection is accepted, the server acknowledges the connection request and initiates the task.

Security (Optional): If TLS/SSL is to be used (default), the server should immediately perform a handshake to establish a secure communication channel before proceeding with any data exchange.
If TLS/SSL isn't enabled a warning will be logged.

Authentication: After the connection is established, the server prompts the client for authentication credentials (e.g., username and password). 
Default support should be with certificate.
The credentials are verified against the database's user management system. If authentication fails, the connection is terminated with an appropriate error message.

Session Initialization: If authentication is successful, a session is created for the client. This session will store relevant data like the user’s ID, roles/permissions, and any session-specific settings (e.g., timeouts, locale).

Command Loop: The server enters a command processing loop where it continuously reads commands from the client, processes them, and sends back the results. This loop continues until the client disconnects or the session is terminated.

### Session management
Session Creation: Upon successful authentication, a session object is created, containing the user's context, active transactions, and other metadata. This session persists for the duration of the connection.

Session State: The session may maintain various states such as "Idle," "Active," or "In Transaction," depending on the client's activities. Proper state management ensures correct handling of multi-step operations like transactions.

Session Timeout: Implement a mechanism to automatically close sessions that have been idle for too long, thereby freeing up resources.

Session Termination: When the client disconnects or if an error occurs, the session is properly terminated, releasing all resources associated with it.


### SQL parsing
Converts the SQL string into an AST an later on into instructions for the transaction manager

#### AST
Leverage [sqlparser-rs](https://github.com/sqlparser-rs/sqlparser-rs) to convert to AST.

#### Query validator
The query validator ensures that the parsed SQL query is syntactically and semantically correct. It checks for valid table names, column names, data types, and other constraints defined in the database schema.

Schema Validation: Ensure that the tables and columns referenced in the SQL query exist in the database schema.
Type Checking: Validate that the operations in the query make sense given the data types of the columns (e.g., you can't perform arithmetic on a string).
Constraint Checking: Ensure that the query adheres to constraints such as primary keys, foreign keys, and unique constraints.

#### Execution plan generator
Role:

The execution plan generator converts the validated AST into an execution plan, a step-by-step instruction set that the database's execution engine will follow to perform the query.

Implementation:

Logical Plan Generation: Create a logical plan representing the high-level operations (e.g., filter, join, sort) that need to be performed to execute the query.
Optimization: Optimize the logical plan by applying query optimization techniques (e.g., predicate pushdown, join reordering).
Physical Plan Generation: Convert the logical plan into a physical plan that maps directly to the operations the execution engine will perform. This step may involve choosing specific algorithms for joins, sorting, and other operations based on cost estimation.
Process:

Logical Plan Creation: Based on the AST, generate a logical plan that represents the abstract operations needed to execute the query.

Optimization: Apply query optimization techniques to the logical plan to improve performance.
Physical Plan Creation: Convert the optimized logical plan into a physical plan, selecting the most efficient execution strategies.

#### Integration with Other Components

Connection Phase: After the connection phase, the parsed SQL queries are sent to the SQL parsing phase for processing. The connection phase will receive the results of the parsing (success or error) and continue the command loop accordingly.

Transaction Management: The execution plan generated in this phase will be handed over to the transaction management component, which will ensure that the operations are executed within the context of a transaction, adhering to ACID properties.
Storage: The storage component will be responsible for fetching or updating the data as dictated by the execution plan.

### Storage layer
The storage should 


## Ideas
Support for WASI/WASM for extendability.
Rest-api for admins (Need to think about it, might introduce more complexity which is not needed)
Support for NoSql, KV, clustering, sharding, etc'
QUIC for faster database.
Support to store data not on file-system (S3 etc')
access control (RBAC)

