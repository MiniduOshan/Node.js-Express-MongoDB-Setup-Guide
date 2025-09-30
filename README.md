# Node.js + Express + MongoDB (Mongoose) Setup Guide

A complete step-by-step guide to set up a **Node.js** project with **Express** and **MongoDB (Mongoose)**, including configuration, project structure, best practices, **cookies**, **session handling**, and **JWT (installation & token creation)**.

---

## Table of Contents

* [Prerequisites](#prerequisites)
* [Project Setup](#project-setup)
* [Installing Dependencies](#installing-dependencies)
* [Environment Variables](#environment-variables)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [Creating Express Server](#creating-express-server)
* [Creating Models](#creating-models)
* [Creating Routes and Controllers](#creating-routes-and-controllers)
* [Using Middleware](#using-middleware)
* [Cookies](#cookies)
* [Session Handling](#session-handling)
* [JWT: Install & Create Tokens](#jwt-install--create-tokens)
* [Protecting Routes (Auth Middleware)](#protecting-routes-auth-middleware)
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

* **express** → Web framework for building APIs.
* **mongoose** → ODM (Object Data Modeling) library for MongoDB.
* **dotenv** → Loads environment variables from `.env`.

Install auth, cookies and sessions related deps:

```bash
npm install cookie-parser express-session connect-mongo jsonwebtoken
```

**What these do:**

* **cookie-parser** → Parses cookies from incoming requests.
* **express-session** → Creates server-side sessions.
* **connect-mongo** → Stores sessions in MongoDB (production-ready).
* **jsonwebtoken** → Create/verify JWT access/refresh tokens.

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

## Environment Variables

Create a `.env` file with required secrets and config.

```
MONGO_URI=mongodb://localhost:27017/mydatabase
PORT=5000
SESSION_SECRET=super-secret-session-key
JWT_SECRET=super-secret-jwt-key
NODE_ENV=development
COOKIE_DOMAIN=localhost
```

> **Never commit real secrets.** Use strong, unique values in production.

---

## Connecting to MongoDB

1. Create `db.js`:

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
const cookieParser = require('cookie-parser');
const session = require('express-session');
const MongoStore = require('connect-mongo');

dotenv.config();
connectDB();

const app = express();
const PORT = process.env.PORT || 5000;
const isProd = process.env.NODE_ENV === 'production';

// Parse JSON and cookies
app.use(express.json());
app.use(cookieParser());

// Session middleware (server-side sessions)
app.use(
  session({
    name: 'sid',
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({ mongoUrl: process.env.MONGO_URI }),
    cookie: {
      httpOnly: true,
      secure: isProd, // true if behind HTTPS in production
      sameSite: isProd ? 'lax' : 'lax',
      maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
      domain: process.env.COOKIE_DOMAIN || undefined,
    },
  })
);

app.get('/', (req, res) => {
  res.send('Hello World!');
});

// Routes
const userRoutes = require('./routes/userRoutes');
const authRoutes = require('./routes/authRoutes');
app.use('/api/users', userRoutes);
app.use('/api/auth', authRoutes);

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
  passwordHash: {
    type: String,
    required: false, // set true if you add password-based login
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
});

module.exports = mongoose.model('User', userSchema);
```

> For brevity, hashing is not shown. In real apps, hash with **bcrypt** and validate input.

---

## Creating Routes and Controllers

**Route:**

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const { getUsers, createUser } = require('../controllers/userController');
const { requireAuth } = require('../middleware/auth');

router.get('/', requireAuth, getUsers); // protected with JWT or session
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

Register Routes already shown in **Creating Express Server**.

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

## Cookies

Install & use **cookie-parser** (already installed above):

```javascript
// index.js
const cookieParser = require('cookie-parser');
app.use(cookieParser());
```

**Set a cookie:**

```javascript
// routes/authRoutes.js (example endpoint)
res.cookie('theme', 'dark', {
  httpOnly: true,
  sameSite: 'lax',
  secure: process.env.NODE_ENV === 'production',
  maxAge: 1000 * 60 * 60 * 24, // 1 day
});
```

**Read a cookie:**

```javascript
const theme = req.cookies.theme; // requires cookie-parser
```

> Prefer **HttpOnly** cookies for sensitive data (tokens, session IDs). Avoid storing secrets in non-HttpOnly cookies.

---

## Session Handling

Install

```bash
npm install express-session connect-mongo
```

Initialize (already shown in `index.js`):

```javascript
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({ mongoUrl: process.env.MONGO_URI }),
    cookie: { httpOnly: true, secure: isProd, sameSite: 'lax' },
  })
);
```

**Using the session:**

```javascript
// routes/authRoutes.js
router.post('/login-session', async (req, res) => {
  // ... authenticate user ...
  req.session.userId = 'someUserId';
  res.json({ message: 'Logged in with session' });
});

router.post('/logout-session', (req, res) => {
  req.session.destroy(() => {
    res.clearCookie('sid');
    res.json({ message: 'Logged out' });
  });
});
```

> Sessions are stateful (server stores data). JWT is stateless (server verifies token signature). You can use either or both depending on needs.

---

## JWT: Install & Create Tokens

Install JSON Web Tokens:

```bash
npm install jsonwebtoken
```

**Token utility:**

```javascript
// utils/jwt.js
const jwt = require('jsonwebtoken');

const signAccessToken = (payload, opts = {}) => {
  return jwt.sign(payload, process.env.JWT_SECRET, {
    expiresIn: '15m',
    ...opts,
  });
};

const verifyToken = (token) => jwt.verify(token, process.env.JWT_SECRET);

module.exports = { signAccessToken, verifyToken };
```

**Auth routes (issue JWT and set as HttpOnly cookie):**

```javascript
// routes/authRoutes.js
const express = require('express');
const router = express.Router();
const { signAccessToken } = require('../utils/jwt');

// Fake login for demo
router.post('/login', async (req, res) => {
  const { email } = req.body;
  // TODO: verify user + password (hash compare), omitted for brevity
  const userId = '123';

  const accessToken = signAccessToken({ sub: userId, email });

  // Put token in HttpOnly cookie (recommended for browsers)
  res.cookie('access_token', accessToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 1000 * 60 * 15, // 15 minutes
  });

  res.json({ message: 'Logged in', accessToken }); // also return in body if needed (e.g., mobile clients)
});

router.post('/logout', (req, res) => {
  res.clearCookie('access_token');
  res.json({ message: 'Logged out' });
});

module.exports = router;
```

> You can also implement **refresh tokens** with longer expiry, stored in HttpOnly cookies or DB, to re-issue short-lived access tokens.

---

## Protecting Routes (Auth Middleware)

Create a middleware to read JWT from the Authorization header **or** cookie and verify it.

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const requireAuth = (req, res, next) => {
  try {
    let token = null;

    // Prefer Authorization: Bearer <token>
    const auth = req.headers.authorization || '';
    if (auth.startsWith('Bearer ')) {
      token = auth.substring(7);
    }

    // Fallback to cookie
    if (!token && req.cookies && req.cookies.access_token) {
      token = req.cookies.access_token;
    }

    if (!token) return res.status(401).json({ error: 'Unauthorized' });

    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = { id: payload.sub, email: payload.email };
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
};

module.exports = { requireAuth };
```

**Use it on routes:**

```javascript
// routes/userRoutes.js
const { requireAuth } = require('../middleware/auth');
router.get('/', requireAuth, getUsers);
```

---

## Testing the Server

Run the development server:

```bash
npm run dev
```

Test endpoints with Postman or curl:

```bash
GET http://localhost:5000/api/users           // requires JWT
POST http://localhost:5000/api/auth/login     // get token
POST http://localhost:5000/api/auth/logout    // clear token cookie
```

Example login request body:

```json
{
  "email": "john@example.com",
  "password": "supersecret"
}
```

To call protected route with **Bearer token**:

```
Authorization: Bearer <accessToken>
```

---

## Project Structure

```
my-node-app/
│
├── controllers/
│   └── userController.js
├── middleware/
│   └── auth.js
├── models/
│   └── User.js
├── routes/
│   ├── userRoutes.js
│   └── authRoutes.js
├── utils/
│   └── jwt.js
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
* **HTTPS in production** to secure cookies & tokens.
* Short-lived access tokens (e.g., 15m) + refresh tokens (e.g., 7d) if needed.
* Validate request data and sanitize inputs.
* Hash passwords (bcrypt) and never store plaintext passwords.
* Rotate secrets when compromised, and handle token revocation (blacklist/rotation) if required.

---

## Troubleshooting

* **MongoDB connection failed:**

  * Ensure MongoDB service is running.
  * Verify `MONGO_URI` in `.env`.

* **Server not starting:**

  * Ensure `PORT` is set.
  * Check for syntax errors in `index.js`.

* **JWT always invalid/expired:**

  * Confirm the same `JWT_SECRET` is used for signing & verifying.
  * Check system time & token expiry.
  * Make sure you pass token via `Authorization` header or `access_token` cookie.

* **Session not persisting:**

  * Ensure cookie name matches and is not blocked by the browser.
  * For cross-site requests, consider `sameSite: 'none'` and `secure: true` (HTTPS required).

---

## References

* [Node.js Docs](https://nodejs.org/en/docs/)
* [Express Docs](https://expressjs.com/)
* [MongoDB Docs](https://docs.mongodb.com/)
* [Mongoose Docs](https://mongoosejs.com/docs/)
* [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)
* [express-session](https://github.com/expressjs/session)
* [connect-mongo](https://github.com/jdesboeufs/connect-mongo)
* [cookie-parser](https://github.com/expressjs/cookie-parser)

---

**Happy Coding! 🚀**
