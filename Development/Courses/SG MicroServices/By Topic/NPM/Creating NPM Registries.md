Organization is probably the best way of doing this.

### Publishing Packages
- Package.json:
- Notice org/dir structure of name
```json
{

  "name": "@parisbtickets/common",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
	"build": "npm run clean && tsc"
	"clean": "del ./build*"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```
- common folder must be a git repo
``` powershell
npm publish --access public
```

---
## Sharing TypeScript

### Setup
- We used a particualr version of TS. Need to protect against version differences and even the fact we might use JS.
- Need to transpile it to JS before publishing
-  Tooling for this:
``` Powershell
tsc --init 
npm i typescript del-cli --save-dev
```

In tsconfig.json:
- Uncomment 'declaration' 
- Uncomment 'build' and specify ''.\\build'

Pack ing pkg.json
-  Good practice to remove everything from build when a change is made, so need to make a clean script to delte build dir.
- Change 'main' (this is the file that the import statement reaches into to find what's being imported)
-  Add 'types' (types definition file.)
- Add 'files' (tells npm what to include final published version of package)
- Add .gitingnore and add build and node_modules

```json
{
  "name": "@parisbtickets/common",
  "version": "1.0.0",
  "description": "",
  "main": "./build/index.js",
  "types": "./build/index.d.ts",
  "files": [
    "build/**/*"
  ],
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "tsc"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "del-cli": "^5.0.0",
    "typescript": "^4.8.4"
  }
}
```

### Making a change and publishing
For example, exporting somethign fromwithin the package:

```typescript
#index.ts
interface Colour {
  red: number;
  blue: number;
}
const colour: Colour = {
  red: 10,
  blue: 72373,
};
console.log(colour);
export default colour;

```

Process: 
1) `git add .; commit -m 'somethign added'`
2) increment version of package `npm version patch`
3) `npm run build`
4) `npm publish`

but let's not do this manually! (but not in a real project)

add script to pkg.json:

`"pub": "git add. && git commit -m \"Updates\" && nm version patch && npm run build && npm publish"`


## Exporting modules and importing them

Import to index.ts, export from index.ts
```typescript
export * from "./errors/badrequesterror";
export * from "./errors/customerror";
export * from "./errors/databseconnectionerror";
export * from "./errors/notfounderror";
export * from "./errors/forbiddenerror";
export * from "./errors/requestvalidationerror";
export * from "./middlewares/currentuser";
export * from "./middlewares/errorhandler";
export * from "./middlewares/requireauth";
export * from "./middlewares/validaterequest";
```
Be sure to install all dependencies in  the package:

`
`npm i express express-validator cookie-session jsonwebtoken @types/cookie-session @types/express @types/jsonwebtoken`

## Making changes to common package

1) make Changes
2) Publish changes
3) Update package:
`npm update @parisbtickets/common`
