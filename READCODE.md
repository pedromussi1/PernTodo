<h1>Code Breakdown</h1>

<p>This page should cover all the steps that were taken to create the PernTodo web application. Code snippets will be shown and explained, along with screenshots of how the database looks on PostgreSQL and analyzing how the database connects to the website in the localhost.</p>

<h2>Server</h2>

<p>index.js:

CORS: The app.use(cors()) line enables CORS, allowing our Express server to accept requests from different origins or domains.
JSON Parsing: app.use(express.json()) configures Express to parse incoming JSON requests, making it easier to work with JSON data sent from clients.
Routes:

Create a Todo:
app.post("/todos", async (req, res) => {...}): This route handles POST requests to create a new todo item. It extracts the description from the request body, inserts the new todo into the database, and sends back the newly created todo as a JSON response.

Get All Todos:
app.get("/todos", async (req, res) => {...}): This route handles GET requests to fetch all todo items from the database and sends them back as a JSON response.

Get a Specific Todo:
app.get("/todos/:id", async (req, res) => {...}): This route handles GET requests to fetch a specific todo item based on its ID from the database and sends it back as a JSON response.

Update a Specific Todo:
app.put("/todos/:id", async (req, res) => {...}): This route handles PUT requests to update a specific todo item based on its ID in the database and sends a success message back as a JSON response.

Delete a Specific Todo:
app.delete("/todos/:id", async (req, res) => {...}): This route handles DELETE requests to delete a specific todo item based on its ID from the database and sends a success message back as a JSON response.

Server Start:
app.listen(5000, () => {...}): This line starts the Express server on port 5000, and a message is logged to the console indicating that the server has started successfully.

</p>



### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

// Importing required modules
const express = require("express");
const app = express();
const cors = require("cors");
const pool = require("./db"); // Connecting to the PostgreSQL database

// Middleware setup
app.use(cors()); // Enable Cross-Origin Resource Sharing
app.use(express.json()); // Parse JSON bodies sent by clients

// ROUTES //

// Create a new todo item
app.post("/todos", async (req, res) => {
    try {
        const { description } = req.body; // Destructuring description from request body
        // Insert new todo into the database and return the inserted todo
        const newTodo = await pool.query("INSERT INTO todo (description) VALUES($1) RETURNING * ", [description]);
        res.json(newTodo.rows[0]); // Send the newly created todo as JSON response
    } catch (err) {
        console.error(err.message); // Log any errors to the console
    }
});

// Get all todo items
app.get("/todos", async (req, res) => {
    try {
        // Fetch all todos from the database
        const allTodos = await pool.query("SELECT * FROM todo");
        res.json(allTodos.rows); // Send the todos as JSON response
    } catch (err) {
        console.error(err.message); // Log any errors to the console
    }
});

// Get a specific todo item by ID
app.get("/todos/:id", async (req, res) => {
    try {
        const { id } = req.params; // Extract todo ID from request parameters
        // Fetch todo with the given ID from the database
        const todo = await pool.query("SELECT * FROM todo WHERE todo_id = $1", [id]);
        res.json(todo.rows[0]); // Send the fetched todo as JSON response
    } catch (err) {
        console.error(err.message); // Log any errors to the console
    }
});

// Update a specific todo item by ID
app.put("/todos/:id", async (req, res) => {
    try {
        const { id } = req.params; // Extract todo ID from request parameters
        const { description } = req.body; // Destructuring description from request body
        // Update todo with the given ID in the database
        await pool.query("UPDATE todo SET description = $1 WHERE todo_id = $2", [description, id]);
        res.json("Todo was updated!"); // Send success message as JSON response
    } catch (err) {
        console.error(err.message); // Log any errors to the console
    }
});

// Delete a specific todo item by ID
app.delete("/todos/:id", async (req, res) => {
    try {
        const { id } = req.params; // Extract todo ID from request parameters
        // Delete todo with the given ID from the database
        await pool.query("DELETE FROM todo WHERE todo_id = $1", [id]);
        res.json("Todo was deleted!"); // Send success message as JSON response
    } catch (err) {
        console.error(err.message); // Log any errors to the console
    }
});

// Start the server on port 5000
app.listen(5000, () => {
    console.log("Server has started on port 5000");
});

```
</details>

<hr>

### <h3>db.js</h3>

<details>
<summary>Click to expand code</summary>

```js

// Importing the Pool class from the 'pg' module to create a connection pool
const Pool = require("pg").Pool;

// Creating a new Pool instance with database connection configurations
const pool = new Pool({
    user: "postgres",       // Database user
    password: "1234",       // Database password
    host: "localhost",      // Database host
    port: 5432,             // Database port
    database: "perntodo"    // Database name
});

// Exporting the pool instance to be used in other files
module.exports = pool;

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>

### <h3>index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

```
</details>

<hr>
