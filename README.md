# Initial Setting

2022-07-22

## Install next.js

next.js 12.22.0 + react 18.2.0 + typescript 4.7.4

```bash
npx create-next-app@latest --typescript
```

## Install release-please 13.18.7

```bash
npm install -D release-please
```

Set release-please Action

Create `.github/workflows/release-please.yml`

```yml
on:
  push:
    branches:
      - main
name: release-please
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/release-please-action@v3
        with:
          release-type: node
          package-name: release-please-action
```

## husky + lint-staged

```bash
npx husky-init && npm install
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'

npm install -D lint-staged


npx husky set .husky/pre-commit 'npx lint-staged'
```

npx --no-install commitlint --edit "$1" -> $1 추가안될시 위치에 직접 추가

.lintstagedrc.js

```js
const path = require('path');

const buildEslintCommand = (filenames) =>
  `next lint --fix --file ${filenames
    .map((f) => path.relative(process.cwd(), f))
    .join(' --file ')}`;

module.exports = {
  '*.{js,jsx,ts,tsx}': [buildEslintCommand],
};
```

## prettier + eslint + commitlint

package.json

```json
{
  "lint-staged": {
    "src/**/*.{js,jsx,ts,tsx}": ["eslint --max-warnings=0", "prettier -w"],
    "src/**/*.{json,css,scss,md}": ["prettier -w"]
  }
}
```

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

commitlint.config.js

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'body-leading-blank': [1, 'always'],
    'body-max-line-length': [2, 'always', 100],
    'footer-leading-blank': [1, 'always'],
    'footer-max-line-length': [2, 'always', 100],
    'header-max-length': [2, 'always', 100],
    'scope-case': [2, 'always', 'lower-case'],
    'subject-case': [
      2,
      'never',
      ['sentence-case', 'start-case', 'pascal-case', 'upper-case'],
    ],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'chore',
        'style',
        'refactor',
        'ci',
        'test',
        'perf',
        'revert',
        'build',
      ],
    ],
  },
};
```

.eslintrc.json

```bash
npm install -D eslint-config-prettier
npm install -D typescript @typescript-eslint/parser
npm install -D @typescript-eslint/eslint-plugin
npm install -D eslint-plugin-unused-imports
```

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint", "unused-imports"],
  "extends": [
    "next",
    "prettier",
    "eslint:recommended",
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "no-unused-vars": "off",
    "no-console": "warn",

    "@typescript-eslint/no-unused-vars": "off",
    "unused-imports/no-unused-imports": "error",
    "unused-imports/no-unused-vars": [
      "warn",
      {
        "vars": "all",
        "varsIgnorePattern": "^_",
        "args": "after-used",
        "argsIgnorePattern": "^_"
      }
    ]
  },
  "ignorePatterns": [],
  "globals": {}
}
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "~/*": ["./public/*"]
    }
  },
  "include": ["next-env.d.ts", "src/**/*.ts", "src/**/*.tsx"],
  "moduleResolution": ["node_modules", ".next", "node"]
}
```

## vercel

vercel-ignore-build-step.sh

```bash
#!/bin/bash

echo "VERCEL_GIT_COMMIT_REF: $VERCEL_GIT_COMMIT_REF"

if [[ "$VERCEL_GIT_COMMIT_REF" == "main" || "$VERCEL_GIT_COMMIT_REF" == "dev"  ]] ; then
  # Proceed with the build
	echo "✅ - Build can proceed"
  exit 1;

else
  # Don't build
  echo "🛑 - Build cancelled"
  exit 0;
fi
```

vercel.json

```json
{
  "github": {
    "silent": true
  }
}
```

vercel - project - setting - git

ignored build step 항목에 추가

bash vercel-ignore-build-step.sh

## tailwindcss

```bash
npm install -D tailwindcss postcss autoprefixer
npm install -D @tailwindcss/forms
npx tailwindcss init -p
```

tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [require('@tailwindcss/forms')],
};
```

global.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

\_document.tsx

## jest

```bash
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom
npm install -D eslint-plugin-testing-library
```

.eslint.json

```json
{
  "plugins": ["testing-library"],
  "overrides": [
    // Only uses Testing Library lint rules in test files
    {
      "files": [
        "**/__tests__/**/*.[jt]s?(x)",
        "**/?(*.)+(spec|test).[jt]s?(x)"
      ],
      "extends": ["plugin:testing-library/react"]
    }
  ]
}
```

package.json

```json
{
  "scripts": {
    "test": "jest --watch",
    "test:ci": "jest --ci"
  }
}
```

jest.config.js

```js
// eslint-disable-next-line @typescript-eslint/no-var-requires
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});
const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleDirectories: ['node_modules', '<rootDir>/'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^~/(.*)$': '<rootDir>/public/$1',
  },
};
module.exports = createJestConfig(customJestConfig);
```

jest.setup.js

```js
import '@testing-library/jest-dom/extend-expect';
```

## cypress

```bash
npm install -D cypress
npm install -D eslint-plugin-cypress
```

.eslintrc.json

```json
{
  "plugins": ["cypress"],
  "env": {
    "cypress/globals": true
  },
  "extends": ["plugin:cypress/recommended"]
}
```

package.json

```json
{
  "scripts": {
    "cypress": "cypress open",
    "cypress:headless": "cypress run",
    "e2e": "start-server-and-test start http://localhost:3000 cypress",
    "e2e:headless": "start-server-and-test start http://localhost:3000 cypress:headless"
  }
}
```

[jest-cypress](https://docs.cypress.io/guides/component-testing/component-test-troubleshooting#Cypress-Types-Conflict-with-Jest)

```ts
/// <reference types="cypress" />
```

[.github/workflows/cypress.yml](https://docs.cypress.io/guides/continuous-integration/github-actions#What-you-ll-learn)

---

## 추가필요

## next-sitemap + next-seo

## lighthouse

## storybook
