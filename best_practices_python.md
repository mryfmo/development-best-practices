### `best_practices_python.md`

```markdown
# Python Best Practices (Python 3.12 & 3.11)

## 1. Toolchain & Environment

To enhance efficiency, reproducibility, and maintainability in Python development, adopting a modern toolchain is essential. The emergence of `uv`, in particular, has significantly accelerated traditional package management workflows. Project management tools like `Rye` and `Hatch` provide a consistent experience centered around `pyproject.toml`. In AI system development, these tools contribute to managing large-scale dependencies and building reproducible experimental environments.

| Purpose                      | Recommended Tool                                     | Key Points & Rationale                                                                 |
| ------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Package & Virtual Environment Management** | **`uv`** (`uv venv`, `uv pip ...`)             | Built in Rust. A fast alternative to pip/virtualenv/pip-tools. Dependency resolution and installation are 10-100x faster. Supports lock file generation (`uv pip compile -o requirements.lock` or `uv lock`) and reproducible installs (`uv pip sync requirements.lock`). |
| **Project Management**        | **Rye** or **Hatch**                       | Manages project structure, dependencies, builds, and script execution based on `pyproject.toml` (PEP 621). Can use `uv` internally. Rye also manages Python installations. |
| **CI/CD Dependency Resolution**    | `uv pip sync --strict --no-build requirements.lock` | Based on the lock file, maximizes the use of wheel caches, significantly reducing setup time in CI. `--strict` enforces an exact match with the lock file.   |
| **Static Analysis (Lint & Format)** | **Ruff** (lint) + **Ruff Format**                | Built in Rust. Integrates functionalities of Flake8, isort, Black, etc., and is extremely fast. Configurable in `pyproject.toml` (`[tool.ruff]`).   |
| **Type Checking**              | **Pyright** (Recommended) or **Mypy** (`--strict`)        | Pyright offers excellent integration with VS Code (Pylance) and provides fast feedback. Mypy has a long history and is reputable for strict type checking.     |
| **Documentation**        | **Sphinx** (or **MkDocs** with Material)       | Standard for API documentation generation. Auto-generates from docstrings, supports various output formats.                                                              |
| **Task Runner**            | **Justfile** (`just ...`) or `hatch run ...`, `rye run ...` | Declaratively manages project-specific commands as an alternative to Makefiles.                             |
| **Security Vulnerability Scanning** | **`uv pip audit`** or **Bandit**, **Safety** | `uv pip audit` checks dependencies for vulnerabilities based on the PyPI Advisory Database. Bandit performs static security analysis of code. Safety also scans dependencies. |
| **Testing Framework**      | **pytest**                                     | Highly functional, extensible, and allows for concise test writing. Rich plugin ecosystem (e.g., `pytest-cov`, `pytest-asyncio`). |

## 2. Coding Style

Pythonic code is readable, understandable, and adheres to Python's philosophy. PEP 8 (Style Guide for Python Code) and PEP 257 (Docstring Conventions) serve as the foundation. Ruff enforces or assists with many of these conventions.

*   **PEP 8 Compliance and Ruff:**
    *   **Naming Conventions:**
        *   Module names: Short, all-lowercase `snake_case`.
        *   Package names: Short, all-lowercase. Underscores are discouraged.
        *   Class names: `CamelCase`.
        *   Type variables: `CamelCase`, preferably short (e.g., `T`, `KT`, `VT`).
        *   Exception names: `CamelCase`, ending with `Error`.
        *   Function names, method names, variable names: `snake_case`.
        *   Constants: Defined at the module level, `UPPER_SNAKE_CASE`.
        *   Indicating internal use: Single leading underscore (`_internal_var`).
        *   Avoiding name conflicts: Single trailing underscore (`class_`).
        *   Name mangling (private attributes within a class): Double leading underscore (`__private_attr`). Use sparingly.
    *   **Code Layout:**
        *   Indentation: 4 spaces. No tabs.
        *   Line length: Ruff Format's default is 88 characters. Adhere to project-agreed length (e.g., 100 characters; PEP 8 suggests 79).
        *   Line breaks: Prefer implicit line continuation inside parentheses, brackets, and braces. Avoid explicit line continuation with backslashes. Consider readability when breaking lines, e.g., before operators.
        *   Blank lines: Two blank lines between top-level function and class definitions. One blank line between method definitions in a class. Use single blank lines sparingly within functions to separate logical sections.
    *   **Imports:**
        *   Use `import module` or `from package import specific_module_or_function`.
        *   Avoid wildcard imports (`from module import *`) unless explicitly exposing an API via `__all__` or for internal APIs.
        *   Import order: Ruff auto-sorts (isort-compatible). Group imports in the order: standard library, third-party libraries, local application/library specific. Alphabetize within each group.
        *   Prefer absolute imports. Explicit relative imports (`from .module import name`) are acceptable only within packages.
*   **Comments:**
    *   **Docstrings (PEP 257):** Write for all public modules, functions, classes, and methods.
        *   Describe purpose, arguments, return values, potential exceptions, and usage examples if necessary.
        *   A one-line docstring should be concise. For multi-line docstrings, the first line is a summary, followed by a blank line, then a more detailed explanation.
        *   Adhering to prominent formats like Sphinx (reStructuredText), Google style, or NumPy style facilitates integration with documentation tools. Checkable with Ruff (`D` ruleset - pydocstyle).
        *   For AI systems, consider including model version, training data source, important assumptions, or constraints in docstrings.
    *   **Block Comments:** Multi-line comments. Each line starts with `#` and a space. Same indentation level as the code.
    *   **Inline Comments:** On the same line as a statement, separated by at least two spaces from the code. Format: `# comment`. Avoid obvious comments. Use sparingly to clarify the intent of code or complex logic.
    *   **TODO Comments:** Format as `# TODO(username): Description` or `# FIXME: Description` to indicate temporary work or areas needing correction.
*   **Expressions and Statements:**
    *   Avoid compound statements (multiple statements on one line, e.g., `if foo: bar; baz`). Use newlines instead.
    *   Distinguish `is` from `==`: `is` compares object identity (same memory address). `==` compares value equality. Always use `is None` or `is not None` for `None` comparisons.
    *   Boolean comparisons: `if foo:` instead of `if foo is True:`.
    *   Lambda expressions: Limit to small, simple cases. Define complex logic as regular functions using `def`.
*   **Pythonic Code:**
    *   Use EAFP (Easier to Ask for Forgiveness than Permission) and LBYL (Look Before You Leap) appropriately depending on the situation. EAFP (e.g., attribute access using `try-except`) is generally preferred in Python.
    *   Effectively use iterators and generators for memory-efficient code.
    *   Use list, dictionary, and set comprehensions appropriately.
    *   Consider using `__slots__` for classes with fixed instance attributes to reduce memory usage and speed up attribute access. However, be aware of limitations (e.g., no `__dict__`, care with multiple inheritance) and use judiciously.
*   **Type Hints:** (See "8. Type System Usage" for details) Type hints significantly improve readability and maintainability.

## 3. Security

*   **Dependency Management and Vulnerability Scanning:**
    *   Use virtual environments (`uv venv` or managed by Rye/Hatch) to isolate project dependencies.
    *   Regularly update dependencies and apply security patches (check with `uv pip list --outdated`).
    *   Install packages only from trusted sources (PyPI or approved indexes) and pin versions using lock files (`uv.lock`, `rye.lock`, `hatch.lock`, or hash-pinned `requirements.txt`) generated by `uv pip compile` or project managers, ensuring reproducible builds.
    *   Use tools like **`uv pip audit`**, **Safety**, or **Snyk** to scan dependencies for known vulnerabilities. Integrate this into CI.
*   **Secret Management:**
    *   Never hard-code secrets (passwords, API keys, tokens) in code or configuration files. Instead, use environment variables or dedicated secret management services (HashiCorp Vault, AWS Secrets Manager, Google Cloud Secret Manager, etc.).
    *   For development, `.env` files (loaded via libraries like `python-dotenv`) can be used, but for production, injection via environment variables is standard.
    *   Ensure secret values are not exposed in logs or error messages. Consider masking.
*   **Input Validation and Sanitization:**
    *   Treat all external input (user forms, API requests, CLI arguments, files, data from databases, etc.) as potentially malicious.
    *   Always validate and sanitize input. For database operations, use parameterized queries (placeholders) to prevent SQL injection. Never construct OS commands or SQL queries by concatenating untrusted data with strings.
    *   For web applications, use framework validation features or libraries like **Pydantic**, **Cerberus**, or **marshmallow** to enforce schemas and types, and to validate data.
*   **Avoid `eval()` and `exec()`:**
    *   Do not use Python's `eval()` or `exec()` on untrusted input. They can execute arbitrary code and are extremely dangerous.
    *   Use `ast.literal_eval` for evaluating literals of simple data types. If dynamic code execution is unavoidable, severely restrict the scope (e.g., pass `{"__builtins__": None}`), but generally, redesign to avoid it.
*   **Principle of Least Privilege:**
    *   Run your application with the minimum privileges required. For example, avoid running as root. Use non-root users in container environments by default.
    *   Restrict file system permissions for application files and credentials.
    *   Apply least privilege within your code. Avoid wildcard imports (`from module import *`) and minimize the use of `global` variables.
    *   When spawning subprocesses, avoid `shell=True` and pass commands and arguments as a list (`subprocess.run([...], check=True)`). Properly escape or validate inputs.
*   **Authentication and Authorization Security:**
    *   Do not store passwords in plaintext. Use strong hashing functions (bcrypt or Argon2) with appropriate salts and stretching (iteration counts).
    *   Utilize secure session management features of frameworks and set `HttpOnly`, `Secure`, and `SameSite=Lax` or `SameSite=Strict` attributes on cookies.
    *   Prefer industry-standard authentication protocols (OAuth2, OpenID Connect, JWT) over custom mechanisms. If using JWT, ensure proper algorithm selection, key management, and expiration.
    *   Enforce multi-factor authentication (MFA) for sensitive operations where applicable.
    *   Clearly define authorization logic; consider Role-Based Access Control (RBAC) or Attribute-Based Access Control (ABAC).
*   **HTTPS and Encryption:**
    *   Always transmit sensitive data over HTTPS (TLS 1.2 or higher, TLS 1.3 recommended). For Python clients like `requests` or `aiohttp`, ensure SSL verification is enabled by default and not disabled (`verify=False` should be avoided).
    *   Encrypt sensitive data at rest using proven libraries like **cryptography** (e.g., Fernet symmetric encryption). Key management is crucial.
    *   For web applications, apply security headers like HSTS (HTTP Strict Transport Security) (e.g., Django's security middleware, Flask-Talisman).
*   **Web Application Security (OWASP Top 10 Mitigation):**
    *   Leverage framework security features (CSRF protection, XSS mitigation, etc.). Escape user-generated content before HTML output using features like Django's auto-escaping or Jinja2's (Flask) `escape()`.
    *   Use Content Security Policy (CSP) headers to restrict loadable resources and mitigate XSS.
    *   Limit CORS (Cross-Origin Resource Sharing) to known origins if the API is not public.
    *   Be cautious of insecure deserialization; avoid loading pickle data from untrusted sources.
*   **Logging and Error Handling:**
    *   Implement robust logging but do not log sensitive information like credentials or personal data. Consider structured logging (e.g., JSON logs using `python-json-logger`).
    *   In production, do not expose internal stack traces or debug information to end-users. Return generic error messages. Log detailed errors server-side.
    *   Monitor for exceptions and unusual activity (potential signs of an attack) and set up alerts (e.g., Sentry, OpenTelemetry).
*   **Security Testing and Automation:**
    *   Integrate security checks into the CI/CD pipeline:
        *   **Bandit**: Static Application Security Testing (SAST) for Python code.
        *   **`uv pip audit` / Safety / Snyk**: Software Composition Analysis (SCA) for dependency vulnerabilities.
        *   Container image scanning (e.g., Trivy, Clair).
    *   Consider regular penetration testing and Dynamic Application Security Testing (DAST) tools.
    *   Mandate security checks in code reviews.
*   **Supply Chain and Environment:**
    *   Be aware of supply chain attacks. Verify the integrity of critical packages if possible (hashes or signatures, e.g., Sigstore/PyPI's Trusted Publishing). Use lock files to pin exact versions.
    *   Run Python applications in restricted environments in production (e.g., containers with minimal OS, Python slim images). Use sandboxing or AppArmor/SELinux profiles if applicable.
*   **AI System-Specific Security:**
    *   **Adversarial Attacks:** Attacks that cause a model to misclassify by adding minute perturbations to input data. Consider countermeasures such as adversarial training, input validation/transformation, and defensive distillation.
    *   **Data Poisoning:** Malicious data injected into training data to degrade model performance or embed backdoors. Mitigate with training data validation, anomaly detection, and trusted data sources.
    *   **Model Stealing/Extraction:** Attacks mimicking model behavior or extracting parameters via APIs. Consider API rate limiting, query obfuscation, differential privacy, and watermarking.
    *   **Privacy Breaches:** Models memorizing and leaking sensitive information from training data during inference. Mitigate with differential privacy, federated learning, and anonymization/removal of sensitive data.
    *   **Inference API Protection:** Implement authentication/authorization, rate limiting, input size limits, and detection of anomalous input patterns.
    *   **Secure Model Management:** Verify model file integrity (hashes/signatures), use secure storage, and implement access controls.

## 4. Performance Optimization

*   **Efficient Data Structures:** Use Python's built-in data structures understanding their characteristics.
    *   **`list`**: Ordered collections, fast iteration, and appends to the end. Inserting/deleting at the beginning of large lists is slow (O(N)); use `collections.deque` for efficient double-ended queues.
    *   **`dict` / `set`**: Fast lookups by key (average O(1) membership tests). Sets are much faster than lists for membership checks. Dictionaries preserve insertion order since Python 3.7+.
    *   **`tuple`**: Immutable fixed-length sequences. Slightly more memory-efficient and hashable for use as dict keys.
*   **Minimize Loops and Embrace Vectorization:** Python loops can be relatively slow for large datasets due to interpreter overhead.
    *   Whenever possible, use vectorized operations provided by libraries like **NumPy** and **Pandas** to offload heavy computations to C-level optimized code. This can yield orders-of-magnitude speedups for numerical computations.
    *   In AI/ML, tensor operations provided by frameworks like TensorFlow, PyTorch, and JAX are highly optimized.
*   **Avoid Unnecessary Work:**
    *   Cache results of expensive function calls using `functools.lru_cache` (memoization), especially for pure functions with repeated inputs.
    *   Use generator expressions (e.g., `(x*x for x in range(10))`) to produce items on demand instead of loading all elements into memory at once. This is memory-efficient and suitable for large datasets.
    *   Utilize early returns in conditional branches and short-circuit evaluation with `any()` and `all()`.
*   **Optimize Critical Loops:** When Python loops are necessary, reduce overhead inside the loop.
    *   Localize frequently used attribute lookups or method calls by binding them to local variables before the loop (e.g., `local_append = my_list.append`).
    *   Use built-in functions implemented in C like `sum()`, `any()`, `max()` instead of manual loops.
    *   For simple "loop and collect" tasks, use list comprehensions over `for` loops. They are more Pythonic and often slightly faster.
*   **Concurrency and Parallelism:**
    *   Due to Python's Global Interpreter Lock (GIL), threads within a single process do not speed up CPU-bound tasks but are effective for I/O-bound tasks.
    *   **I/O-bound tasks** (network requests, file operations): Use the `threading` module or `concurrent.futures.ThreadPoolExecutor` to overlap waiting times. Since Python 3.7, **`asyncio`** with `async/await` provides efficient management of numerous I/O-bound tasks. `asyncio.TaskGroup` (Python 3.11+) is useful for structured concurrency.
    *   **CPU-bound tasks**: Use the `multiprocessing` module, `concurrent.futures.ProcessPoolExecutor`, or `joblib` for parallel execution across multiple processes. Alternatively, leverage C-extension libraries like NumPy/Pandas. Experimental support for per-interpreter GILs (PEP 684) in Python 3.12 may offer future improvements in this area.
    *   AI systems often rely on libraries like TensorFlow/PyTorch that release the GIL or perform computations in C/C++/GPU.
*   **Memory Management and Object Allocation:**
    *   Avoid excessive object creation in inner loops. Reuse objects where possible.
    *   When building large strings from pieces, use `"".join(list_of_strings)` instead of string concatenation (`+`) in a loop.
    *   Explicitly `del` large objects that are no longer needed or let them go out of scope so the garbage collector can reclaim memory. Use profilers (`memory_profiler`) to monitor memory usage.
*   **Use Latest Python Version:** Upgrade to the latest Python 3.x release (e.g., Python 3.11 or 3.12) whenever possible. The "Faster CPython" project has brought significant performance improvements, especially since 3.11, often without requiring code changes.
*   **Profiling:** Before optimizing, use profiling tools like **cProfile**, **profile**, **Py-Spy**, **Yappi**, or **Scalene** to identify bottlenecks. "Measure, don't guess." Focus optimization efforts on the parts of the code where the program spends the most time (often a small fraction of the code).
*   **C Extensions and Alternative Implementations:** For computationally intensive tasks, consider moving critical sections to C/C++, **Cython**, or **Numba**. **PyPy** (a JIT-compiled Python interpreter) can improve performance for certain types of long-running computations. Rust extensions (via PyO3/Maturin) are also an option.
*   **Efficient I/O and Data Handling:**
    *   For file or network I/O, use buffering and efficient libraries. For large file reads, use chunked reading or the `io` module's buffered I/O.
    *   For binary data, use the `struct` module or NumPy (`numpy.fromfile`).
    *   For large datasets, use optimized libraries and formats like Pandas (CSV/Parquet), PyArrow (Arrow/Parquet format), or Polars.
    *   For inter-process communication where speed is critical, consider more efficient serialization formats than JSON (e.g., MessagePack, Protocol Buffers, Apache Arrow).
*   **Garbage Collector Tuning (Advanced):** In certain long-running applications (e.g., web services), the garbage collector (especially for cyclic garbage) can occasionally impact performance. The `gc` module allows tuning thresholds or temporarily disabling GC, but this is an advanced optimization and should only be done after confirming GC is an issue through measurement.

## 5. Testing and CI/CD

*   **Write Tests for All Critical Code:** Develop unit tests for functions and modules. Each test should verify one logical behavior and be isolated, fast, and deterministic. Aim for high coverage on core logic (business rules, algorithms, key data transformations in AI models).
*   **Testing Framework (pytest):** While Python's built-in `unittest` module is available, **pytest** is highly recommended for its conciseness, powerful fixture system, marking, and rich plugin ecosystem.
*   **Fast and Independent Tests:** Tests should not depend on each other's state. Use pytest fixtures or `unittest`'s `setUp/tearDown` (or `setUpClass/tearDownClass`) to ensure each test runs in a clean environment. Unit tests should run in milliseconds. Slow tests (e.g., involving database or external API access) should be minimized or moved to a separate integration test suite and marked (e.g., `@pytest.mark.slow`).
*   **Test Behavior, Not Implementation:** Write tests that assert expected outcomes or behaviors (outputs, state changes, side effects) rather than internal implementation details. This makes tests more robust to refactoring. Aim for one logical assert per test where possible. Use pytest's **parameterized tests** (`@pytest.mark.parametrize`) to easily run the same test logic against many inputs.
*   **Appropriate Use of Mocks and Stubs:** For unit tests of code with external dependencies (database calls, network requests, file I/O) or side effects, use mocks to isolate the system under test. Python's `unittest.mock` library (accessible via pytest's `mocker` fixture) or pytest's `monkeypatch` fixture can be used. However, use mocks judiciously, as over-mocking can make tests brittle. Designing for dependency injection (DI) can also facilitate testing by allowing test doubles (fake objects) to be substituted.
*   **Property-Based Testing:** For functions with complex logic or a wide range of inputs (common in AI data processing), consider property-based testing with libraries like **Hypothesis**. Instead of writing specific examples, you describe properties (invariants) that should hold for all inputs, and the library generates numerous random inputs to find counterexamples.
*   **Testing in AI Systems:**
    *   **Data Validation Tests:** Test data cleaning, preprocessing, and feature engineering logic.
    *   **Model Quality Tests:** For trained models, verify inference results against known input-output pairs (golden tests). Test that model performance metrics (accuracy, precision, recall, etc.) meet predefined thresholds.
    *   **Robustness Tests:** Test model behavior against noisy data or edge cases.
    *   **Bias and Fairness Tests:** Evaluate if model predictions exhibit unfair bias towards specific attributes.
*   **Continuous Integration (CI):**
    *   Set up a CI pipeline (GitHub Actions, GitLab CI, Jenkins, etc.) to run tests on every push or pull request.
    *   CI should, at a minimum, install dependencies (`uv pip sync ...`), run linters (Ruff) and static analyzers (Mypy, Bandit), and execute the test suite (`pytest`).
    *   Configure CI to run tests across relevant environments (multiple Python versions, multiple OSes). **Nox** or **Tox** can automate testing across multiple Python versions. `uv` can be used within these tools.
    *   Leverage `uv` in CI for faster dependency resolution and installation, and efficient caching.
*   **Test Coverage and Quality Gates:**
    *   Measure code coverage using **coverage.py** (integrates with pytest via `pytest-cov`). Aim for high coverage (e.g., 80%+) but, more importantly, ensure critical paths are covered.
    *   Set a minimum coverage threshold in CI to prevent adding code without tests.
    *   Run type checkers (Mypy, Pyright) in CI to maintain the correctness and consistency of type annotations.
*   **Continuous Deployment (CD) / Continuous Training (CT) / Continuous Monitoring (CM):**
    *   If applicable, automate deployment after tests pass (e.g., after merging to the main branch).
    *   Use Docker for consistent deployment environments. CD scripts should handle database migrations and rollouts to production/staging reproducibly. Include post-deployment sanity tests or health checks.
    *   For AI systems, consider CT (automating model retraining and evaluation) and CM (continuously monitoring model performance in production and detecting drift) in addition to CD.
*   **Documentation and Doctests:**
    *   Verify that code examples in docstrings or READMEs are correct by turning them into tests. Python's `doctest` module checks if code in docstrings produces the shown output. Pytest can also run doctests.
*   **CI for Code Quality:** In addition to tests, run linters (Ruff) and formatters (Ruff Format `--check`) in CI to enforce style before code merges. Also, run security linters (Bandit, or Ruff's Bandit rules) and type checkers (Mypy/Pyright).

## 6. Module Structure & Packaging

*   **Project Layout:** Organize your Python project with a clean, logical structure. The **"src layout"** is a common approach, placing your package(s) inside a top-level `src/` directory.
    ```
    project-root/
    ├── pyproject.toml
    ├── README.md
    ├── src/
    │   └── mypackage/
    │       ├── __init__.py
    │       ├── core.py
    │       └── utils.py
    ├── tests/
    │   └── test_core.py
    └── uv.lock (or requirements.lock, etc.)
    ```
    The src layout ensures that tests and applications use the installed package, not a local module of the same name, preventing import confusion.
*   **Modules and Imports:** Divide your code into modules (Python files) by responsibility or feature. If a single file grows too large, consider splitting it into a package (a directory with an `__init__.py` and multiple sub-modules). Use absolute imports within your package (e.g., `from mypackage.core import something`). PEP 8 recommends absolute imports. Use explicit relative imports (`from . import submodule`) only for closely related modules within the same package.
*   **Encapsulation and API Design:** Treat the internal module structure as an implementation detail and define a clear public API. Indicate private code using a single leading underscore (`_private_function`). For libraries, consider a flat interface in `__init__.py` by importing key classes/functions, or define `__all__` to control wildcard import behavior.
*   **Packaging with `pyproject.toml`:**
    *   Use the modern Python packaging standard via **`pyproject.toml`** (PEP 518/PEP 621). This file declares build system requirements and project metadata in a `[project]` table.
    *   Tools like **Rye** and **Hatch** manage projects centered around `pyproject.toml`, simplifying build, test, and publishing workflows. `uv` also supports `pyproject.toml`.
    *   Specify package name, version, description, authors, Python version requirements, dependencies, and entry points (console scripts) in `pyproject.toml`.
        ```toml
        [project]
        name = "mypackage"
        version = "0.1.0"
        description = "AI system utilities"
        authors = [{name = "Your Name", email = "you@example.com"}]
        requires-python = ">=3.9" # Minimum Python version supported by the project
        dependencies = [
            "numpy>=1.23", # Prefer specific version constraints
            "requests>=2.28,<3.0" # Upper bounds if known breaking changes
        ]
        # Optional dependencies (e.g., for development, testing)
        [project.optional-dependencies]
        dev = ["ruff", "pytest", "mypy"]
        docs = ["sphinx"]

        [project.scripts]
        mytool = "mypackage.cli:main"
        ```
*   **Virtual Environments and Dependency Locking:**
    *   Always develop and deploy within a virtual environment. Create and manage environments using **`uv venv`**. Rye and Hatch automate virtual environment management.
    *   Lock dependencies using a lock file (e.g., `uv.lock` (uv's native format), `requirements.lock` (output of `uv pip compile`), `hatch.lock`, `rye.lock`) to ensure reproducible environments. Install dependencies from this lock file in CI/CD and production (`uv pip sync`).
    *   Separate core dependencies from development/test dependencies using optional dependency groups in `pyproject.toml`.
*   **Documentation and Metadata:**
    *   Include a `README.md` at the project root describing the project, usage examples, and installation instructions. An open-source project must also include a `LICENSE` file.
    *   Properly describe classifiers and project URLs (homepage, documentation, repository) in the `[project]` table of `pyproject.toml`.
*   **Entry Points and CLI:** If your package provides a command-line tool, define entry points in `pyproject.toml`'s `[project.scripts]` (or `[project.gui-scripts]`) for easy execution.
*   **Tests and Packaging:** Test code (e.g., in the `tests/` directory) is usually not included in the distributed package unless intended for end-user execution. When using the src layout, tests require the package to be installed (e.g., `uv pip install -e .[dev]` for editable install with dev dependencies).
*   **Large Projects and Modularization:** For very large codebases or AI systems with multiple components, consider a monorepo with multiple packages (e.g., using Rye or Hatch workspace features) or namespace packages. Alternatively, maintain a single package with clearly separated subpackages (e.g., `mypackage.data`, `mypackage.model`).
*   **Resource Files:** If your package needs to include data files, model files, etc., use `importlib.resources` (Python 3.9+ offers improved APIs for direct file system paths) or `importlib_resources` (backport for older Python versions) for access. Ensure these files are included in the package distribution by specifying them in `pyproject.toml` (via build backend settings, e.g., Hatch's `sources` or `force-include` in `[tool.hatch.build.targets.wheel]`, or setuptools' `package_data`/`include_package_data` with `MANIFEST.in`).

## 7. Linting, Formatting, and Toolchain

*   **Integrated Linter & Formatter (Ruff):**
    *   Use **Ruff** for code error detection (linting) and style unification (formatting). Ruff, built in Rust, provides functionalities of many tools like Flake8, isort, PyUpgrade, Bandit, and Pylint, with high speed.
    *   Manage rulesets (e.g., `lint.select = ["E", "F", "W", "I", "UP", "ANN", "D"]`) and formatter settings (e.g., `format.line-length = 88`) centrally in the `[tool.ruff]` section of `pyproject.toml`. Ruff also covers some Bandit rules.
    *   Integrate with editors (e.g., VS Code Ruff extension), `pre-commit` hooks, and CI to detect and fix issues early.
*   **Type Checkers (Mypy / Pyright):**
    *   Use **Mypy** or **Pyright** to verify static type hints. Pyright integrates smoothly and quickly with VS Code (Pylance extension).
    *   Manage configurations in `pyproject.toml` (e.g., `[tool.mypy]` or `[tool.pyright]`) and run in CI to detect type errors. Strongly recommend enabling `--strict` mode or individual strict flags (e.g., Mypy's `disallow_untyped_defs = True`).
*   **`pre-commit` Hooks:**
    *   Use the **pre-commit** framework to automate quality checks like Ruff (lint/format) and Mypy/Pyright before commits are made. This maintains code quality standards in the repository and reduces CI failures.
*   **Python Interpreter and Version Management:**
    *   Use the latest CPython interpreter for development. Explicitly state the supported Python version range for your project in `pyproject.toml`'s `requires-python`.
    *   Use **Nox** or **Tox** to automate testing across multiple Python versions if needed. Rye and Hatch also support multi-version management.
*   **Build and Deployment Toolchain:**
    *   For pure Python packages, `uv`, Rye, and Hatch support building and uploading to PyPI (often using Twine internally).
    *   For packages with C extensions, consider using **cibuildwheel** to build wheels for multiple platforms. For Rust extensions, use **Maturin**.
    *   For deploying web applications, use proven application servers like **Gunicorn** (WSGI) or **Uvicorn** (ASGI).
*   **Development Environment and Jupyter Notebooks:**
    *   If using Jupyter Notebooks, use the same kernel as the project's virtual environment to maintain consistency.
    *   Format Notebook code with Ruff. If versioning notebooks, consider using tools like nbstripout to remove output cells and execution counts. Refactor notebook logic into regular Python scripts or modules when moving to production code. Tools like Jupytext can facilitate two-way synchronization between notebooks and Python scripts.
*   **Continuous Tool Updates:** Regularly update development tools (Ruff, Mypy, uv, Rye/Hatch, etc.) to benefit from the latest features, bug fixes, and support for new Python syntax.
*   **Containerization:**
    *   If deploying AI systems via Docker/Kubernetes, follow Dockerfile best practices.
    *   Use multi-stage builds to include only necessary dependencies in the final image. Choose lightweight base images like `python:3.X-slim`.
    *   Build Docker images in CI and perform vulnerability scans (e.g., Trivy).
    *   Consider using distroless images where possible to further reduce the attack surface.

## 8. Type System Usage (Type Hints in Python)

*   **Proactive Adoption of Type Hints:** Add type annotations to function parameters, return values, class attributes, and variables to make code self-documenting, catch bugs early, and improve IDE support. E.g., `def process(data: list[str]) -> dict[str, int]: ...`.
*   **Modern Type Syntax:**
    *   Python 3.9+: Use built-in collection types as generics directly, e.g., `list[int]` (no need for `typing.List`).
    *   Python 3.10+: Use the `|` operator for Union types, e.g., `str | None` (instead of `typing.Optional[str]` or `typing.Union[str, None]`).
    *   Python 3.12+: PEP 695 introduced a more concise syntax for generic class and function type parameter declarations (e.g., `class MyList[T]: ...`, `def get_item[T](collection: list[T], index: int) -> T: ...`, `type Point[T] = tuple[T, T]`).
*   **Leverage Static Type Checkers:** Integrate **Mypy** or **Pyright** into the development process and CI to verify the correctness of type annotations. Treat type errors like lint errors and fix them. Gradual typing is possible, starting with key modules or classes. Use `# type: ignore` comments sparingly and document the reason.
*   **Generics (`typing.TypeVar`, PEP 695):** Use generics to maintain type information when writing functions or classes that operate汎用的に (generically) on various types. Utilize `TypeVar`'s `bound` and `constraints` as well.
*   **Protocols and Structural Typing (`typing.Protocol`):** Use protocols (PEP 544) to define interfaces that accept objects with specific methods or attributes without requiring a common base class (static duck typing). `@runtime_checkable` can be used for runtime checks, but protocols are primarily for static analysis.
*   **Avoid `Any`; Consider `object`:** `Any` disables type checking, so avoid it whenever possible. If a function truly accepts any type, use `object`. For unknown external data, clarify types using type guards or runtime validation libraries like Pydantic, in conjunction with type hints.
*   **`Optional` and `Union` Types:** Use `Optional[X]` (or `X | None`) for values that can be `None`. Use `Union[X, Y, Z]` (or `X | Y | Z`) for values that can be one of several types. Type checkers enforce proper handling of these cases.
*   **Dataclasses (`dataclasses`) and `TypedDict`:**
    *   Use the **`@dataclass`** decorator (Python 3.7+) for classes that primarily store data. It auto-generates `__init__`, `__repr__`, `__eq__`, etc., based on type hints. Options like `slots=True` (Python 3.10+) and `frozen=True` are also useful.
    *   Use **`TypedDict`** (PEP 589) to define types for dictionaries with specific string keys and value types, such as JSON objects. `total=False` allows for optional keys.
*   **Type Aliases (`typing.TypeAlias` and PEP 695 `type` statement):** Use `TypeAlias` (Python 3.10+ or `typing_extensions`) or the more concise `type Point = tuple[float, float]` syntax (Python 3.12+) to give readable names to complex type annotations.
*   **Type Guards and Narrowing:** Use checks like `isinstance()`, `callable()`, `hasattr()`, or literal value comparisons to narrow down the type of variables from Union types or `Any` to more specific types. Custom type guard functions (`typing.TypeGuard` - Python 3.10+) can also be created.
*   **Literal Types (`typing.Literal`):** Define types that accept only specific literal values, e.g., `Literal["read", "write"]`.
*   **`typing.Self`:** (Python 3.11+) Use the `Self` type to indicate that a class method returns an instance of that same class (useful in builder patterns, etc.).
*   **`typing.override` (Python 3.12+):** A decorator to explicitly mark that a method is intended to override a method from a superclass, helping type checkers verify the override's correctness.
*   **`typing.Unpack` and `TypeVarTuple` (Python 3.11+):** (PEP 646) Used for more precise typing of variadic arguments (e.g., `*args: Unpack[tuple[int, str]]`) and for generic typing of tuple structures like NumPy's `shape` (e.g., `Shape = TypeVarTuple('Shape'); Array[Unpack[Shape]]`).
*   **`@dataclass_transform` (Python 3.11+, `typing_extensions`):** (PEP 681) Allows ORM libraries and similar tools to inform type checkers about methods (like `__init__`) that are dynamically generated by decorators or factory functions.
*   **Third-Party Library Type Stubs:** For third-party libraries without inline type information, use stub files (`.pyi`) from the Typeshed project or create custom ones. Many popular libraries now ship with type hints or provide them via Typeshed.
*   **Runtime Type Checking (e.g., Pydantic):** While static type hints are generally ignored at runtime, for handling external data (API responses, config files, etc.), using libraries like Pydantic for runtime data validation and type conversion is highly effective. This significantly reduces runtime errors due to unexpected data formats or types.

## 9. Latest Language Features and Usage (Python 3.12 & 3.11)

*   **Python 3.12:**
    *   **PEP 695 (Type Parameter Syntax):** Introduced a more concise and explicit syntax for type parameters in generic classes, functions, and type aliases.
        ```python
        # Old way (Python < 3.12)
        from typing import TypeVar
        T = TypeVar('T')
        class MyListOld(list[T]): ...
        def first_old(l: list[T]) -> T: return l

        # New way (Python 3.12+)
        class MyList[T](list[T]): ...
        def first[T](l: list[T]) -> T: return l
        type Point[V] = tuple[V, V] # Generic type alias
        ```
    *   **PEP 701 (Syntactic Formalization of f-strings):** The f-string parser became stricter, clarifying some ambiguous cases. F-strings can now be more easily reused and support more flexible expressions (nested quotes, comments, backslashes).
    *   **Per-Interpreter GIL (PEP 684 - Experimental):** While not fully implemented, this PEP paves the way for future improvements in CPU-bound parallelism in specific cases by having a GIL per sub-interpreter. Python 3.12 includes initial support. Usage like `threading.Thread(target=..., daemon=True, subinterpreter=True)` is being explored as an alternative to `multiprocessing.Process`.
    *   **`pathlib.Path.walk()`:** A directory tree traversal feature similar to `os.walk()` was added to `pathlib`, enabling more object-oriented file system operations.
    *   **`typing.override` decorator:** Explicitly marks that a method is intended to override a superclass method, allowing type checkers to verify this.
    *   **Improved Linux `perf` profiler support:** Provides more detailed profiling information for Python functions.

*   **Python 3.11:**
    *   **Significant Performance Improvements ("Faster CPython"):** Averaged 25% (up to 10-60%) faster than Python 3.10. Improvements primarily in startup time, function calls, and certain operations, thanks to the Specializing Adaptive Interpreter.
    *   **Exception Groups and `except*` (PEP 654):** New syntax to raise and handle multiple unrelated exceptions simultaneously, used by `asyncio.TaskGroup`, for example.
    *   **TOML Standard Library Support (`tomllib`):** Reading TOML (Tom's Obvious, Minimal Language) files is now possible with the standard library (useful for parsing `pyproject.toml`, etc.).
    *   **`typing.Self` (PEP 673):** A type annotation to indicate that a class method returns an instance of that same class.
    *   **`typing.LiteralString` (PEP 675):** A type to indicate that a function expects only literal strings, helping prevent SQL injection and similar vulnerabilities.
    *   **`typing.TypeVarTuple` and `typing.Unpack` (PEP 646):** Generics for variadic type arguments, allowing more precise typing of tuple structures like NumPy's `shape`.
    *   **Atomic grouping `(?>...)` and possessive quantifiers `*+`, `++`, `?+`, `{m,n}+` in `re`:** Help suppress backtracking in the regex engine, improving performance for certain patterns and helping prevent ReDoS attacks.
    *   **Task Groups in `asyncio` (PEP 654):** `asyncio.TaskGroup` makes it easier to structure and await multiple asynchronous tasks, with improved exception handling.
    *   **Zero Cost Exceptions (PEP 678):** The runtime overhead of `try` blocks is now nearly zero unless an exception is actually raised.

*   **Modern Features Common to Both Versions:**
    *   **Structural Pattern Matching (`match`/`case` - Python 3.10+):** Allows for more declarative ways to write complex conditional logic. Continuously improved in 3.11 and 3.12, with powerful guard clauses and name binding.
    *   **f-strings:** Recommended as the standard way for string formatting.
    *   **`async`/`await`:** Fundamental for asynchronous programming, used with the `asyncio` library.
    *   **Type Hints:** Python's type system is continuously evolving, with newer versions adding more expressive and user-friendly features.

Leveraging these features enables writing more readable, maintainable, and performant Python code. Maximize the use of available features according to your project's Python version requirements.
```
