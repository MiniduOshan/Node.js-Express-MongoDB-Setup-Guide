# Node.js + Express + MongoDB Setup Guide

A complete step-by-step guide to set up a **Node.js** project with **Express** and **MongoDB**, including configuration, project structure, and best practices.

---

## Table of Contents

* [Prerequisites](#prerequisites)
* [Project Setup](#project-setup)
* [Installing Dependencies](#installing-dependencies)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [Creating Express Server](#creating-express-server)
* [Creating Routes and Controllers](#creating-routes-and-controllers)
* [Using Middleware](#using-middleware)
* [Testing the Server](#testing-the-server)
* [Project Structure](#project-structure)
* [Best Practices](#best-practices)
* [Troubleshooting](#troubleshooting)
* [References](#references)

---

## Prerequisites

* Node.js (v14 or above)
* npm or yarn
* MongoDB installed locally or a MongoDB Atlas account
* Basic knowledge of JavaScript and Node.js

---

## Project Setup

**Create a new Node.js project:**

```bash
mkdir my-node-app
cd my-node-app
npm init -y
```

---

## Installing Dependencies

Install core dependencies:

```bash
npm install express mongoose dotenv
```

Install development dependencies:

```bash
npm install -D nodemon
```

Update `package.json` scripts to use nodemon:

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js"
}
```

---

## Connecting to MongoDB

1. **Create a `.env` file** in the root directory:

```
MONGO_URI=mongodb://localhost:27017/mydatabase
PORT=5000
```

2. **Connect to MongoDB using Mongoose:**

```javascript
// db.js
const mongoose = require('mongoose');
const dotenv = require('dotenv');

dotenv.config();

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('MongoDB connected');
  } catch (err) {
    console.error(err);
    process.exit(1);
  }
};

module.exports = connectDB;
```

---

## Creating Express Server

```javascript
// index.js
const express = require('express');
const dotenv = require('dotenv');
const connectDB = require('./db');

dotenv.config();
connectDB();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware to parse JSON
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## Creating Routes and Controllers

**Example of a simple route and controller:**

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const { getUsers, createUser } = require('../controllers/userController');

router.get('/', getUsers);
router.post('/', createUser);

module.exports = router;
```

```javascript
// controllers/userController.js
const getUsers = (req, res) => {
  res.json({ message: 'Get all users' });
};

const createUser = (req, res) => {
  const user = req.body;
  res.json({ message: 'User created', user });
};

module.exports = { getUsers, createUser };
```

**Include routes in server:**

```javascript
const userRoutes = require('./routes/userRoutes');
app.use('/api/users', userRoutes);
```

---

## Using Middleware

* **Example of middleware:**

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});
```

* Use `express.json()` to parse incoming JSON requests.

---

## Testing the Server

Start the development server:

```bash
npm run dev
```

Use Postman or curl to test your endpoints, e.g., `GET http://localhost:5000/api/users`.

---

## Project Structure

```
my-node-app/
│
├── controllers/
│   └── userController.js
├── routes/
│   └── userRoutes.js
├── db.js
├── index.js
├── package.json
├── package-lock.json
└── .env
```

---

## Best Practices

* Keep routes, controllers, and models in separate folders.
* Use environment variables for sensitive information.
* Handle errors properly using middleware.
* Use nodemon for development for auto-restart.

---

## Troubleshooting

* **MongoDB connection failed:**

  * Make sure MongoDB service is running
  * Check `MONGO_URI` in `.env`
* **Server not starting:**

  * Ensure `PORT` is set
  * Check for syntax errors in `index.js`

---

## References

* [Node.js Documentation](https://nodejs.org/en/docs/)
* [Express Documentation](https://expressjs.com/)
* [MongoDB Documentation](https://docs.mongodb.com/)
* [Mongoose Documentation](https://mongoosejs.com/docs/)

---

**Happy Coding! 🚀**
