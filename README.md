
## 1. Project Setup

```bash
# Initialize project
npm init -y

# Install dependencies
npm install express pg dotenv
```

## 2. Database Configuration (`db.js`)

```javascript
const { Pool } = require('pg');
require('dotenv').config();

const pool = new Pool({
  user: process.env.DB_USER || 'postgres',
  host: process.env.DB_HOST || 'localhost',
  database: process.env.DB_NAME || 'express_api',
  password: process.env.DB_PASSWORD || 'postgres',
  port: process.env.DB_PORT || 5432,
});

module.exports = {
  query: (text, params) => pool.query(text, params),
};
```

## 3. Main Application (`server.js`)

```javascript
require('dotenv').config();
const express = require('express');
const db = require('./db');
const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());

// Routes
app.get('/', (req, res) => {
  res.send('PostgreSQL CRUD API');
});

// GET all users
app.get('/users', async (req, res) => {
  try {
    const { rows } = await db.query('SELECT * FROM users');
    res.json(rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// GET single user
app.get('/users/:id', async (req, res) => {
  try {
    const { rows } = await db.query('SELECT * FROM users WHERE id = $1', [req.params.id]);
    if (rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// POST create user
app.post('/users', async (req, res) => {
  const { name, email, age } = req.body;
  
  // Validation
  if (!name || !email || !age) {
    return res.status(400).json({ error: 'Name, email, and age are required' });
  }

  try {
    const { rows } = await db.query(
      'INSERT INTO users (name, email, age) VALUES ($1, $2, $3) RETURNING *',
      [name, email, age]
    );
    res.status(201).json(rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// PUT update user
app.put('/users/:id', async (req, res) => {
  const { name, email, age } = req.body;
  
  // Validation
  if (!name || !email || !age) {
    return res.status(400).json({ error: 'Name, email, and age are required' });
  }

  try {
    const { rows } = await db.query(
      'UPDATE users SET name = $1, email = $2, age = $3 WHERE id = $4 RETURNING *',
      [name, email, age, req.params.id]
    );
    if (rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json(rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// DELETE user
app.delete('/users/:id', async (req, res) => {
  try {
    const { rowCount } = await db.query('DELETE FROM users WHERE id = $1', [req.params.id]);
    if (rowCount === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.status(204).end();
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
```

## 4. Environment Variables (`.env`)

```env
DB_USER=postgres
DB_HOST=localhost
DB_NAME=express_api
DB_PASSWORD=postgres
DB_PORT=5432
PORT=3000
```

## 5. Database Setup

Run these SQL commands to create your database and table:

```sql
CREATE DATABASE express_api;

\c express_api

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER NOT NULL
);
```

## 6. README.md

```markdown
# PostgreSQL CRUD API with Express.js

A simple RESTful API demonstrating CRUD operations with Express.js and PostgreSQL.

## Setup Instructions

1. Clone the repository
2. Install dependencies:
   ```bash
   npm install
   ```
3. Set up PostgreSQL database (see Database Setup section)
4. Create `.env` file with your database credentials
5. Start the server:
   ```bash
   node server.js
   ```

## API Endpoints

| Method | Endpoint    | Description          |
|--------|-------------|----------------------|
| GET    | /users      | Get all users        |
| GET    | /users/:id  | Get single user      |
| POST   | /users      | Create new user      |
| PUT    | /users/:id  | Update user          |
| DELETE | /users/:id  | Delete user          |

## Example Requests

### Create User (POST /users)
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}
```

### Update User (PUT /users/1)
```json
{
  "name": "John Smith",
  "email": "john.smith@example.com",
  "age": 31
}
```

## Testing with Postman

1. Import the Postman collection from `postman_collection.json`
2. Test all endpoints with sample data

## Database Schema

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER NOT NULL
);
```

## Dependencies

- express
- pg (PostgreSQL client)
- dotenv (for environment variables)
```

## 7. Postman Collection (Export as JSON)

Create a Postman collection with these requests:
- GET /users
- GET /users/:id
- POST /users
- PUT /users/:id
- DELETE /users/:id
