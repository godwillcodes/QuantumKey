

## TypeScript & Vue Integration

TypeScript needs a little extra help to understand `.vue` files. We use `vue-tsc` for type-checking and **Volar** to keep the TypeScript language service in sync with `.vue` types.

## Project Setup

### Install Dependencies

Start by installing the project's dependencies:

```sh
npm install
npm run dev
npm run build
```

## Install browsers (only needed the first time)

npx playwright install

# Build the project (required before running tests)

npm run build

# Run all end-to-end tests

npm run test:e2e

# Run tests on Chromium only

npm run test:e2e -- --project=chromium

# Run a specific test file

npm run test:e2e -- tests/example.spec.ts

# Run tests in debug mode

npm run test:e2e -- --debug
