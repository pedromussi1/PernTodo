<h1>Code Breakdown</h1>

<p>This page should cover all the steps that were taken to create the PernTodo web application. Code snippets will be shown and explained, along with screenshots of how the database looks on PostgreSQL and analyzing how the database connects to the website in the localhost.</p>

<h2>Server</h2>

<p>Player.cs:

Manages the player character's movement and interactions within the kitchen environment.
Handles player input, movement, and interaction events.
Communicates with various kitchen objects such as counters and plates.



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

