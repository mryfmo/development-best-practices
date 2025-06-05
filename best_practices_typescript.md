# TypeScript Best Practices (TypeScript 5.5 & 5.4)

## 1. Toolchain & Environment

In TypeScript development, a toolchain that leverages its robust type system while enabling fast builds, testing, and efficient package management is crucial. Modern tools like `Volta` for Node.js version management, `pnpm` or `Bun` for package management, and `Biome` for linting/formatting are recommended. For AI application development, considerations include handling large datasets, interfacing with numerical computation libraries, and potentially sharing types with backends written in Python/Rust.

| Purpose                               | Recommended Tool                                                                 | Key Points & Rationale                                                                                                                               |
| ---------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Node.js / Package Manager Version Management** | **Volta**                                                                  | Automatically switches and uses Node.js and package manager (pnpm, yarn, npm) versions pinned in `package.json` per project. Resolves environment discrepancies in team development. |
| **Package Manager**                 | **pnpm** (Recommended) / **Bun install** (Alternative)                                   | `pnpm` features a symlink/hardlink-based `node_modules` structure saving disk space and fast installs. `Bun` provides fast installation as part of its all-in-one toolset. |
| **Monorepo Management**                     | **Turborepo** / **Nx**                                                       | Integrates with `pnpm workspaces`, etc., to efficiently cache and parallelize builds, tests, and deployments in large monorepos.                                          |
| **Lint / Format**                    | **Biome** (formerly Rome)                                                           | A fast, integrated tool built in Rust. Replaces ESLint, Prettier, and TypeScript ESLint functionalities with simpler configuration. Run `biome check --apply` for simultaneous linting and formatting.       |
| **Type-Safe APIs / Data Validation**         | **`ts-reset`** + **Zod**                                                     | `ts-reset` overrides standard library type definitions to be stricter and safer (e.g., `JSON.parse` returns `unknown`). Zod provides runtime schema definition and validation, synchronized with TypeScript types. |
| **Testing Framework**                 | **Vitest** (Recommended) / **Bun test** (Alternative)                                    | Vite-based fast test runner. Offers Jest-compatible API and native ESM/TS support. `Bun test` also runs very fast on the Bun runtime.                     |
| **Build / Bundling**                  | **tsup** (for libraries) / **Vite**, **ESBuild**, **Bun build** (for apps)        | `tsup` is ESBuild-based, specialized for building TypeScript libraries with easy configuration. Vite, ESBuild, and Bun build offer very fast bundling. SWC-based tools are also alternatives.     |
| **Type Definition Testing**                     | **`dtslint`** / **`tsd`** / **`// @ts-expect-error`**, **`// @ts-ignore`** (sparingly) | Tests if library type definitions (`.d.ts`) function as intended.                                                                                   |
| **Documentation Generation**             | **TypeDoc**                                                                | Generates API documentation from TSDoc comments.                                                                                                          |
| **AST Manipulation/Code Generation**             | **ts-morph**, **TypeScript Compiler API**                                    | Supports advanced use cases like code analysis, refactoring, and code generation.                                                                        |

## 2. Coding Style

TypeScript coding style builds upon JavaScript best practices while aiming to maximize the benefits of TypeScript's unique type system. Integrated tools like Biome facilitate the application of consistent style.

*   **Consistent Style with Biome:**
    *   Use **Biome** to unify code linting and formatting. Biome covers many rules from Prettier and ESLint (+ TypeScript ESLint) and operates with high speed due to its Rust implementation.
    *   Create a `biome.json` in the project root to configure project-specific settings (indentation style [2 or 4 spaces], line length [e.g., 80 or 100 characters], quote style [single or double], semicolon usage [always or as-needed]), and lint rules.
    *   Enforce consistent style across the team by integrating Biome into commit hooks (e.g., `lint-staged` with Biome) and CI (running `biome check --apply` or `biome format --write` and `biome lint --apply`).
*   **Naming Conventions:**
    *   Variables, functions, method names, property names: `camelCase` (e.g., `userName`, `calculateTotalAmount`).
    *   Classes, interfaces, type aliases, enum names: `PascalCase` (e.g., `UserService`, `IUser`, `UserId`, `HttpStatus`).
    *   Constants (global values that don't change, enum members): `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`, `HttpStatus.OK`) or `PascalCase` (for enum members).
    *   Generic type parameters: Short single uppercase letters (`T`, `K`, `V`, `E`) are conventional, but more descriptive `PascalCase` names (e.g., `TResponse`, `TError`) can improve readability.
    *   Private properties or methods: Prefer ECMAScript private fields (`#fieldName`, `#methodName()`). The older convention of an `_` prefix is still seen but `#` provides stronger privacy.
*   **File and Folder Structure:**
    *   Modularize by feature, domain, or layer (e.g., `src/features/user`, `src/lib/utils`, `src/core/services`).
    *   **Barrel files (`index.ts`):** Re-export the public API of a module to simplify import paths from outside (e.g., `import { UserService } from '@/services';`). However, overuse or overly large barrel files can risk circular dependencies or hinder tree-shaking; consider splitting entry points for larger modules.
    *   Name files consistently according to their content (e.g., React components as `PascalCase.tsx`, hooks as `useCamelCase.ts`, others as `camelCase.ts` or `kebab-case.ts`).
*   **Classes vs. Functions:**
    *   Use classes when needing to encapsulate state and related behavior closely (e.g., domain entities, services, base UI components) or when leveraging inheritance/polymorphism.
    *   Prefer plain functions or arrow functions for simple data transformations, utilities, and stateless logic. Incorporate functional programming concepts where appropriate (pure functions, immutability, higher-order functions).
    *   Be mindful of the "Composition over inheritance" principle to avoid unnecessarily deep and complex class hierarchies.
*   **Comments and Documentation (TSDoc):**
    *   **TSDoc (`/** ... */`):** Write for all public APIs (functions, classes, interfaces, type aliases, methods, properties, enum members, etc.).
        *   Use standard tags like `@param` (argument with its type and description), `@returns` (return value type and description), `@throws` (potential exceptions), `@template` (generic type parameters), `@deprecated`, `@see` (links to related information), and `@example` (usage examples) appropriately.
        *   Since type information is often clear from the code, TSDoc should primarily describe the logic's intent, side effects, preconditions/postconditions, algorithmic overviews, and design rationale.
        *   API documentation can be generated using tools like TypeDoc. The `@typescript-eslint/eslint-plugin`'s `eslint-plugin-tsdoc` can validate TSDoc syntax.
    *   **Inline Comments (`// ...`):** Use to supplement the intent of a line of code or temporarily explain complex parts. Avoid comments for obvious code.
    *   **Block Comments (`/* ... */`):** Use for multi-line comments or temporarily commenting out code.
    *   **TODO/FIXME Comments:** Use formats like `// TODO: Description (@username optional)` or `// FIXME: Description` to indicate future tasks or areas needing correction. Linters can be configured to detect these tags.
*   **Import Organization:**
    *   Group `import` statements at the top of the file, typically in logical groups (e.g., Node.js built-in modules, external libraries, internal modules/absolute paths, internal modules/relative paths). Sort alphabetically within each group (Biome can automate this).
    *   **`import type { ... } from 'module';`**: Explicitly use for type-only imports. This ensures the import is completely erased at build time, preventing runtime side effects (especially important with `isolatedModules: true`).
    *   **`verbatimModuleSyntax: true` (`tsconfig.json`):** Recommended since TypeScript 5.0. Enforces the use of `import type` and makes the distinction between type and value imports stricter, replacing older options like `importsNotUsedAsValues` and `preserveValueImports`.
    *   Prefer named exports (`export const foo = ...; export class Bar ...;`) over default exports (`export default`). Reasons include better refactoring support, easier auto-imports by IDEs, avoidance of naming collisions, and more efficient tree-shaking.
*   **Avoid `any`; Use `unknown`:** (See "8. Type System Usage" for details) `any` destroys type safety; avoid it whenever possible. Use `unknown` for values whose type is truly unknown, and then use type guards or runtime validation (like Zod) to handle them safely.
*   **Idioms for Readability and Maintainability:**
    *   **Ternary operator (`condition ? exprT : exprF`):** Limit to simple conditional assignments; avoid nesting. Use `if/else` for complex conditions.
    *   **Early returns/Guard clauses:** Handle invalid inputs or edge cases at the beginning of a function and return early. This keeps the main logic less nested and more readable.
    *   **Magic numbers and hardcoded strings:** Define as named constants using `enum` (especially string enums), `as const` objects, or module-level `const` variables.
    *   **Promise-based asynchronous operations:** Use `async/await` extensively to avoid callback hell and nested Promise chains. Handle errors with `try...catch`.
    *   **Immutability:** Use immutable data structures whenever possible (`readonly` modifier, `Readonly<T>`, `ReadonlyArray<T>`, spread syntax, immutable array methods like `map`, `filter`, `reduce`, `toSorted`, `toReversed`). This makes state changes easier to track and prevents unexpected side effects.
    *   **Nullish Coalescing (`??`) and Optional Chaining (`?.`):** Utilize to handle `null` or `undefined` values safely and concisely.
    *   **Destructuring Assignment:** Enhances readability when extracting values from objects and arrays, but avoid overly nested destructuring.

## 3. Security

*   **Strict Compiler Options:** Enable `"strict": true` in `tsconfig.json`. This activates several options that help detect potential errors at compile time:
    *   `strictNullChecks`: Forces explicit handling of `null` and `undefined`, significantly reducing null-pointer-like errors.
    *   `noImplicitAny`: Prevents variables from implicitly defaulting to `any` when their type cannot be inferred, enforcing explicit typing.
    *   `noUncheckedIndexedAccess` (TS 4.1+): Makes array and object index access return `T | undefined`, forcing existence checks (recommended to enable separately as it's not part of `strict`).
    *   `exactOptionalPropertyTypes` (TS 4.4+): Distinguishes whether an optional property explicitly allows `undefined`.
*   **Runtime Data Validation:** Since TypeScript types are erased at compile time, validate all external inputs (API responses, user input, files, environment variables, data from databases, etc.) at runtime.
    *   Use schema validation libraries like **Zod**, **io-ts**, **TypeBox**, **Valibot**, or **class-validator** to validate data shape, types, and constraints. Zod is popular for its ease of integration with TypeScript types.
*   **DOM XSS (Cross-Site Scripting) Mitigation (Frontend):**
    *   Modern frameworks like React, Angular, and Vue provide default XSS protection via escaping. Do not disable these features.
    *   Avoid setting `innerHTML` directly with user input. Use `textContent` or a trusted sanitization library (like DOMPurify).
    *   Consider using the Trusted Types API to fundamentally prevent string-based DOM XSS.
*   **Injection Attack Mitigation (Backend):**
    *   SQL Injection: Use parameterized queries or prepared statements provided by ORMs or query builders (TypeORM, Prisma, Knex.js, etc.). Never concatenate user input directly into SQL strings.
    *   Command Injection: When using `child_process.exec` or similar, strictly sanitize user input or, preferably, use `execFile` to pass command and arguments separately.
    *   NoSQL Injection: Even with NoSQL databases like MongoDB, be cautious when constructing queries with user input. Use appropriate escaping or validation.
*   **Authentication and Secret Management:**
    *   Do not embed API keys or secrets in frontend code. Manage these in the backend and access via a proxy if needed. If embedding environment variables at build time, limit them to non-sensitive, publicly exposable ones.
    *   In the backend, load secrets from environment variables or secret management services (AWS Secrets Manager, HashiCorp Vault, Google Cloud Secret Manager, etc.).
    *   Hash passwords using strong algorithms like bcrypt or Argon2.
*   **Secure API Usage:**
    *   Use Web Crypto API (browser) or Node.js `crypto` module for cryptographic operations; avoid implementing custom crypto algorithms.
    *   Use `crypto.getRandomValues()` (browser) or `crypto.randomBytes()` (Node.js) for generating cryptographically secure random numbers (`Math.random()` is not secure).
*   **Content Security Policy (CSP) and Security Headers:**
    *   For frontend applications, set a CSP to allow resource loading only from trusted sources, mitigating XSS and other attacks. Configure directives like `script-src`, `style-src`, `img-src`, `connect-src` appropriately.
    *   In the backend, use middleware like Helmet (for Express.js) to set security-related HTTP headers such as `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security` (HSTS), `Referrer-Policy`, and `Permissions-Policy`.
*   **Dependency Vulnerability Management:**
    *   Regularly run `pnpm audit --audit-level=critical` (or `npm audit`, `yarn audit`) to check for known vulnerabilities in dependencies.
    *   Implement tools like GitHub Dependabot or Snyk for automatic detection and notification/PR creation for vulnerable dependencies.
    *   Commit lock files and use `--frozen-lockfile` (pnpm/yarn) or `ci` (npm) commands in CI to prevent unintended dependency updates.
*   **Avoid `eval()` and `new Function()`:** These functions, which execute code from dynamic strings, pose high security risks and should be avoided.
*   **Prototype Pollution Prevention:** When recursively merging objects based on user input, be aware of prototype pollution attacks. Avoid using untrusted keys or use dedicated libraries. Creating objects with `Object.create(null)` (no prototype) can also be effective.
*   **Enforce HTTPS:** Always use HTTPS in production to protect sensitive information. Set `Secure`, `HttpOnly`, and `SameSite=Lax` or `SameSite=Strict` attributes on cookies appropriately.
*   **AI Application-Specific Security:**
    *   **Untrusted Training Data:** When using externally sourced training data or user-generated content for training, validate it for malicious content (e.g., offensive material, privacy-violating data, data intended to embed backdoors).
    *   **Inference API Protection:** Implement proper authentication and authorization. Introduce rate limiting if necessary. Validate input size and format to prevent excessive resource consumption.
    *   **Sensitive Information Masking:** Mask personal or sensitive information in AI-generated text or logs.
    *   **Configuration Flaws:** Check for weak default passwords, publicly exposed management interfaces, and unnecessarily open ports.

## 4. Performance Optimization

*   **Choose Appropriate Data Structures:** Use `Map` (for fast key-based access), `Set` (for unique value collections and fast existence checks), and `Array` (for ordered lists) according to the use case. `WeakMap`/`WeakSet` can help prevent memory leaks in certain scenarios.
*   **Minimize DOM Operations (Frontend):**
    *   When using Virtual DOM libraries (React, Vue) or differential update mechanisms (Angular, Svelte), follow their optimization best practices (e.g., React's `key` attribute, `memo`, `useMemo`, `useCallback`).
    *   If manipulating the DOM with vanilla JS, batch updates (e.g., using `DocumentFragment`), use event delegation, and schedule rendering operations with `requestAnimationFrame` as DOM access is expensive.
*   **Leverage Asynchronous Operations (Backend):** In Node.js environments, use non-blocking asynchronous APIs (`async/await`, Promises) for I/O-bound operations to avoid blocking the event loop. Offload CPU-bound heavy processing to worker threads (`worker_threads`).
*   **Code Splitting and Lazy Loading:**
    *   Use modern bundlers (Webpack, Rollup, Parcel, ESBuild, Bun) to perform code splitting.
    *   Utilize dynamic imports (`await import('./module')`) on a route or component basis to load only the necessary code for the initial view, deferring the rest for on-demand loading.
*   **Tree Shaking:** Use ES module syntax to enable bundlers to eliminate unused code (dead code) through tree shaking. Be mindful of module imports with side effects (`sideEffects` in `package.json`).
*   **Memoization and Caching:**
    *   Memoize results of expensive pure functions (e.g., using `lodash.memoize`, `reselect` for Redux) to avoid re-computation.
    *   Cache frequently accessed but infrequently changing data (like API responses) on the client-side (e.g., Service Worker Cache API, IndexedDB) or server-side (e.g., Redis, Memcached). Set HTTP cache headers (ETag, Cache-Control) appropriately.
*   **Bundle Size Optimization:**
    *   Analyze dependencies (e.g., with `webpack-bundle-analyzer`, `source-map-explorer`) and consider alternatives for unnecessary or large libraries (e.g., date-fns vs. moment.js).
    *   Optimize and compress static assets like images and fonts (e.g., WebP, AVIF, WOFF2).
    *   Configure `target` and `lib` in `tsconfig.json` appropriately to reduce unnecessary polyfills. Specify more modern ECMAScript versions if targeting up-to-date browsers.
*   **Loops and Iterations:** Loops processing large amounts of data can impact performance.
    *   The performance characteristics of traditional `for` loops, `for...of`, and array methods (`forEach`, `map`, `filter`) vary by JavaScript engine and use case. Profile if a bottleneck is suspected.
    *   In most cases, prioritize readability and maintainability over micro-optimizations.
*   **Web Workers (Frontend):** Offload CPU-intensive computations (image processing, data analysis, complex algorithms) to Web Workers to keep the main (UI) thread responsive. Libraries like Comlink can simplify communication with Workers.
*   **Server-Side Rendering (SSR) / Static Site Generation (SSG) / Incremental Static Regeneration (ISR):** Consider SSR, SSG, or ISR using frameworks like Next.js, Nuxt.js, Astro, or Remix to improve initial page load performance (FCP, LCP) and SEO for web applications.
*   **Profiling:** Use browser developer tools (Performance tab, Lighthouse) or Node.js profilers (`--prof` option, `node --inspect`) to identify performance bottlenecks.
*   **TypeScript Compiler Options:**
    *   `isolatedModules: true` is recommended. It ensures each file can be transpiled independently, improving compatibility with transpilers like Babel and esbuild.
    *   For development, use `incremental: true` builds and fast execution tools like `tsx`, `swc-node`, or `bun run` to reduce build/startup times.
*   **WebAssembly (Wasm):** For extremely performance-critical computations (game physics, video encoding, parts of machine learning model inference), consider compiling code written in Rust or C++ to WebAssembly and using it from TypeScript.
*   **Edge Computing:** Consider using edge computing platforms (Cloudflare Workers, Vercel Edge Functions, Deno Deploy, etc.) for faster API responses and offloading processing closer to users geographically.

## 5. Testing and CI/CD

*   **Choice of Testing Framework:**
    *   **Vitest:** A Vite-based fast test runner. Offers a Jest-compatible API and native TypeScript/ESM support. Features minimal configuration and fast feedback with Hot Module Replacement (HMR).
    *   **Bun test:** A fast test runner built into the Bun runtime, also Jest-compatible.
    *   **Jest:** Still widely used, but Vitest or Bun test may offer better performance for large projects. TypeScript support via `ts-jest` or Babel.
*   **Types of Tests:**
    *   **Unit Tests:** Test individual functions, classes, or components in isolation. Use mocks and stubs effectively. Focus on business logic and pure functions.
    *   **Integration Tests:** Test the interaction between multiple modules or services, such as API endpoints or database integrations.
    *   **End-to-End (E2E) Tests:** Use tools like Playwright (recommended), Cypress, or Puppeteer to simulate user interactions and test entire application flows. Focus on critical user scenarios.
    *   **Snapshot Tests:** Record the output of UI components or large data structures to detect unintended changes. Supported by Jest and Vitest. Reviewing changes is crucial.
    *   **Visual Regression Tests:** Integrate Storybook with tools like Chromatic or Percy to detect visual changes in UI components.
*   **Assertion Libraries:** Use the one built into your test framework (e.g., `expect` in Vitest/Jest) or libraries like Chai.
*   **Mocks and Stubs:** Use mocking features of Vitest/Jest (`vi.mock`, `vi.spyOn`, `jest.mock`, `jest.spyOn`) or libraries like Sinon.JS to control external dependencies and side effects.
*   **Test Coverage:** Measure test coverage with Vitest (`v8` or `istanbul`) or Jest (`--coverage`) and report it in CI. Focus on the quality of tests (covering important logic) as much as the percentage. Use `@vitest/coverage-v8` or `@vitest/coverage-istanbul`.
*   **TypeScript and Testing:**
    *   Write test code in TypeScript to benefit from type safety.
    *   Use utility types (`Partial`, `Required`, etc.) or test library type definitions (e.g., `vi.Mocked`) for typing mock objects.
    *   Test type definitions themselves using `// @ts-expect-error` comments or tools like `dtslint` or `tsd`.
*   **Continuous Integration (CI):**
    *   Set up CI services (GitHub Actions, GitLab CI, etc.) to automatically perform the following on every push and pull request:
        1.  Install dependencies (e.g., `pnpm install --frozen-lockfile`). Consider store caching for `pnpm` in CI.
        2.  Linting and formatting checks (e.g., `biome check .` or `biome lint . && biome format --check .`).
        3.  Type checking (e.g., `tsc --noEmit`).
        4.  Unit and integration test execution (e.g., `vitest run --coverage`).
        5.  Build (e.g., `pnpm build`).
        6.  (Optional) E2E test execution.
        7.  (Optional) Vulnerability scanning (e.g., `pnpm audit --audit-level=high`).
*   **Parallel and Isolated Tests:** Assume tests run in parallel and write them to be independent of each other. Avoid shared state and order dependency.
*   **Continuous Deployment (CD):** After successful tests and builds in CI, automate deployment to staging and production environments. Utilize serverless platforms (Vercel, Netlify, AWS Lambda) or containers (Docker + Kubernetes/ECS). Consider canary releases or blue/green deployments.
*   **Faster Development Feedback:** Use Vitest's watch mode with HMR, and ESBuild/SWC-based transpilers (like `tsx`, `swc-node`, `bun run`) to speed up test execution and build feedback during development.

## 6. Module Structure & Packaging (for libraries and applications)

*   **Adopt ES Modules (ESM):**
    *   Use ESM (`import`/`export` syntax) whenever possible. Node.js now has mainstream ESM support (specify `"type": "module"` in `package.json`), and bundlers primarily use ESM.
    *   If interoperability with CommonJS (`require`/`module.exports`) is needed, configure and build carefully (`tsup` and other tools support dual CJS/ESM builds).
*   **Proper `package.json` Configuration:**
    *   Accurately describe basic information: `name`, `version`, `description`, `license`, `author`, `repository`, `keywords`.
    *   Correctly set `main` (CJS entry point), `module` (ESM entry point), `types` (type definition file entry point), and `exports` (Node.js Conditional Exports, the most recommended way to define entry points).
        ```json
        // package.json example for a library
        {
          "name": "my-library",
          "version": "1.0.0",
          "type": "module",
          "main": "./dist/index.cjs", // Fallback for older CJS environments
          "module": "./dist/index.js", // ESM entry for bundlers
          "types": "./dist/index.d.ts", // Main type definition file
          "exports": {
            ".": { // Main entry point
              "import": { // For ESM consumers (e.g., import ... from 'my-library')
                "types": "./dist/index.d.ts",
                "default": "./dist/index.js"
              },
              "require": { // For CJS consumers (e.g., require('my-library'))
                "types": "./dist/index.d.cts", // Separate CJS types if needed
                "default": "./dist/index.cjs"
              }
            },
            "./package.json": "./package.json" // Allow access to package.json
            // Potentially other subpath exports: "./feature": { ... }
          },
          "files": ["dist"], // Files to include in the published package
          "scripts": {
            "build": "tsup src/index.ts --format esm,cjs --dts --clean",
            // ...
          },
          "publishConfig": { // Optional: for scoped packages or specific registry
            "access": "public"
          }
        }
        ```
*   **Build Tools:**
    *   For library development, use **tsup** (ESBuild-based, easy configuration), **unbuild** (Rollup-based), or directly **ESBuild** or **Rollup**. These support dual CJS/ESM builds and type definition generation.
    *   For application bundling, use **Vite**, **Webpack**, **Parcel**, **ESBuild**, or **Bun**.
    *   Control target environment (browser, Node.js version, etc.), format (ESM, CJS, UMD), source map generation, and minification in build settings.
*   **Type Definition File Generation and Distribution:**
    *   Set `"declaration": true` and `"declarationMap": true` (for source maps) in `tsconfig.json` to generate `.d.ts` files.
    *   Verify that the generated type definition files are correct and user-friendly (type definition testing is recommended).
    *   Specify the type definition entry point in `package.json`'s `types` or `exports` field.
*   **Monorepo Package Management:**
    *   Utilize **pnpm workspaces**, **yarn workspaces**, or **npm workspaces** to efficiently manage multiple related packages in a single repository.
    *   Combine with **Turborepo** or **Nx** to optimize workflows like building, testing, versioning, and publishing.
*   **Dependency Management:**
    *   `dependencies`: Packages required for the application/library to run.
    *   `devDependencies`: Packages needed only during development, building, or testing (e.g., TypeScript itself, Vitest, Biome, tsup).
    *   `peerDependencies`: Specifies packages that the library expects the host environment to provide (e.g., a React component library requires React). `peerDependenciesMeta` can specify optional peers.
    *   Include lock files (e.g., `pnpm-lock.yaml` from `pnpm`) in version control to pin dependency versions and ensure reproducibility.
*   **Versioning and Publishing:**
    *   Follow Semantic Versioning (SemVer) when publishing libraries.
    *   Use `npm publish` (or `pnpm publish`, `yarn publish`) to publish to the npm registry.
    *   Consider tools like `changesets` or `semantic-release` to automate the release process.
*   **Considerations for Tree Shaking:**
    *   When designing libraries, minimize side effects and structure code to be easily tree-shaken by bundlers relying on ESM static analysis. Set the `sideEffects` field in `package.json` appropriately.
    *   Avoid top-level immediately-executed code or dynamic additions to `prototype`.
*   **`.npmignore` / `package.json`'s `files` field:** Properly manage files included in and excluded from the published package to minimize its size (e.g., source files (`src/`), test files, config files (other than `tsconfig.build.json`) are usually excluded).

## 7. Linting, Formatting, and Toolchain

(Since major tools are covered in "1. Toolchain & Environment," this section focuses on their configuration, integration into workflows, and supplementary tools.)

*   **Biome Configuration and Usage:**
    *   Create `biome.json` in the project root and customize rules according to project requirements. Biome provides many recommended rules by default, but individual rules can be enabled/disabled as needed.
        ```json
        // biome.json example
        {
          "$schema": "https://biomejs.dev/schemas/1.7.0/schema.json", // Use latest schema version
          "organizeImports": { "enabled": true },
          "formatter": {
            "enabled": true,
            "formatWithErrors": false,
            "indentStyle": "space",
            "indentWidth": 2,
            "lineWidth": 80 // Or your preferred line width
          },
          "linter": {
            "enabled": true,
            "rules": {
              "recommended": true, // Start with recommended rules
              "suspicious": {
                "noExplicitAny": "warn" // Example: make 'any' a warning
              },
              "style": {
                "useNamingConvention": "error" // Enforce naming conventions
                // "noUselessElse": "error"
              },
              "correctness": {
                // "noUnusedVariables": "error" // Usually handled well
              }
            }
          },
          "javascript": { // Settings specific to JS/TS
            "formatter": {
              "quoteStyle": "double", // Or "single"
              "trailingComma": "all", // Or "es5"
              "semicolons": "always" // Or "asNeeded"
            }
          }
        }
        ```
    *   Install the Biome extension in editors like VS Code for auto-formatting on save and real-time lint warnings.
    *   Use `lint-staged` with `husky` (or `simple-git-hooks`) to set up commit hooks that run `biome check --apply-unsafe` (or `biome format --write` and `biome lint --apply-unsafe`) on staged files before committing (`--apply-unsafe` applies potentially destructive fixes; `--apply` is safer).
*   **TypeScript (`tsconfig.json`) Configuration:**
    *   Enable strict type-checking options in `compilerOptions` (e.g., `"strict": true`, `"noUnusedLocals": true`, `"noUnusedParameters": true`, `"noImplicitReturns": true`, `"exactOptionalPropertyTypes": true`).
    *   Configure `target` (output ECMAScript version) and `module` (module system) according to the project's runtime environment and bundler. For modern environments, targeting `ES2022` or `ESNext` and using `moduleResolution: "bundler"` (TS 5.0+), `NodeNext` (Node.js), or `ESNext` (browser bundlers) as the module resolution strategy is common.
    *   `baseUrl` and `paths` can define module path aliases (e.g., `@/*`), but build tools and test runners may also need corresponding configurations.
    *   `esModuleInterop: true` is recommended to improve interoperability with CommonJS modules.
    *   `skipLibCheck: true` skips type checking of declaration files in dependencies, speeding up builds, but risks missing type definition issues in dependencies.
    *   `verbatimModuleSyntax: true` (TS 5.0+): Makes the handling of type-only imports/exports (`import type`, `export type`) stricter, eliminating ambiguity. Replaces older `importsNotUsedAsValues` and `preserveValueImports`.
    *   `isolatedModules: true`: Ensures each file can be transpiled independently, enhancing compatibility with transpilers like Babel and esbuild.
*   **Editor Integration:**
    *   Utilize TypeScript language support, Biome integration, and debugging features in IDEs like VS Code (recommended) or WebStorm to their fullest.
    *   Place an `.editorconfig` file in the project root to unify basic editor settings like indent style and character encoding across the team.
*   **Node.js Version Management (Volta):**
    *   Volta pins Node.js and package manager versions in `package.json`'s `"volta"` field, automatically switching versions between projects.
        ```json
        // package.json
        "volta": {
          "node": "20.11.1", // Specify an LTS or current version
          "pnpm": "8.15.4"  // Specify pnpm version
        }
        ```
*   **Task Runners:**
    *   Define common tasks like build, test, and lint in `package.json`'s `scripts` field.
    *   For complex task sequences or cross-platform needs, consider build systems like `Nx` or tools like `zx` (Google's tool for writing shell scripts in JavaScript).
*   **Debugging:**
    *   Use browser developer tools or Node.js's inspector protocol (`--inspect` flag) for debugging. VS Code's debugger integrates well with these.
    *   Enable source maps (`"sourceMap": true` in `tsconfig.json` and build tool settings) to refer to the original TypeScript code during debugging.
*   **CI/CD Integration:** (See "5. Testing and CI/CD") Configure the CI pipeline to correctly execute these tools and act as quality gates.

## 8. Type System Usage

*   **Thoroughly Avoid `any`:** `any` completely disables type checking and should only be a last resort.
    *   **Utilize `unknown`:** For values whose type is genuinely unknown (e.g., responses from external APIs, `JSON.parse()` results), use `unknown`. Variables of type `unknown` cannot be operated on until their type is narrowed using type guards (e.g., `typeof value === 'string'`, `value instanceof MyClass`, custom type guard functions) or type assertions (`value as string` - only if safety is confirmed). This makes handling unknown values safer than `any`.
    *   **Generics:** Use type parameters (`<T>`) when creating reusable components or functions that can work with various types. Utilize constraints (`extends`) and default types.
*   **Strict Null Checks (`strictNullChecks`):**
    *   When enabled, all types are non-nullable by default. `null` or `undefined` must be explicitly included in a type using a Union (e.g., `string | null`).
    *   Use optional chaining (`?.`), nullish coalescing (`??`), the non-null assertion operator (`!`) - as a last resort, and type guards appropriately to handle potential `null` / `undefined` values.
*   **Union and Intersection Types:**
    *   **Union Types (`|`):** Indicate that a value can be one of several types (e.g., `string | number`). Use type guards to narrow down the type before use. Discriminated Unions are a particularly powerful pattern.
    *   **Intersection Types (`&`):** Combine multiple types into a new type that has all members of all constituent types (e.g., `TypeA & TypeB`). Useful for mixins or adding properties to existing types.
*   **Literal and Template Literal Types:**
    *   Define types that accept only specific literal values (e.g., `type Status = "success" | "error";`, `type HttpMethod = "GET" | "POST";`).
    *   Use template literal types (TS 4.1+) to represent string patterns more precisely (e.g., `type EventName = `${'user' | 'product'}_${'created' | 'updated'}`;`).
*   **`interface` vs. `type` Aliases:**
    *   Both can be used to define object shapes.
    *   `interface` supports declaration merging and is well-suited for `implements` clauses in classes. It had performance advantages in older TS versions in some cases.
    *   `type` aliases are more versatile and can be used for more complex type definitions, including Union types, Intersection types, tuple types, aliases for primitive types, and results of Mapped Types or Conditional Types.
    *   It's important to have a consistent policy within the team. A common convention is to use `interface` for object shapes and `type` for everything else (Unions, Intersections, primitive aliases, etc.), though `type` is increasingly favored for its greater flexibility.
*   **Utility Types:** Leverage TypeScript's built-in utility types (`Partial<T>`, `Required<T>`, `Readonly<T>`, `Pick<T, K>`, `Omit<T, K>`, `ReturnType<F>`, `Parameters<F>`, `Awaited<P>`, `NonNullable<T>`, etc.) to efficiently create new types from existing ones.
*   **Type Guard Functions:** Define functions with a return type predicate of the form `value is Type` to write custom logic for narrowing a variable's type within a specific block. Assertion functions (`asserts condition`) serve a similar purpose.
*   **Conditional Types:** Type-level ternary operators (`SomeType extends OtherType ? TrueType : FalseType`). Used for complex type manipulations and mappings. Can be combined with the `infer` keyword to extract types.
*   **Mapped Types:** Create new types based on the properties of existing types. Use the `in` keyword, key remapping (via `as` clause), and `+`/`-` modifiers (for adding/removing `readonly` or `optional`).
*   **`const` Assertions:** Use when you want the type system to infer the most specific literal type (e.g., `"hello"` instead of `string`) or to make objects and arrays recursively `readonly` (e.g., `const obj = { foo: "bar" } as const;`, `const arr = [1, 2] as const;`).
*   **`satisfies` Operator (TS 4.9+):** Verifies that an expression satisfies a certain type while preserving the specific inferred type of the expression. This allows for type safety while still utilizing more precise type information.
    ```typescript
    type Colors = "red" | "green" | "blue";
    // The keys of `palette` should be one of `Colors`,
    // but we want the type of each key's value to be inferred as its specific tuple type.
    const palette = {
      red:,
      green:,
      blue:,
      // yellow:, // Error if "yellow" is not in (Colors | string)
    } satisfies Record<Colors | string, [number, number, number]>; // Or just Record<string, ...>

    const redValue = palette.red; // Type is, not just [number, number, number]
    ```
*   **`readonly` Modifier and `Readonly<T>` / `ReadonlyArray<T>`:** Used to express immutable data structures at the type level, preventing reassignment of properties or array elements.
*   **Tuple Types:** Suitable for fixed-length arrays where each element can have a different type. Labeled tuple elements (TS 4.0+) improve readability. Rest elements (`...Type[]`) are also available.
*   **Index Signatures (`[key: string]: any`):** Allow objects to have arbitrary string keys but reduce type safety. Consider `Record<Keys, ValueType>` or Mapped Types for more specific keys if known.
*   **Branded Types (Nominal Types):** A technique to distinguish types that have the same structure but different semantic meanings. Usually achieved by intersecting with a unique property.
    ```typescript
    type UserId = string & { __brand: "UserId" };
    type ProductId = string & { __brand: "ProductId" };

    function processUserId(id: UserId) { /* ... */ }
    // const uid = "user-123" as UserId; // Type assertion needed for creation
    // processUserId("prod-456" as any as UserId); // Error if "prod-456" is passed without 'any'
    ```

## 9. Latest Language Features and Usage (TypeScript 5.5 & 5.4)

*   **TypeScript 5.5 (Key Features):**
    *   **Inferred Type Predicates:** TypeScript can now infer type predicates like `value is Type` in type guard functions without explicit return type annotations, slightly simplifying type guard authoring.
    *   **Isolated Declarations (`.d.ts` file) Support (`--isolatedDeclarations` flag):** Ongoing work to ease restrictions for generating type definition files (`.d.ts`) with single-file transpile tools (e.g., Babel, esbuild), enabling support for more TypeScript syntax.
    *   **`${configDir}` Placeholder in `tsconfig.json` (Introduced TS 5.4, improvements ongoing in 5.5):** A dynamic path reference within configuration files pointing to the directory containing that config file.
    *   **Improved Regular Expression Syntax Checking:** Stricter parsing and error reporting for regular expressions, aligned with ECMAScript specifications.
    *   **JSDoc Tooling and Inference Improvements:** Continuous enhancements to type inference from JSDoc comments in JavaScript files and related tooling support.
    *   **Type Support for `Promise.withResolvers`:** Type definitions for the new ECMAScript Promise creation utility.

*   **TypeScript 5.4 (Key Features):**
    *   **`NoInfer` Utility Type in Closures:** The `NoInfer<T>` utility type was introduced to suppress inference for specific type parameters during generic function calls. This helps prevent unintended type inferences and encourages more explicit type specifications.
        ```typescript
        function createProcessor<T>(processor: (data: T) => void, initialData: NoInfer<T>) { /* ... */ }
        // The type of initialData will not influence the inference of T
        ```
    *   **Support for `Object.groupBy` and `Map.groupBy`:** Type definitions for the ECMAScript Stage 3 proposals `Object.groupBy` and `Map.groupBy` were added (runtime support requires a corresponding JavaScript engine or polyfill).
    *   **Improved Narrowing for Assignments (Preserved Narrowing in Closures following Last Assignments):** After a variable is assigned within a conditional block, its type is now more accurately narrowed, and this narrowed type is preserved within closures.

*   **Recent Key Features (Highlights since TS 5.0, etc.):**
    *   **Decorators (Compliant with Stage 3 Proposal - TS 5.0):** The decorator syntax for adding metaprogramming behavior to classes and their members was revamped to align with the ECMAScript Stage 3 proposal. Not compatible with older experimental decorators (`experimentalDecorators`). Used in DI containers, ORMs, etc.
    *   **`const` Type Parameters (TS 5.0):** Adding a `const` modifier to generic type parameters allows the type system to infer type arguments as literal types or `readonly` whenever possible.
        ```typescript
        function getNames<const T extends readonly { name: string }[]>(args: T): T[number]["name"][] {
            return args.map(x => x.name);
        }
        const readonlyArgs = [{ name: "Alice" }, { name: "Bob" }] as const;
        const names = getNames(readonlyArgs); // type is ("Alice" | "Bob")[]
        ```
    *   **`satisfies` Operator (TS 4.9):** (See "8. Type System Usage")
    *   **`using` and `await using` Declarations (TS 5.2):** Supports the ECMAScript `Symbol.dispose` / `Symbol.asyncDispose` proposal for reliable resource disposal (RAII-style). Useful for managing file handles, database connections, etc.
        ```typescript
        const getResource = () => {
          return {
            [Symbol.dispose]: () => { console.log("Resource disposed!"); },
            doWork: () => console.log("Working...")
          };
        };
        function main() {
          using resource = getResource(); // Symbol.dispose is called when main exits
          resource.doWork();
        }
        ```
    *   **`export type *` (TS 5.0) and `export * as ns` (with `type` modifier support in TS 5.0):** More flexible type-only re-exports. Used with `verbatimModuleSyntax` for clear distinction between types and values.
    *   **Improved Exhaustiveness Checks for Union Enums (TS 5.0):** Better checking in `switch` statements that all members of a union type are covered.
    *   **`@satisfies` Support in JSDoc (TS 5.0):** Allows similar type checking as the `satisfies` operator within JSDoc comments in JavaScript files.
    *   **Configuration Option Improvements:** `moduleResolution: "bundler"` (TS 5.0) provides module resolution aligned with modern bundler behavior. Support for `extends` arrays in `tsconfig.json` for multiple configuration files (TS 5.0), etc.
    *   **Type Support for Iterator Helpers (TS 5.2+):** Type definitions for new iterator helper methods like `.map()`, `.filter()`.
    *   **Improved Type Inference for `Promise.race` and `Promise.any`.**

Leveraging these new features and improvements helps maximize the capabilities of TypeScript's type system, leading to safer and more maintainable code. Regularly review the official documentation and release notes to consider adopting new features appropriate for your project's needs.
