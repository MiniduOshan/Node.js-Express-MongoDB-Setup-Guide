# Node.js + Express + MongoDB (Mongoose) Setup Guide

A complete step-by-step guide to set up a **Node.js** project with **Express** and **MongoDB (Mongoose)**, including configuration, project structure, and best practices.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Installing Dependencies](#installing-dependencies)
- [Connecting to MongoDB](#connecting-to-mongodb)
- [Creating Express Server](#creating-express-server)
- [Creating Models](#creating-models)
- [Creating Routes and Controllers](#creating-routes-and-controllers)
- [Using Middleware](#using-middleware)
- [Testing the Server](#testing-the-server)
- [Project Structure](#project-structure)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Prerequisites

- Node.js (v14 or above)
- npm or yarn
- MongoDB installed locally or a MongoDB Atlas account
- Basic knowledge of JavaScript and Node.js

---

## Project Setup

Create a new Node.js project:

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

**What these do:**
- **express** → Web framework for building APIs.
- **mongoose** → ODM (Object Data Modeling) library for MongoDB.
- **dotenv** → Loads environment variables from `.env`.

Install development dependency:

```bash
npm install -D nodemon
```

Update `package.json` scripts:

```json
"scripts": {
  "start": "node index.js",
  "dev": "nodemon index.js"
}
```

---

## Connecting to MongoDB

1. Create a `.env` file:

```
MONGO_URI=mongodb://localhost:27017/mydatabase
PORT=5000
```

2. Setup Mongoose connection:

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
    console.log('MongoDB connected with Mongoose');
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

## Creating Models

Example **User model** with Mongoose:

```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
});

module.exports = mongoose.model('User', userSchema);
```

---

## Creating Routes and Controllers

**Route:**

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const { getUsers, createUser } = require('../controllers/userController');

router.get('/', getUsers);
router.post('/', createUser);

module.exports = router;
```

**Controller:**

```javascript
// controllers/userController.js
const User = require('../models/User');

// Get all users
const getUsers = async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

// Create a new user
const createUser = async (req, res) => {
  try {
    const user = new User(req.body);
    await user.save();
    res.status(201).json(user);
  } catch (err) {
    res.status(400).json({ error: err.message });
  }
};

module.exports = { getUsers, createUser };
```

**Register Routes in Server:**

```javascript
// index.js
const userRoutes = require('./routes/userRoutes');
app.use('/api/users', userRoutes);
```

---

## Using Middleware

Example custom middleware:

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});
```

Use `express.json()` to parse incoming JSON requests.

---

## Testing the Server

Run the development server:

```bash
npm run dev
```

Test endpoints with Postman or curl:

```bash
GET http://localhost:5000/api/users
POST http://localhost:5000/api/users
```

Example POST request body:

```json
{
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

## Project Structure

```
my-node-app/
│
├── controllers/
│   └── userController.js
├── models/
│   └── User.js
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

- Keep routes, controllers, and models in separate folders.
- Use environment variables for sensitive information.
- Handle errors using middleware.
- Use nodemon for automatic restarts in development.
- Validate request data before saving to MongoDB.

---

## Troubleshooting

- **MongoDB connection failed:**
  - Ensure MongoDB service is running.
  - Verify `MONGO_URI` in `.env`.

- **Server not starting:**
  - Ensure `PORT` is set.
  - Check for syntax errors in `index.js`.

---

## References

- [Node.js Docs](https://nodejs.org/en/docs/)
- [Express Docs](https://expressjs.com/)
- [MongoDB Docs](https://docs.mongodb.com/)
- [Mongoose Docs](https://mongoosejs.com/docs/)

---

**Happy Coding! 🚀**
