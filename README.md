# mean-stack-markdown

## Step 0: Adding dependencies
Before writing any code, you want to do the following to set up your fullstack app:
1. `yarn init`
2. add `start` and `dev` scripts to `package.json`
```
"scripts": {
  "start": "node app.js",
  "dev": "nodemon app.js"
}
```
3. yarn add the following dependencies:
* `express`
* `mongoose`
* `bcryptjs`
* `cors`
* `jsonwebtoken`
* `body-parser`
* `passport-jwt`
4. yarn add --dev `nodemon`

## Step 1: Setting Up App.js
In the root directory, touch `app.js`.
```js
const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');
const cors = require('cors');
const passport = require('passport');
const mongoose = require('mongoose');

const app = express();
const port = process.env.PORT || 3000;

mongoose.connect();

app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false });
app.use(express.static(path.join(__dirname, 'public')));

app.listen(port, () => {
  console.log('Server started on port', port);
}
```

We also want to create our routes for our users that we want to create.
Back in `app.js`:

```js
const userRoutes = require('./routes/user-routes');
app.use('/users', userRoutes);
```

## Step 2: Setup Mongoose and UserSchema
We want to be able to create our users on our front end but we first need to set up Mongoose and create our Schema.

Now in our root directory, mkdir `db` and touch `config.js`.
In our `config.js`:

```js
module.exports = {
  database: 'mongodb://localhost:27017/<name>',
  secret: 'r2n3m4j5k6l7l8n0b'
}
```
Back in our root directory, mkdir `models` and touch `user.js`.
In our `user.js`:

```js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const db = require('../models/config');

const UserSchema = mongoose.Schame({
username: {
  type:String,
  required: true,
},
firstname: {
  type:String
},
lastname: {
  type:String
},
email: {
  type:String,
  required: true,
}

const User = mongoose.model('User' UserSchema);
// export schema and methods for mongoose queries
module.exports = {
  User,
  getUserById: function(id, callback) {
    User.findById(id, callback);
  },
  getUserByUsername: function(username, callback) {
    const query = { username }
    User.findOne(query, callback);
  },
  addUser: function(newUser, callback) {
    bcrypt.genSalt(10, (err, salt) => {
      bcrypt.hash(newUser.password, salt, (err, hash) => {
        if (err) { throw err; }
        newUser.password = hash;
        newUser.save(callback);
      });
    });
  }
}
```
Lastly back in `app.js`, we want to import the `db/config` file we created earlier to finish our mongoose setup.
```js
const db = require('./db/config');

mongoose.connect(db.database);

mongoose.connection.on('connected', () => {
  console.log('Connect to database', db.database);
}

mongoose.connection.on('error', err => {
  console.log('Database error', err);
}
```
Our final app.js should look like this:
<details>
<summary>app.js</summary>
  
```js
const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');
const cors = require('cors');
const passport = require('passport');
const mongoose = require('mongoose');
const db = require('./db/config');

// initialize express app
const app = express();
const port = process.env.PORT || 3000;

// mongodb setup
mongoose.connect(db.database);

mongoose.connection.on('connected', () => {
  console.log('Connect to database', db.database);
}

mongoose.connection.on('error', err => {
  console.log('Database error', err);
}

// more middleware
app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false });
app.use(express.static(path.join(__dirname, 'public')));

// listening for server start
app.listen(port, () => {
  console.log('Server started on port', port);
}

// setting up specific routes
const userRoutes = require('./routes/user-routes');
app.use('/users', userRoutes);
```
</details>
