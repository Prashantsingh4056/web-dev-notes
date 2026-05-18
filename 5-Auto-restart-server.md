## To Auto run project 

*nodemon is a tool that helps develop Node.js based applications by automatically restarting the node application when file changes in the directory are detected.*

*nodemon does not require any additional changes to your code or method of development. nodemon is a replacement wrapper for node. To use nodemon, replace the word node on the command line when executing your script.*

# Setup

`npm install --save-dev nodemon`

```
"devDependencies": {
    "nodemon": "^3.1.14",
    "prettier": "3.8.1"
  }
```

Then in `package.json` file : 

Change these lines to :
```
"scripts": {
    "dev": "nodemon src/index.js",
    "start": "node src/index.js"
  }
```