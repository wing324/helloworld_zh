# React Use SVG by svgr tool

1. Create `assests` and `Icons` folder under `src` folder.

2. Put SVG source code into `assests` folder.

3. In `package.json` file, we add the following script:

   ```react
   "scripts": {
       "svgr": "svgr -d src/Icons/ src/assets/"
     },
   ```

4. Then, let's download `svgr` tool by Yarn.

   ```shell
   yarn add @svgr/cli
   ```

5. The last step we will run command `yarn run svgr`.

   When the command finished, we can check `Icons` folder, we will find `svgr` tool helps us convert all svg files to react components(js files).