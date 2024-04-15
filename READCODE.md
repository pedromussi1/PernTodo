<h1>Code Breakdown</h1>

<p>This page should cover all the steps that were taken to create the PernTodo web application. Code snippets will be shown and explained, along with screenshots of how the database looks on PostgreSQL and analyzing how the database connects to the website in the localhost.</p>

<h2>Server</h2>

<h3>(Server) index.js:</h3>

<p>
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

<h3>db.js</h3>
<p>
Importing the Pool Class:
const Pool = require("pg").Pool;: We're importing the Pool class from the pg module. The pg module is a PostgreSQL client for Node.js that allows us to interact with the PostgreSQL database.

Creating a Pool Instance:
const pool = new Pool({...});: We're creating a new instance of the Pool class and passing an object with the database connection configurations:

user: The username used to authenticate with the PostgreSQL database.

password: The password used to authenticate with the PostgreSQL database.

host: The hostname or IP address where the PostgreSQL database is running (in this case, it's localhost).

port: The port number on which the PostgreSQL database is listening (default is 5432).

database: The name of the database to connect to (perntodo in this example).

Exporting the Pool Instance:
module.exports = pool;: We're exporting the pool instance so that it can be imported and used in other files (like in the Express application file) to perform database operations such as querying, inserting, updating, and deleting data.
</p>

<h2>Client</h2>

<h3>(Client) index.js</h3>

<p>

Creating a Root for the React Application:

const root = ReactDOM.createRoot(document.getElementById('root'));: We're creating a root for the React application using the ReactDOM.createRoot() method. This method takes the 'root' element from the DOM (usually a div with an id of 'root') and returns a root object that can be used to render the React application.
Rendering the App Component Inside the Root:

root.render(...): We're using the render method of the root object to render the App component inside the root of the React application.

<React.StrictMode> {...} </React.StrictMode>: We're using React.StrictMode to wrap the App component. StrictMode is a tool for highlighting potential problems in the application during development, such as deprecated APIs and side effects.

We're rendering the App component inside the StrictMode wrapper. The App component is the main component of the React application, which renders the child components InputTodo and ListTodos.
    
</p>

<h3>EditTodo.js</h3>

<p>

EditTodo Component:

The EditTodo component is responsible for rendering a modal that allows users to edit the description of a todo item. It consists of the following key parts:

Props:

todo: A prop containing the details of the todo item to be edited.
State Variables:

description: A state variable initialized with the description of the todo item passed as a prop to store the updated description.
Event Handlers:

updateDescription: An asynchronous function that handles the update of the todo item's description. It sends a PUT request to the backend API with the updated description to update the todo item in the database.
Rendered Elements:

A button to trigger the modal to edit the todo item.
A modal containing an input field for editing the todo item's description, an "Edit" button to update the description, and a "Close" button to close the modal.
</p>

<h3>InputTodo</h3>

<p>

InputTodo Component:

The InputTodo component is responsible for rendering a form that allows users to input a new todo item. It consists of the following key parts:

State Variables:

description: A state variable initialized with an empty string to store the description of the new todo item.
Event Handlers:

onSubmitForm: An asynchronous function that handles the form submission. It sends a POST request to the backend API with the new todo item's description to add it to the database.
Rendered Elements:

A heading indicating that this is a Pern Todo List.
A form containing an input field for the todo item description and a submit button.
</p>

<h3>ListTodos</h3>

<p>

The ListTodos component is responsible for rendering a list of todo items fetched from the backend. It consists of the following key parts:

State Variables:

todos: A state variable initialized with an empty array to store the list of todo items fetched from the backend.
Event Handlers:

deleteTodo: An asynchronous function that handles the deletion of a todo item. It sends a DELETE request to the backend API to delete the specified todo item and updates the todos state to remove the deleted todo item.
Effect Hook:

useEffect: A hook that runs the getTodos function when the component mounts to fetch all todo items from the backend.
Rendered Elements:

A table displaying the list of todo items fetched from the backend.
Each todo item is displayed as a table row containing its description, an EditTodo component to edit the todo item, and a Delete button to delete the todo item.
    
</p>


### <h3>(Server) index.js</h3>

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

### <h3>App.js</h3>

<details>
<summary>Click to expand code</summary>

```js

// Importing required modules and components
import React, { Fragment } from 'react'; // Importing React and Fragment from the 'react' library
import './App.css'; // Importing the CSS file for styling

// Importing custom components
import InputTodo from "./components/InputTodo"; // Importing the InputTodo component
import ListTodos from './components/ListTodos'; // Importing the ListTodos component

function App() {
  // Rendering the App component
  return (
    <Fragment> {/* Using Fragment to group multiple elements without adding an extra node to the DOM */}
      <div className="container"> {/* Container for styling purposes */}
        <InputTodo/> {/* Rendering the InputTodo component */}
        <ListTodos/> {/* Rendering the ListTodos component */}
      </div>
    </Fragment>
  );
}

export default App; // Exporting the App component to be used in other parts of the application


```
</details>

<hr>

### <h3>(Client) index.js</h3>

<details>
<summary>Click to expand code</summary>

```js

// Importing required modules and components
import React from 'react'; // Importing React from the 'react' library
import ReactDOM from 'react-dom/client'; // Importing ReactDOM from the 'react-dom/client' library
import './index.css'; // Importing the CSS file for styling
import App from './App'; // Importing the App component

// Creating a root for the React application
const root = ReactDOM.createRoot(document.getElementById('root')); // Creating a root for the React application with the 'root' element from the DOM

// Rendering the App component inside the root
root.render(
  <React.StrictMode> {/* Using React.StrictMode to detect potential problems in the application */}
    <App /> {/* Rendering the App component */}
  </React.StrictMode>
);

```
</details>

<hr>

### <h3>EditTodo.js</h3>

<details>
<summary>Click to expand code</summary>

```js

import React, { Fragment, useState } from "react";

// EditTodo component
const EditTodo = ({ todo }) => {

    // State variable to store the description of the todo item
    const [description, setDescription] = useState(todo.description);

    // Function to update the description of the todo item
    const updateDescription = async (e) => {
        e.preventDefault(); // Preventing the default form submission behavior
        try {
            const body = { description }; // Creating a body object with the updated description
            // Sending a PUT request to update the todo item with the new description
            const response = await fetch(`http://localhost:5000/todos/${todo.todo_id}`, {
                method: "PUT",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(body)
            });

            window.location = "/"; // Redirecting to the homepage after successful update
        } catch (err) {
            console.error(err.message); // Logging any errors to the console
        }
    }

    // Rendering the EditTodo component
    return (
        <Fragment>
            {/* Button to trigger the modal */}
            <button type="button" className="btn btn-warning" data-toggle="modal" data-target={`#id${todo.todo_id}`}>
                Edit
            </button>

            {/* Modal for editing the todo item */}
            <div className="modal" id={`id${todo.todo_id}`} onClick={() => setDescription(todo.description)}>
                <div className="modal-dialog">
                    <div className="modal-content">

                        {/* Modal header */}
                        <div className="modal-header">
                            <h4 className="modal-title">Edit Todo</h4>
                            <button type="button" className="close" data-dismiss="modal" onClick={() => setDescription(todo.description)}>&times;</button>
                        </div>

                        {/* Modal body */}
                        <div className="modal-body">
                            {/* Input field to edit the description */}
                            <input type='text' className="form-control" value={description} onChange={e => setDescription(e.target.value)} />
                        </div>

                        {/* Modal footer */}
                        <div className="modal-footer">
                            {/* Edit button */}
                            <button type="button" className="btn btn-warning" data-dismiss="modal" onClick={e => updateDescription(e)}>Edit</button>
                            {/* Close button */}
                            <button type="button" className="btn btn-danger" data-dismiss="modal" onClick={() => setDescription(todo.description)}>Close</button>
                        </div>

                    </div>
                </div>
            </div>
        </Fragment>
    );
};

export default EditTodo; // Exporting the EditTodo component

```
</details>

<hr>

### <h3>InputTodo.js</h3>

<details>
<summary>Click to expand code</summary>

```js

import React, { Fragment, useState } from 'react';

// InputTodo component
const InputTodo = () => {

    // State variable to store the description of the todo item
    const [description, setDescription] = useState("");

    // Function to handle form submission
    const onSubmitForm = async e => {
        e.preventDefault(); // Preventing the default form submission behavior
        try {
            const body = { description }; // Creating a body object with the description
            // Sending a POST request to add a new todo item
            const response = await fetch("http://localhost:5000/todos", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(body)
            });

            window.location = "/"; // Redirecting to the homepage after successful addition
        } catch (err) {
            console.error(err.message); // Logging any errors to the console
        }
    };

    // Rendering the InputTodo component
    return (
        <Fragment>
            {/* Heading */}
            <h1 className="text-center mt-5">Pern Todo List</h1>

            {/* Form for adding a new todo item */}
            <form className="d-flex mt-5" onSubmit={onSubmitForm}>
                {/* Input field to enter the description of the new todo item */}
                <input type="text" className="form-control" value={description} onChange={e => setDescription(e.target.value)} />

                {/* Submit button */}
                <button className="btn btn-success">Add</button>
            </form>
        </Fragment>
    );
};

export default InputTodo; // Exporting the InputTodo component

```
</details>

<hr>

### <h3>ListTodos.js</h3>

<details>
<summary>Click to expand code</summary>

```js

import React, { Fragment, useEffect, useState } from "react";

import EditTodo from "./EditTodo";

// ListTodos component
const ListTodos = () => {

    // State variable to store the list of todos
    const [todos, setTodos] = useState([]);

    // Function to delete a todo item
    const deleteTodo = async (id) => {
        try {
            // Sending a DELETE request to delete the todo item with the specified id
            const deleteTodo = await fetch(`http://localhost:5000/todos/${id}`, {
                method: "DELETE"
            });

            // Updating the todos state to remove the deleted todo item
            setTodos(todos.filter(todo => todo.todo_id !== id));
        } catch (err) {
            console.error(err.message); // Logging any errors to the console
        }
    }

    // Function to fetch all todo items from the backend
    const getTodos = async () => {
        try {
            // Sending a GET request to fetch all todo items
            const response = await fetch("http://localhost:5000/todos");
            const jsonData = await response.json(); // Parsing the JSON response

            // Updating the todos state with the fetched todo items
            setTodos(jsonData);
        } catch (err) {
            console.error(err.message); // Logging any errors to the console
        }
    };

    // useEffect hook to fetch todos when the component mounts
    useEffect(() => {
        getTodos(); // Calling the getTodos function
    }, []);

    // Logging the todos to the console
    console.log(todos);

    // Rendering the ListTodos component
    return ( 
        <Fragment>
            {/* Table to display the list of todos */}
            <table className="table mt-5 text-center">
                <thead>
                    <tr>
                        <th>Description</th>
                        <th>Edit</th>
                        <th>Delete</th>
                    </tr>
                </thead>
                <tbody>
                    {/* Mapping through the todos array and rendering each todo as a table row */}
                    {todos.map(todo => (
                        <tr key={todo.todo_id}>
                            <td>{todo.description}</td>
                            <td>
                                {/* EditTodo component to edit the todo item */}
                                <EditTodo todo={todo} />
                            </td>
                            <td>
                                {/* Delete button to delete the todo item */}
                                <button className="btn btn-danger" onClick={() => deleteTodo(todo.todo_id)}>Delete</button>
                            </td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </Fragment>
    );
};

export default ListTodos; // Exporting the ListTodos component

```
</details>

<hr>


