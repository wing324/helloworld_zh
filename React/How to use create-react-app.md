# How to use create-react-app

1. Install Node.js

   <https://nodejs.org/en/>

2. Install create-react-app

   ```shell
   # -g means gloal
   npm install -g create-react-app
   ```

3. Navigate to the folder where you want to create your new project, and create a project by using create-react-app, for example the new project name is "helloworld"

   ```shell
   create-react-app helloworld
   ```

4. Preview your project in your browser

   ```shell
   cd helloworld
   npm start / yarn start
   ```

5. Create a production build and you will get a "build" folder

   ```shell
   npm run build
   ```

   When this has completed, you can follow the onscreen prompts to deploy it to your server or just test it locally using the popular `serve` node package

   