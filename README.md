# Express MVC API [https://chatgpt.com/share/e6f20c59-c465-4cfc-93dd-4cc5bfe8fd22]

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

## Step 12

Adding Migration, Seeding, and Model Update Scripts | update package.json

```js
"scripts": {
  "start": "nodemon src/app.js",
  "migrate": "npx sequelize-cli db:migrate",
  "seed": "npx sequelize-cli db:seed:all",
  "migrate:undo": "npx sequelize-cli db:migrate:undo:all",
  "seed:undo": "npx sequelize-cli db:seed:undo:all"
}
```

## Step 13

npx sequelize-cli seed:generate --name demo-user

## Step 14

update seeder

```js
// seeders/[timestamp]-demo-user.js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.bulkInsert('Users', [
      {
        name: 'John Doe',
        email: 'john.doe@example.com',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
      {
        name: 'Jane Doe',
        email: 'jane.doe@example.com',
        createdAt: new Date(),
        updatedAt: new Date(),
      },
    ], {});
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.bulkDelete('Users', null, {});
  }
};
```

## Step 15

npm run migrate
npm run seed
npm run migrate:undo
npm run seed:undo

## Updating the models

### Step 16

update users.js

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
    password: {
      type: DataTypes.STRING,
      allowNull: false,
    },
  });
  return User;
};
```

### Step 17

npx sequelize-cli migration:generate --name add-password-to-users

### Step 18

edit migration

```js
// migrations/[timestamp]-add-password-to-users.js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('Users', 'password', {
      type: Sequelize.STRING,
      allowNull: false,
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('Users', 'password');
  }
};
```

### Step 19

npm run migrate

### Step 20

npm install bcrypt

### Step 21

update user model

```js
// models/user.js
const bcrypt = require('bcrypt');

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
    password: {
      type: DataTypes.STRING,
      allowNull: false,
    },
  }, {
    hooks: {
      beforeCreate: async (user) => {
        const salt = await bcrypt.genSalt(10);
        user.password = await bcrypt.hash(user.password, salt);
      },
      beforeUpdate: async (user) => {
        if (user.changed('password')) {
          const salt = await bcrypt.genSalt(10);
          user.password = await bcrypt.hash(user.password, salt);
        }
      },
    },
  });

  return User;
};
```

### Step 22

update controller

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
    const { name, email, password } = req.body;
    const newUser = await User.create({ name, email, password });
    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).send(error.message);
  }
};
```

### Step 23

example login

```js
// Example login function
exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ where: { email } });

    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    const isMatch = await bcrypt.compare(password, user.password);

    if (!isMatch) {
      return res.status(401).json({ message: 'Incorrect password' });
    }

    res.status(200).json({ message: 'Login successful' });
  } catch (error) {
    res.status(500).send(error.message);
  }
};
```
