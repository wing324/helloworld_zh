# How to Deploy an app on Heroku

- First of all, you should have a Heroku account, if you don't have it, just sign up for free.

- Then, you should have an app, if you don't have, you can just use the sample. Assume our source code in `src` folder

  **We have to put our code at root folder, because Heroku will find code at root folder, the root folder in the post is `src` folder**

  ```javascript
  // src/index.js
  
  // import express by using require, since node.js doesn't support import express
  const express = require('express');
  const app = express();
  
  app.get('/', (req, res) => {
    res.send({'hi': 'there'});
  });
  
  // process.env.PORT in order to user Heroku port for our app
  const PORT = process.env.PORT || 5000
  app.listen(PORT);
  
  
  // src/package.json
  // we have to add "engines" field to tell HeroKu what version of node+npm you used
  // we have to add "scripts" field to tell HeroKu how to run our app
  
  {
    "name": "src",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "engines": {
      "node": "10.16.0",
      "npm": "6.9.0"
    },
    "scripts": {
      "start": "node index.js"
    },
    "author": "Wing Yu",
    "license": "ISC",
    "dependencies": {
      "express": "^4.17.1"
    }
  }
  ```

- Install Heroku CLI

  ```shell
  $ npm install -g heroku
  ```

- Check Heroku installation

  ```shell
  $ heroku --version
  heroku/7.37.0 win32-x64 node-v10.16.0
  ```

- Login your Heroku by Heroku CLI

  ```shell
  $ heroku login
  heroku: Press any key to open up the browser to login or q to exit:
  ```

- Go to your app root folder(the post is `src` folder), and create Heroku app

  ```shell
  cd src/
  heroku create
  # After you finish run the command, you will get 2 link
  # link1 | link2
  # link1 is your app website
  # link2 is your app git repo site on Heroku
  ```

- Push your code into GitHub, you should have a git repo for your app.

- Add your git repo into your Heroku git repo

  ```shell
  git remote add heroku link2
  ```

- Push your GitHub code to your Heroku git repo

  ```shell
  git push heroku master
  ```

- Open your app on Heroku 

  ```shell
  heroku open
  # or open link1
  ```

  