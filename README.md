# Node.js + Express + MongoDB (Mongoose) with Auth, Cookies, Sessions, Bcrypt & JWT

A complete step-by-step guide to set up a **Node.js** project with **Express** and **MongoDB (Mongoose)**, including CRUD APIs, **sessions & cookies**, **bcrypt password hashing**, and **JWT authentication**.

---

## Table of Contents

* [Prerequisites](#prerequisites)
* [Project Setup](#project-setup)
* [Installing Dependencies](#installing-dependencies)
* [Environment Variables](#environment-variables)
* [Connecting to MongoDB](#connecting-to-mongodb)
* [Creating Express Server](#creating-express-server)
* [Cookies & Sessions](#cookies--sessions)
* [Authentication (Bcrypt + JWT)](#authentication-bcrypt--jwt)

  * [User Model](#user-model)
  * [Auth Controller](#auth-controller)
  * [Auth Routes](#auth-routes)
  * [JWT Middleware](#jwt-middleware)
* [CRUD API](#crud-api)
* [Testing the API](#testing-the-api)
* [Project Structure](#project-structure)
* [Best Practices](#best-practices)
* [Troubleshooting](#troubleshooting)
* [References](#references)

---

## Prerequisites

* Node.js (v14 or higher)
* npm or yarn
* MongoDB installed locally or a MongoDB Atlas cluster
* Basic knowledge of JavaScript & APIs

---

## Project Setup

```bash
mkdir my-full-app
cd my-full-app
npm init -y
```

---

## Installing Dependencies

```bash
npm install express mongoose dotenv cookie-parser express-session connect-mongo bcrypt jsonwebtoken
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

Create `.env`:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/mydatabase
SESSION_SECRET=mySuperSessionSecret
JWT_ACCESS_SECRET=myStrongAccessSecret
JWT_REFRESH_SECRET=myStrongRefreshSecret
COOKIE_SECURE=false
```

---

## Connecting to MongoDB

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
    console.log('✅ MongoDB connected');
  } catch (err) {
    console.error('❌ MongoDB connection error:', err.message);
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

// Middleware
app.use(express.json());
app.use(cookieParser());

// Session setup
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    store: MongoStore.create({ mongoUrl: process.env.MONGO_URI }),
    cookie: {
      httpOnly: true,
      secure: process.env.COOKIE_SECURE === 'true',
      maxAge: 1000 * 60 * 60, // 1 hour
    },
  })
);

// Routes
app.get('/', (req, res) => res.send('Server running ✅'));
app.use('/api/users', require('./routes/userRoutes'));
app.use('/api/auth', require('./routes/authRoutes'));

app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));
```

---

## Cookies & Sessions

Example session usage:

```javascript
app.get('/session-demo', (req, res) => {
  req.session.views = (req.session.views || 0) + 1;
  res.json({ views: req.session.views });
});
```

---

## Authentication (Bcrypt + JWT)

### User Model

```javascript
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  password: { type: String, required: true, minlength: 6, select: false },
});

// Hash before save
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Compare passwords
userSchema.methods.comparePassword = function (candidate) {
  return bcrypt.compare(candidate, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

---

### Auth Controller

```javascript
// controllers/authController.js
const User = require('../models/User');
const jwt = require('jsonwebtoken');

const signToken = (id) =>
  jwt.sign({ id }, process.env.JWT_ACCESS_SECRET, { expiresIn: '15m' });

const register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    const exists = await User.findOne({ email });
    if (exists) return res.status(400).json({ error: 'Email exists' });

    const user = await User.create({ name, email, password });
    res.status(201).json({ message: 'Registered', user: { id: user._id, email } });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

const login = async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email }).select('+password');
    if (!user || !(await user.comparePassword(password)))
      return res.status(400).json({ error: 'Invalid credentials' });

    const token = signToken(user._id);
    res.cookie('jwt', token, { httpOnly: true, secure: process.env.COOKIE_SECURE === 'true' });
    res.json({ message: 'Logged in', token });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

const me = async (req, res) => {
  res.json({ user: req.user });
};

module.exports = { register, login, me };
```

---

### Auth Routes

```javascript
// routes/authRoutes.js
const express = require('express');
const { register, login, me } = require('../controllers/authController');
const { requireJWT } = require('../middleware/auth');
const router = express.Router();

router.post('/register', register);
router.post('/login', login);
router.get('/me', requireJWT, me);

module.exports = router;
```

---

### JWT Middleware

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const requireJWT = async (req, res, next) => {
  try {
    const token = req.cookies.jwt || req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'No token' });

    const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET);
    req.user = await User.findById(decoded.id).select('_id name email');
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};

module.exports = { requireJWT };
```

---

## CRUD API

### User Controller

```javascript
// controllers/userController.js
const User = require('../models/User');

const getUsers = async (req, res) => {
  const users = await User.find().select('name email');
  res.json(users);
};

const updateUser = async (req, res) => {
  const updated = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
  if (!updated) return res.status(404).json({ error: 'User not found' });
  res.json(updated);
};

const deleteUser = async (req, res) => {
  const deleted = await User.findByIdAndDelete(req.params.id);
  if (!deleted) return res.status(404).json({ error: 'User not found' });
  res.json({ message: 'User deleted' });
};

module.exports = { getUsers, updateUser, deleteUser };
```

### User Routes

```javascript
// routes/userRoutes.js
const express = require('express');
const { getUsers, updateUser, deleteUser } = require('../controllers/userController');
const { requireJWT } = require('../middleware/auth');
const router = express.Router();

router.get('/', requireJWT, getUsers);
router.put('/:id', requireJWT, updateUser);
router.delete('/:id', requireJWT, deleteUser);

module.exports = router;
```

---

## Testing the API

Start server:

```bash
npm run dev
```

Test with Postman or curl:

```bash
# Register
POST http://localhost:5000/api/auth/register
{
  "name": "Alice",
  "email": "alice@example.com",
  "password": "Password123"
}

# Login
POST http://localhost:5000/api/auth/login
{
  "email": "alice@example.com",
  "password": "Password123"
}

# Get profile
GET http://localhost:5000/api/auth/me

# Get all users (JWT protected)
GET http://localhost:5000/api/users
```

---

## Project Structure

```
my-full-app/
│
├── controllers/
│   ├── authController.js
│   └── userController.js
├── middleware/
│   └── auth.js
├── models/
│   └── User.js
├── routes/
│   ├── authRoutes.js
│   └── userRoutes.js
├── db.js
├── index.js
├── package.json
├── .env
```

---

## Best Practices

* Use **httpOnly cookies** for JWT to prevent XSS attacks.
* Store secrets in `.env`, never commit them.
* Hash all passwords with bcrypt.
* Use session for server-rendered apps; JWT for APIs/mobile.
* Add input validation with `joi` or `zod`.

---

## Troubleshooting

* **Mongo not connecting** → Check `MONGO_URI` and ensure MongoDB is running.
* **JWT errors** → Ensure token secret matches `JWT_ACCESS_SECRET`.
* **Sessions not persisting** → Verify `COOKIE_SECURE` and domain config.

---

## References

* [Express Docs](https://expressjs.com/)
* [Mongoose Docs](https://mongoosejs.com/docs/)
* [bcrypt](https://github.com/kelektiv/node.bcrypt.js)
* [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)
* [express-session](https://github.com/expressjs/session)

---

Happy Cooding🚀
