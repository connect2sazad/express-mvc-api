# Express MVC API

test

## Step 1

npm init -y

## Step 2

npm i express mysql2 sequelize sequelize-cli nodemon

## Step 3

mkdir controllers
mkdir models
mkdir config
mkdir routes

## Step 4

create app.js

## Step 5

npx sequelize-cli init

## Step 6

set database in config.js

## Step 7

create user.js

```js
// models/user.js
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define('User', {
    name: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
    },
  });
  return User;
};
```

## Step 8

npx sequelize-cli model:generate --name User --attributes name:string,email:string --force

## Step 9

create userController.js

```js
// src/controllers/userController.js
const { User } = require('../models');

exports.getAllUsers = async (req, res) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (error) {
    res.status(500).send(error.message);
  }
};

exports.createUser = async (req, res) => {
  try {
    const { name, email } = req.body;
    const newUser = await User.create({ name, email });
    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).send(error.message);
  }
};
```

## Step 10

create userRoutes.js

```js
// src/routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.get('/users', userController.getAllUsers);
router.post('/users', userController.createUser);

module.exports = router;
```

## Step 11

edit app.js

```js
// src/app.js
const express = require('express');
const bodyParser = require('body-parser');
const userRoutes = require('./routes/userRoutes');
const { sequelize } = require('../models');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());
app.use('/api', userRoutes);

sequelize.sync().then(() => {
  console.log('Database & tables created!');
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```
