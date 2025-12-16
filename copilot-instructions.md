# Gauntlet Security Platform - Copilot Instructions

## Architecture Overview

Gauntlet is a **cloud security posture management (CSPM)** platform with multiple interconnected components:

| Component | Language | Purpose |
|-----------|----------|---------|
| `Gauntlet-backend/` | Python | Main CSPM CLI with AWS/Azure/GCP modules |
| `Gauntlet-frontend/` | React + Vite | Dashboard UI using AWS Amplify auth |
| `Gauntlet-SBOM/` | Python + FastAPI | Software Bill of Materials scanner with AI agents |
| `Gauntlet-SCA/` | Python | Software Composition Analysis and secret scanning |
| `gauntlet-spectra/` | Rust | High-performance concurrent secret scanner |
| `Gauntlet-Emails/` | TypeScript + React Email | Email notification service |
| `Gauntlet-DB/` | Python + SQLAlchemy | Shared database models (PostgreSQL) |

---

## Cross-Component Dependencies

**Gauntlet-DB is the shared foundation** - Install it before other Python packages:
```bash
uv pip install git+https://github.com/gauntlet-security/Gauntlet-DB.git
```

Database import pattern:
```python
from db.models import AWSCredentials, CloudCredentials, ExceptionInfo
from db.utils import get_db_session
```

---

## Code Standards by Language

### Python (Backend, SBOM, SCA, DB)

**Tooling:** Ruff (linting) + Bandit (security) via pre-commit

#### Formatting Rules
- **Line length:** 99 characters
- **Indentation:** 4 spaces
- Blank line **before** `if`, `for`, `while`, `try` blocks
- **No** blank line before `else`, `elif`, `except`, `finally`

```python
# ✅ CORRECT
def process_data(items):
    result = []

    for item in items:
        processed = transform(item)

        if processed.is_valid:
            result.append(processed)
        else:
            logger.warning("Invalid item")

    return result
```

#### Constants & Variables
- Constants at file top after imports: `UPPER_SNAKE_CASE`
- Variables: `snake_case` - must be meaningful (no single chars, no numbers)
- Classes: `PascalCase`

```python
# Constants at top
MAX_RETRY_ATTEMPTS = 3
API_BASE_URL = "https://api.example.com"

# Good variable names
user_count = len(users)          # ✅
total_price = sum(prices)        # ✅
t = sum(prices)                  # ❌ single char
price2 = adjusted_price          # ❌ number in name
```

#### Function Naming
Use verb-first descriptive names:
- `get_*` / `fetch_*` - Retrieve data
- `create_*` / `build_*` - Create objects
- `validate_*` / `check_*` - Validation
- `is_*` / `has_*` / `can_*` - Boolean checks

```python
def validate_user_credentials(username: str, password: str) -> bool:
    """Validate user login credentials against the database."""
    ...
```

#### Docstrings (Required)
```python
def calculate_total(items: List[Item]) -> float:
    """
    Calculate the total price for all items.

    Args:
        items: List of items with price and quantity.

    Returns:
        Total price as float.

    Raises:
        ValueError: If items list is empty.
    """
```

#### Logging
```python
from logzero import logger
logger.info("Processing started")
logger.exception(f"Error: {e}")  # Auto-captures traceback
```

---

### JavaScript/TypeScript (Frontend, Emails)

**Tooling:** ESLint + Prettier

#### Formatting Rules
- **Indentation:** 2 spaces
- **Quotes:** Single quotes for strings
- **Semicolons:** Required
- **Line length:** 120 characters

#### Naming Conventions
| Element | Convention | Example |
|---------|------------|---------|
| Variables/Functions | `camelCase` | `getUserData`, `isLoading` |
| Constants | `UPPER_SNAKE_CASE` | `API_BASE_URL`, `MAX_RETRIES` |
| Components | `PascalCase` | `UserProfile`, `DataTable` |
| Files (components) | `PascalCase.jsx` | `UserProfile.jsx` |
| Files (utils) | `camelCase.js` | `axios.js`, `helpers.js` |

#### React Patterns
```javascript
// ✅ CORRECT - Hooks at top, early returns
const UserProfile = ({ userId }) => {
  const { data, isLoading, error } = useSWR(`/users/${userId}`, fetcher);

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;

  return <div>{data.name}</div>;
};
```

#### SWR Data Fetching Pattern
```javascript
// Read operations - useSWR
export const get_company_info = () => {
  const { data, isLoading, error, mutate } = useSWR('/company-info', fetcher);
  return { data, isLoading, error, mutate };
};

// Write operations - useSWRMutation
export const useAddException = () => {
  return useSWRMutation('/exceptions/add', addException, {
    onSuccess: () => { mutate_results(); toast.success('Added'); },
    onError: (error) => { toast.error(error); }
  });
};
```

#### Axios Headers
Custom headers for cloud service context:
```javascript
const config = {
  headers: {
    'X-Cloud-Service': cloud_service,
    'X-Account-ID': account_id
  }
};
```

---

### Rust (gauntlet-spectra)

**Tooling:** Clippy (pedantic + nursery lints enabled)

#### Linting Configuration
```toml
# Cargo.toml - strict lints enabled
[lints.clippy]
pedantic = { level = "warn", priority = -1 }
nursery = { level = "warn", priority = -1 }
unwrap_used = "warn"
expect_used = "warn"
```

#### Naming Conventions
| Element | Convention | Example |
|---------|------------|---------|
| Variables/Functions | `snake_case` | `process_file`, `scan_result` |
| Types/Structs/Enums | `PascalCase` | `ScanResult`, `ConfigOptions` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_THREADS`, `DEFAULT_TIMEOUT` |
| Modules | `snake_case` | `scan_utils`, `global_config` |

#### Error Handling
Use `Result` and `?` operator, avoid `.unwrap()`:
```rust
// ✅ CORRECT
fn read_config(path: &Path) -> Result<Config, ConfigError> {
    let content = fs::read_to_string(path)?;
    let config: Config = serde_yaml::from_str(&content)?;
    Ok(config)
}

// ❌ AVOID
let config = fs::read_to_string(path).unwrap();
```

#### Documentation
```rust
/// Scans a repository for secrets.
///
/// # Arguments
/// * `path` - Path to the repository
/// * `config` - Scanner configuration
///
/// # Returns
/// Vector of findings or error
pub fn scan_repository(path: &Path, config: &Config) -> Result<Vec<Finding>> {
    // ...
}
```

---

## Cloud Security Check Pattern

AWS/Azure/GCP checks use a class-based pattern in `{Provider}_Checks/`:

```python
class S3:
    def __init__(self, session, exception_obj, company_name):
        self.session = session
        self.s3_client = self.session.client("s3")
        self.s3_resp = {"pass": [], "fail": [], "cannot_scan": [], "fail_count": {}}
    
    def S3_Checks(self, err_code):  # Main entry - dispatches to specific check
        func_call = getattr(self, err_code)
        passed, failed, cannot_scan, faulty_resource_list = func_call()
    
    def S3_G1(self):  # Check naming: {SERVICE}_{CODE}
        # Returns [passed, failed, cannot_scan, faulty_list]
```

### Response Structure
```python
passed = [["Region", "PASS", "-", "Check description", "N/A"]]
failed = [["Region", "FAIL", "Details", "Check description", "severity"]]
cannot_scan = [["Region", "CANNOT SCAN", "Access Denied", "Check description", "severity"]]
```

### Error Handling
```python
ERR_LIST = ["AccessDenied", "UnauthorizedOperation", "AccessDeniedException"]

try:
    response = self.client.some_operation()
except botocore.exceptions.ClientError as error:
    if error.response["Error"]["Code"] in ERR_LIST:
        cannot_scan.append([...])
    else:
        logger.exception(error)
```

---

## Development Workflows

### Python Projects
```bash
uv pip install -e ".[dev]"   # Install with dev dependencies
pre-commit install           # Setup git hooks
pytest                       # Run tests
```

### Frontend
```bash
npm install && npm start     # Dev server on port 3000
npm run test                 # Vitest
npm run lint:fix             # ESLint + Prettier
```

### Rust
```bash
cargo build --release
cargo test
cargo clippy                 # Strict linting
```

---

## DRY Principle (All Languages)

Extract repeated code into reusable functions:

```python
# ✅ CORRECT - Reusable function
def format_api_response(data, status, message):
    return {"status": status, "message": message, "data": data}

def get_user(user_id):
    return format_api_response(fetch_user(user_id), "success", "User retrieved")
```

```javascript
// ✅ CORRECT - Reusable hook
const useApiMutation = (url, mutator, successMsg) => {
  return useSWRMutation(url, mutator, {
    onSuccess: () => toast.success(successMsg),
    onError: (e) => toast.error(e)
  });
};
```

---

## Key Environment Variables

| Variable | Component | Purpose |
|----------|-----------|---------|
| `API_KEY` | SBOM/SCA | FastAPI authentication |
| `GOOGLE_API_KEY` | SBOM | AI agent (Gemini) |
| `G_DEBUG` | All Python | Debug mode toggle |
| `VITE_APP_BASE_URL` | Frontend | API base URL |

---

## Testing

| Component | Command | Framework |
|-----------|---------|-----------|
| Frontend | `npm test` | Vitest |
| Python | `pytest` | pytest-cov |
| Rust | `cargo test` | Built-in |

---

## File Patterns

- `__version__.py` - Version at package root (Python)
- `pyproject.toml` - Python config with ruff/bandit
- `.pre-commit-config.yaml` - Git hooks
- `sample.py` - Usage examples
- `Cargo.toml` - Rust config with clippy lints
