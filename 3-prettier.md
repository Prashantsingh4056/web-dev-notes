*Prettier is a popular code formatting tool used by software developers. It takes your code and automatically rewrites it to follow a consistent style (e.g., proper spacing, line breaks, and indentation) every time you save.*

## Setup :
`npm install --save-dev --save-exact prettier`

then ->

`npx prettier . --write`

```
"devDependencies":{
    "prettier" : "3.6.2"
}
```

**After that**  

make a file -> `.prettierrc`

then write :
```
{
    "tabWidth" : 2,
    "useTabs" : false,
    "semi" : true,
    "singleQuote" : false,
    "trailingComma" : "all",
    "bracketSpacing" : true,
    "arrowParens" : "always"
}
```

**But we don't want all files to be formatted**

for that , make a file -> `.prettierignore`

write : 

```
node_modules
env 
```