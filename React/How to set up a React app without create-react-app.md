# How to set up a React Project without create-react-app

### 1. Set up project

#### Setting up Node.js or Yarn

> You can download Node.js or Yarn from their official website:
> Node.js: [https://nodejs.org/en/download/](<https://nodejs.org/en/download/>)
> Yarn: [https://yarnpkg.com/lang/en/docs/install/#windows-stable](https://yarnpkg.com/lang/en/docs/install/#windows-stable)

Initializing the project

> - Create a new and empty directory for the project, and get into the new directory.
>
>   ```she
>   mkdir newProject
>   cd newProject
>   ```
>
> - Initialize the project
>
>   ```she
>   npm init
>   // or
>   yarn init
>   ```
>
> - Create a `src` directory to store the project source code inside `newProject` directory
>
>   ```shell
>   mkdir src
>   ```
>
> - Utill now, the project directory looks like:
>
>   ```shell
>   newProject
>   |--src
>   ```

#### Setting up Webpack

> - Install Webpack related package
>
>   ```shell
>   npm install --save-dev webpack webpack-dev-server webpack-cli
>   //or
>   yarn add webpack webpack-dev-server webpack-cli
>   ```
>
>   
>
> - Create a new file `webpack.config.js` under `newProject` folder, and put the below code into the `webpack.config.js` file.
>
>   ```
>   const path = require('path');
>   module.exports = {
>       // define entry file and output
>       entry: './src/index.js',
>       output: {
>           path: path.resolve('dist'),
>           filename: 'main.js'
>       },
>       // define babel loader
>       module: {
>           rules: [
>               { test: /\.jsx?$/, loader: 'babel-loader', exclude: /node_modules/ }
>           ]
>       }
>   };
>   ```

#### Setting up Babel

> - Install Babel related package
>
>   ```shell
>   npm install --save-dev babel-core babel-loader babel-preset-es2015
>   npm install --save-dev babel-plugin-transform-object-rest-spread
>   // or
>   yarn add  babel-core babel-loader babel-preset-es2015
>   yarn add babel-plugin-transform-object-rest-spread
>   ```
>
> - Create a new file `.babelrc` under `newProject` folder, and put the below code into the `.babelrc` file.
>
>   ```shell
>   {
>     "presets":[
>       "@babel/preset-env",
>       "@babel/preset-react"],
>     "plugins": [ "transform-object-rest-spread" ]
>   }
>   ```

#### Setting up the entry files

> - Create `newProject/src/index.js` file, and put the below code into the file.
>
>   ```shell
>   console.log('hello world!')
>   ```
>
> - Create `newProject/index.html` file, and put the below code into the file.
>
>   ```shell
>   <!DOCTYPE html>
>   <html>
>     <head>
>       <meta charset="utf-8">
>       <title>React</title>
>     </head>
>     <body>
>       <div id="root"></div>
>       <script src="main.js"></script>
>     </body>
>   </html>
>   ```

#### Running Project

> - Put below code into `package.json` file.
>
>   ```shell
>   "scripts": {
>     "start": "webpack-dev-server",
>     "build": "webpack"
>   },
>   ```
>
> - Run the project.
>
>   ```shell
>   npm start
>   // or
>   yarn start
>   ```

### 2. Set up React

#### Install React

> Use the below command at the `newProject` folder.
>
> ```shell
> npm install --save react react-dom
> // or
> yarn add react react-dom
> ```

### 3. Set up redux

#### Install Redux

> Use the below command at the `newProject` folder.
>
> ```shell
> npm install --save redux react-redux
> // or
> yarn add redux react-redux
> ```
>
> 