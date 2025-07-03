## Recommended Folder Structure for AWS Lambda (JavaScript)

```
/my-lambda-project
│
├── /src
│   ├── /handlers         # Lambda handler functions (entry points)
│   │    └── index.js
│   ├── /services         # Business logic / service layer (core functions)
│   ├── /libs             # Utility functions, helpers, shared code
│   ├── /models           # Data models, schema validation (if any)
│   └── /config           # Configuration files (e.g., constants, env vars)
│
├── /tests                # Unit and integration tests
│   └── handler.test.js
│
├── package.json
├── jest.config.js        # or other test framework configs
├── .env                  # environment variables for local development
└── README.md
```

### Explanation
- /src: Contains all the source code. This keeps your code isolated from tests, configs, docs, etc.
- /handlers: Lambda entry points, i.e., the functions exported as Lambda handlers that AWS calls. These should be thin wrappers calling your business logic.
- /services: Core business logic independent of AWS. You can unit test these easily.
- /libs: Helper utilities, like date formatting, logging wrappers, etc.
- /models: Data schemas, validation, or classes modeling your domain.
- /config: Configuration variables, constants, environment-specific values.
- /tests: Contains your unit and integration tests corresponding to the source files.
- Root files like package.json for dependencies, test config files, and .env for local environment variables.

### Why this structure?
- Separation of concerns: Keeps Lambda-specific code (handlers) separate from business logic.
- Scalability: As your project grows, you can add more services or utilities without clutter.
- Testability: Business logic can be tested in isolation.
- Reusability: Utility libraries and services can be shared across multiple Lambdas if you have a mono repo or multi-function repo.

### Example handler calling service
```
// src/handlers/index.js
const { processData } = require('../services/myService');

exports.handler = async (event) => {
  const result = processData(event.input);
  return {
    statusCode: 200,
    body: JSON.stringify(result),
  };
};
```

```
// src/services/myService.js
function processData(input) {
  return input * 2;
}

module.exports = { processData };
```

### Simple Build Script Example using npm + zip (no bundler)

If you don’t use bundlers like Webpack or esbuild, the simplest way is:

1. Install dependencies
2. Copy source + node_modules to a build folder
3. Zip the contents of that folder

### 1. Add a build folder and a simple package.json script

```
{
  "scripts": {
    "clean": "rm -rf build",
    "install:prod": "npm install --production",
    "copy-src": "cp -r src build/src",
    "copy-package": "cp package.json build/",
    "zip": "cd build && zip -r ../lambda-package.zip .",
    "build": "npm run clean && npm run install:prod && npm run copy-src && npm run copy-package && npm run zip"
  }
}
```

### 2. What does each script do?

- clean: Removes the old build folder.
- install:prod: Installs only production dependencies (no devDependencies).
- copy-src: Copies your source code into the build directory.
- copy-package: Copies package.json (needed for dependencies).
- zip: Creates a ZIP file from the build folder contents.
- build: Runs all the steps in sequence.

### 3. Notes:

- Run npm run build — this will generate lambda-package.zip ready to upload to AWS Lambda.
- Make sure your Lambda entry point is correctly referenced (e.g., src/handlers/index.handler if your handler is exports.handler in index.js).
- If your code uses native modules or binaries, you might need extra packaging steps.

