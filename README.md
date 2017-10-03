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
* `passport`
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
const config = require('../db/config');

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
},
password: {
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
const config = require('./db/config');

mongoose.connect(config.database);

mongoose.connection.on('connected', () => {
  console.log('Connect to database', config.database);
}

mongoose.connection.on('error', err => {
  console.log('Database error', err);
}
```
Our app.js should now look like this:
<details>
<summary>app.js</summary>
  
```js
const express = require('express');
const path = require('path');
const bodyParser = require('body-parser');
const cors = require('cors');
const passport = require('passport');
const mongoose = require('mongoose');
const config = require('./db/config');

// initialize express app
const app = express();
const port = process.env.PORT || 3000;

// mongodb setup
mongoose.connect(config.database);

mongoose.connection.on('connected', () => {
  console.log('Connect to database', config.database);
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

## Step 3: Setup UserRoutes and UserController
We need to create our `routes` folder and touch `user-routes.js` inside of it.
In our `user-routes` we'll set up our routes to talk to our controllers.
In `user-routes`:
```js
const express = require('express');
const userController = require('../controllers/users-controller.js');

const userRouter = express.Router();

userRouter.get('/profile', usersController.index);
userRouter.post('/register', usersController.create);
userRouter.post('/authenticate', usersController.authenticate);
userRouter.post('/validate', usersController.validate);

module.exports = userRouter;

```
Back in our root directory, we want to create our `controllers` folder and touch `user-controller.js` inside of it.
In `user-controller.js`:
```js
const passport = require('passport');
const jwt = require('jsonwebtoken');
const User = require('../models/user');
const config = require('../db/config');

const userController = {};

userController.create = (req, res) => {
  let newUser = new User({
    username: req.body.username,
    firstname: req.body.firstname,
    lastname: req.body.lastname,
    email: req.body.email,
    password: req.body.password,
  });
  
  User.addNewUser(newUser, (err, user) => {
    err ? res.json({success: false, msg: 'Failed to register user'}) : res.json({success: true, msg: 'User registered!'})
  }
}
```
We should be able to test our User creation in postman at the /user route.

<hr>
Now our file structure should look like this:
<details>
<summary> File Structure </summary>
  
```bash
├── README.md
├── app.js
├── controllers
│   └── users-controller.js
├── db
│   └── config.js
├── models
│   └── user.js
├── node_modules
├── package.json
├── public
└── routes
    └── user-routes.js
```
</details>

## Step 4: Setting up passport
In `app.js`, we want to add some passport middleware:
```js
app.use(passport.initialize());
app.use(passport.session());
```

Now let's create a strategy using `passport-jwt`. 
We want to create a `services` folder on our root directory. Inside it let's touch `passport.js`:

Inside `passport.js`:
```js
const JwtStrategy = require('passport-jwt').Strategy;
const ExtractJwt = require('passport-jwt').ExtractJwt;
const User = require('../models/user');
const config = require('../db/config');

module.exports = function(passport) {
  let opts = {};
  opts.jwtFromRequest = ExtractJwt.fromAuthHeaderWithScheme('jwt');
  opts.secretOrKey = config.secret;
  passport.use(new JwtStrategy(opts, (jwt_payload, done) => {
    User.getUserById({jwt_payload.sub}, (err, user) => {
      if (err) return done(err, false);
      return user ? done(null, user) : done(null, false);
    });
  }));
}
```

Back in our `app.js`, we want to import our `passport.js` file and pass the `passport` module to it.
In `app.js`:
```js
const jwtPassport = require('./services/passport');

jwtPassport(passport);
```

Now in our `users-controller.js`, we want to create our `authenticate` method:
In `users-controller.js`:
```js
usersContoller.authenticate = (req, res) => {
  let username = req.body.username;
  let password = req.body.password;
  
  User.getUserByUsername(username, (err, user) => {
    if(err) throw err;
    if (!user) return res.json({success: false, msg: 'User not found'});
  }
  
  User.comparePassword(password, user.password, (err, isMatch) => {
    if(err) throw err;
    if(isMatch) {
      let token = jwt.sign(user, config.secret, {
        expiresIn: 604800 // one week
      });
      
      res.json({
        success: true,
        token: 'JWT', token,
        user: {
          id: user._id,
          username: user.username,
          firstname: user.firstname,
          lastname: user.lastname,
          email: user.email,
        }
      });
    } else {
      return res.json({success: false, msg: 'Wrong password'});
    }
  });
}
