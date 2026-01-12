# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Development Philosophy

### KISS (Keep It Simple, Stupid)

Simplicity should be a key goal in design. Choose straightforward solutions over complex ones whenever possible. Simple solutions are easier to understand, maintain, and debug.

### YAGNI (You Aren't Gonna Need It)

Avoid building functionality on speculation. Implement features only when they are needed, not when you anticipate they might be useful in the future.

### Design Principles

- **Dependency Inversion**: High-level modules should not depend on low-level modules. Both should depend on abstractions.
- **Open/Closed Principle**: Software entities should be open for extension but closed for modification.
- **Single Responsibility**: Each function, class, and module should have one clear purpose.
- **Fail Fast**: Check for potential errors early and raise exceptions immediately when issues occur.

## 🧱 Code Structure & Modularity

### File and Function Limits

- **Never create a file longer than 500 lines of code**. If approaching this limit, refactor by splitting into modules.
- **Functions should be under 50 lines** with a single, clear responsibility.
- **Classes should be under 100 lines** and represent a single concept or entity.
- **Organize code into clearly separated modules**, grouped by feature or responsibility.
- **Line length must be max 120 characters** (see `pyproject.toml` Black config)
- **Use .venv** (the Python 3.12 virtual environment) whenever executing Python commands, including for unit tests.

### Project Architecture

ClickContracts is a real estate digital signature and contract management platform with a microservices architecture.

### Repository Structure

**⚠️ IMPORTANT: `click-webapp` is NOT a repository - it's a parent folder containing multiple independent Git repositories.**

The project consists of **9 separate GitHub repositories**, each independently versioned and deployed:

#### All Repositories (9 total)

| Repository | Description | Frontend Integration |
|------------|-------------|---------------------|
| `click-frontend` | React application (React 18, TailwindCSS, CRA) | N/A |
| `click-backend-auth` | Authentication and user management | `REACT_APP_AUTH_API` |
| `click-backend-subscription` | Subscription and billing (Stripe) | `REACT_APP_SUBSCRIPTION_API` |
| `click-backend-reporting` | Analytics and reporting | `REACT_APP_REPORTING_API` |
| `click-backend-admin` | Administrative functions | `REACT_APP_ADMIN_API` |
| `click-backend-mls` | Multiple Listing Service integrations | `REACT_APP_MLS_API` |
| `click-backend-settings` | User settings, contacts, roles, onboarding | `REACT_APP_SETTINGS_API` |
| `click-backend-contracts` | Document creation, signing, envelope management | `REACT_APP_CONTRACTS_API` |
| `click-backend-transactions` | Real estate transaction management, SkySlope sync | `REACT_APP_TRANSACTIONS_API` |

#### Frontend Repository

**`click-frontend`** - Main React application
- React 18 with TailwindCSS
- Create React App (CRA)
- Central API layer: `src/utils/lambdas.js`
- Environment configs: `environments/`

#### Backend Service Repositories

All backend services follow the same architecture (AWS Lambda + SAM):

1. **`click-backend-auth`** - Authentication and user management
   - User login and registration
   - Password management and reset
   - Session management
   - User verification and onboarding

2. **`click-backend-subscription`** - Subscription and billing management
   - Stripe payment integration
   - Plan management and pricing
   - Subscription lifecycle management

3. **`click-backend-reporting`** - Analytics and reporting
   - Dashboard data and analytics
   - Report generation

4. **`click-backend-admin`** - Administrative functions
   - System administration and monitoring
   - File operations and S3 management
   - Error tracking (Sentry webhook integration)
   - Administrative utilities

5. **`click-backend-mls`** - Multiple Listing Service integrations
   - Real estate listing data retrieval
   - MLS provider integrations
   - Property search and data sync
   - Agent and office information

6. **`click-backend-settings`** - User settings, contacts, roles, onboarding
   - User preferences and configuration
   - Contact management
   - Role and permission management
   - Onboarding workflows
   - Integrations portal (SkySlope, Follow Up Boss, etc.)
   - **📖 See: `docs/integrations/README.md`** for integrations documentation

7. **`click-backend-contracts`** - Document and envelope management
   - Document creation and signing
   - Envelope management
   - Transaction processing
   - AI-powered document recognition (ai-pdf-classification)

8. **`click-backend-transactions`** - Real estate transaction management
   - Transaction CRUD operations
   - Kanban-style dashboard with stages
   - Checklist templates (BUY/LIS)
   - SkySlope integration for external sync
   - **📖 See: `docs/transactions/README.md`** for complete documentation

### Supporting Directories

These directories exist within specific repositories:
- `email-templates/` - HTML email templates
- `document-templates/` - PDF contract templates
- `form-templates/` - Form configurations
- `digital-signature-pdf/` - PDF signature coordinate mappings
- `text-classifier/` - ML-based document classification
- `api-collections/` - Thunder Client API test collections

### Common Characteristics (All Backend Repos)

- AWS Lambda + SAM architecture
- Uses same shared database (MySQL)
- References same Lambda layers (hostedLayers, sharedLayers)
- Shares S3 buckets and infrastructure
- Python 3.12 virtual environment
- Same `.env` configuration approach
- Follows same coding standards (this CLAUDE.md)
- Independent deployment pipelines via AWS CodeBuild

### Development Workflow

1. Navigate to the specific repository folder within `click-webapp/`
2. Each repo has its own `.git` directory
3. Set up local environment with `.env` file
4. Activate Python 3.12 virtual environment (backend) or use npm (frontend)
5. Follow coding standards from this CLAUDE.md
6. Test locally using SAM or direct Python execution
7. Deploy via AWS CodeBuild pipeline

## Development Commands

### Frontend (Main App)
```bash
cd frontend/
npm install
npm start                    # Local development (uses .env.local)
npm run start:dev           # Development environment
npm run build               # Production build
npm test                    # Run tests
npm run format              # Format with Prettier
```

Notes
- Node.js 18+ recommended (see `frontend/readme.md` for exact versions)
- CRA uses environment files under `frontend/environments/` loaded by `env-cmd`

### Python Components
```bash
# IMPORTANT: Always activate Python 3.12 virtual environment before running Python code
# Python 3.12 virtual environment is located at the project root
cd D:\code\click\click-webapp
.\.venv\Scripts\activate  # On Windows - Python 3.12
# source .venv/bin/activate  # On Linux/Mac - Python 3.12

# Verify Python 3.12 is active
python --version  # Should show Python 3.12.x

# Formatting (uses Black with 120 char line length, configured for Python 3.12)
black --line-length 120 <file>

# Dependencies managed via requirements.txt (updated for Python 3.12 compatibility)
pip install -r requirements.txt
```

**Note**: ALL Python code execution must be done within the activated virtual environment (`.venv`). This ensures proper dependency management and avoids module import errors. The project has been upgraded to Python 3.12.

## Architecture Overview

### Lambda Function Structure
Each backend service follows a consistent pattern:
- `lambdas/<function_name>/app.py` - Main handler function
- `layers/` - Shared utilities and database operations
- `environments/serverless.yaml` - AWS deployment configuration
- `buildspec.yaml` - CodeBuild deployment pipeline

### Folder structure
lambdas/<function_name>/
    app.py  - Main file that contains the lambda handler. If the function is short this will be the only file needed.\
    layer_init.py  - fake file with layer functions (ignored on deplyments - see more below)
    additional_module_1 - additional modules when we want to separate functions from the main file
    tests/  - for when creating tests
        test_app.py
        test_additional_module_1.py

### Common Lambda Patterns
```python
def lambda_handler(event, context):
    try:
        result = main_function(event)
        return layer_init.response_success(result)
    except Exception as exception:
        message = ['Error: Description', str(exception)]
        return layer_init.response_fail(message)
```

### Frontend Architecture
- React Router for navigation
- Sentry integration for error tracking and performance monitoring
- Environment-based configuration via `.env` files
- TailwindCSS for styling
- React Hook Form with Zod validation
 - Intercom widget support (via env variables)

#### API Communication Layer: `lambdas.js`

**Location**: `frontend/src/utils/lambdas.js`

**Purpose**: Central API communication layer that provides a unified interface between the React frontend and AWS Lambda microservices. This file abstracts away the complexity of HTTP requests to multiple backend services.

**Key Features**:
- **Service Routing**: Maps service names to environment-specific API endpoints
- **Token Management**: Handles authenticated vs non-authenticated requests
- **Error Handling**: Centralized error processing with Sentry integration
- **Request Logging**: Comprehensive logging for debugging API calls
- **Version Control**: Handles application version mismatches

**Service Mapping**:
```javascript
// Maps service names to environment variables
'auth' → process.env.REACT_APP_AUTH_API
'contracts' → process.env.REACT_APP_CONTRACTS_API
'subscription' → process.env.REACT_APP_SUBSCRIPTION_API
'settings' → process.env.REACT_APP_SETTINGS_API
'organization' → process.env.REACT_APP_ORGANIZATION_API
'reporting' → process.env.REACT_APP_REPORTING_API
'admin' → process.env.REACT_APP_ADMIN_API
'mlsData' → process.env.REACT_APP_MLS_API
'transactions' → process.env.REACT_APP_TRANSACTIONS_API
'clickAI' → process.env.REACT_APP_CLICK_AI_API  // AI document recognition (not yet integrated)
```


**Usage Pattern**:
```javascript
import lambdas from '../utils/lambdas'

// Non-authenticated call
lambdas.verifyUserEmail(email, (response, success) => {
  if (success) {
    // Handle success
  }
})

// Authenticated call
lambdas.getEnvelopeDetails(idToken, userId, envelopeId, (response, success) => {
  if (success) {
    // Handle success
  }
})
```

**Function Categories**:
- **Auth Services**: Login, registration, password management, user verification
- **Contract Services**: Document creation, signing, envelope management
- **Subscription Services**: Payment processing, plan management, Stripe operations
- **Settings Services**: User preferences, forms, categories, contacts management
- **Reporting Services**: Analytics and dashboard data
- **MLS Services**: Real estate listing integration
- **Admin Services**: File operations, system administration
- **Transaction Services**: Real estate transaction management

**Error Handling**:
- Automatic Sentry error reporting (excludes 0, 401, 500 status codes)
- CORS error detection and user-friendly messages
- Version mismatch detection (status 299)
- Detailed console logging for debugging

Backend Sentry webhooks
- `backend-admin` provides Sentry webhook handlers that send alerts to Slack via SQS

**Important Notes**:
- All functions use callback pattern: `(response, success) => {}`
- Functions are organized by microservice (auth, contracts, etc.)
- Automatically includes version headers and authentication tokens
- Handles URL construction to prevent double slashes

### Key Integrations
- AWS Lambda (compute)
- AWS S3 (document storage)
- Stripe (payments)
- Sentry (monitoring)
- Slack (Sentry alerts routing)
- Intercom (user support widget)
- PDF processing and digital signatures
- MLS data integration
- Email templates via SES
- OpenAI (AI document recognition embeddings)

### Database Operations

**STANDARD: Use `layer_init.database_connect()` with manual connection management for all database connections.** Always ensure connections are properly closed.

#### Default Method (Standard)
```python
connection = layer_init.database_connect()
cursor = connection.cursor()

try:
    # SELECT operations
    cursor.execute("SELECT firstName, lastName, email FROM click.users WHERE id = %s", (user_id,))
    user_data = cursor.fetchone()

    # Multiple rows
    cursor.execute("SELECT * FROM click.users WHERE organizationId = %s", (org_id,))
    users = cursor.fetchall()

    # UPDATE operations
    cursor.execute(
        "UPDATE click.users SET firstName = %s, lastName = %s, email = %s WHERE id = %s",
        ("John", "Doe", "john.doe@example.com", user_id)
    )
    connection.commit()
finally:
    cursor.close()
    connection.close()  # ⚠️ CRITICAL: Always close connections!
```

#### Alternative Method (DatabaseOperationsHandler)

Use `DatabaseOperationsHandler` **only when explicitly requested**. This method provides automatic resource management via context manager:

```python
from layer_init import DatabaseOperationsHandler

def your_function():
    with DatabaseOperationsHandler() as handler:
        query = "SELECT firstName, lastName, email FROM click.users WHERE id = %s"
        handler.cursor.execute(query, (user_id,))
        user_data = handler.cursor.fetchone()

        update_query = "UPDATE click.users SET firstName = %s WHERE id = %s"
        handler.cursor.execute(update_query, ("John", user_id))
        handler.db_connection.commit()
        # Connection automatically closed when block exits
```

#### ❌ Prohibited Patterns

**NEVER USE these deprecated helper methods (regardless of connection method):**
```python
# ❌ PROHIBITED - These methods hide SQL queries and are deprecated
user_name = handler.get_field(...)       # Use direct SELECT instead
user_data = handler.get_fields(...)      # Use direct SELECT instead
handler.update_fields(...)               # Use direct UPDATE instead
```

#### Key Guidelines
- **Default**: Use `layer_init.database_connect()` with try/finally for cleanup
- **Alternative**: Use `DatabaseOperationsHandler` only when explicitly requested
- **Always**: Use direct SQL queries (no deprecated helper methods)
- **Always**: Use parameterized queries to prevent SQL injection
- **Critical**: Always close connections in a `finally` block when using manual connection management

### Local Development Environment

There are two distinct approaches for local development:

#### 1. Raw Local Execution (Direct Python)
For running Python files directly outside AWS Lambda environment:

**Setup:**
- Environment variables: Root-level `.env` file (contains DB credentials, API keys, etc.)
- "Fake" layer_init: Each function directory contains a local `layer_init.py` that mimics the AWS shared layer functionality
- Direct imports: Functions import from local `layer_init.py` instead of AWS layers

**Usage:**
```bash
# Environment variables loaded from .env file
python your_lambda_function/app.py
```

**Key Files:**
- `.env` - Root-level environment variables (DB connections, API keys)
- `local-tests/layer_init.py` - Example fake layer with database_connect() and utility functions
- Individual function `layer_init.py` files - Local implementations of shared utilities

#### 2. AWS SAM Local Debugging
For testing in AWS Lambda-like environment locally using VS Code workspace:

**Setup:**
- Configuration: `workspace.code-workspace` with SAM launch configurations
- Templates: Environment-specific YAML files (local.yaml, demo.yaml, production.yaml)
- Docker: AWS SAM CLI runs functions in Docker containers that simulate Lambda runtime

**Configuration Files:**
- `backend-*/environments/local.yaml` - Local SAM template (dev environment)
- `backend-*/environments/demo.yaml` - Demo environment template  
- `backend-*/environments/production.yaml` - Production environment template

Runtime  
- All Python lambdas target Python 3.12 (upgraded from 3.11, see templates)

**Usage:**
```bash
# Via VS Code debugger using workspace configurations
# Or via SAM CLI:
sam local start-api --template local.yaml
sam local invoke functionName --template local.yaml
```

**VS Code Debugging Example:**
```json
{
  "type": "aws-sam",
  "request": "direct-invoke",
  "name": "Stripe Operations",
  "invokeTarget": {
    "target": "template",
    "templatePath": "backend-subscription/environments/local.yaml",
    "logicalId": "stripeOperations"
  }
}
```

**Note**: The subscription service configuration shown above is for reference only. Active subscription development should use the separate `click-backend-subscription` repository with its own workspace configuration.

#### Environment Configuration Summary
- **Raw Local**: `.env` file + fake layer_init files
- **SAM Local**: `local.yaml` + Docker + AWS credentials for real environment testing
- **Development**: `environments/.env.development` (frontend) + AWS dev environment
- **Production**: Environment variables in AWS Lambda + AWS Secrets Manager

## 🤖 AI PDF Classification Service

### Overview

**Location**: `ai-pdf-classification/`

The AI PDF Classification service is a separate microservice for intelligent document recognition within user-uploaded PDFs. It uses OpenAI embeddings to identify reference documents (contracts, forms, disclosures) by creating "fingerprints" and performing similarity matching.

**Key Capabilities:**
- Generate searchable fingerprints for reference documents (42+ documents)
- Search user PDFs for known reference documents
- Dual search modes: Fast (TF-IDF) vs Accurate (OpenAI embeddings)
- Parallel processing for multiple PDFs and documents
- Page range detection for multi-page matches

### Architecture

The service consists of two Lambda functions:

#### 1. Fingerprint Generator (`fingerprintGenerator`)
Creates searchable fingerprints for reference documents stored in S3.

**Function**: `ai-pdf-classification/lambdas/fingerprint_generator/app.py`

**Purpose:**
- Downloads reference PDFs from S3 based on database metadata
- Extracts text using PyPDF2
- Generates 1536-dimensional embeddings via OpenAI text-embedding-3-small
- Extracts unique phrases and keywords for pre-filtering
- Stores fingerprints in S3 (`click-ai-fingerprints-{environment}`)

**Fingerprint Structure:**
```json
{
  "s3base_key": "addendum-purchase-agreement-mn",
  "document_name": "Addendum to Purchase Agreement (MN)",
  "document_url": "https://click-template-documents-development.s3.us-east-1.amazonaws.com/...",
  "page_count": 1,
  "first_page_text": "...",
  "first_page_embedding": [0.123, -0.456, ...],  // 1536 dimensions
  "embedding_dimension": 1536,
  "embedding_provider": "openai",
  "unique_phrases": ["ADDENDUM TO PURCHASE AGREEMENT", "Minnesota Association"],
  "title_keywords": ["addendum", "purchase", "agreement"]
}
```

**Local Usage:**
```bash
cd ai-pdf-classification/lambdas/fingerprint_generator

# Generate fingerprints for AI documents (42 documents from form templates)
python app.py ai

# Force regenerate all AI documents
python app.py ai_force

# Generate for single document
python app.py single addendum-purchase-agreement-mn
```

**Lambda Configuration:**
- Memory: 2048 MB
- Timeout: 300s (5 minutes)
- Runtime: Python 3.12
- Layers: hostedLayers, hostedLayerAI (new), sharedLayers

#### 2. Document Finder (`documentFinder`)
Searches for reference documents within user-uploaded PDFs.

**Function**: `ai-pdf-classification/lambdas/document_finder/app.py`

**Features:**
- **Parallel PDF Processing**: Process multiple user PDFs simultaneously (up to 5x speedup)
- **Parallel Document Search**: Search for multiple documents concurrently within each PDF
- **Smart Keyword Pre-filtering**: Reduces candidates by 60-80% before similarity matching
- **Dual Search Modes**: Fast (keyword + TF-IDF) vs Accurate (keyword + OpenAI embeddings)
- **Confidence Scoring**: Configurable thresholds (0.75 accurate, 0.6 fast)

**API Request (Single PDF - Backward Compatible):**
```json
{
  "pdf_url": "https://click-ai-classification-input-pdfs.s3.us-east-1.amazonaws.com/document.pdf",
  "search_list": ["addendum-purchase-agreement-mn", "closing-worksheet-mn"],
  "mode": "accurate"
}
```

**API Request (Multiple PDFs - Parallel Processing):**
```json
{
  "pdf_urls": [
    "https://click-ai-classification-input-pdfs.s3.us-east-1.amazonaws.com/document1.pdf",
    "https://click-ai-classification-input-pdfs.s3.us-east-1.amazonaws.com/document2.pdf"
  ],
  "search_list": ["location-map-mn", "seller-refusal-condition-report-wi"],
  "mode": "accurate"
}
```

**API Response:**
```json
[
  {
    "url": "https://click-ai-classification-input-pdfs.s3.us-east-1.amazonaws.com/document1.pdf",
    "click-documents": [
      {
        "s3baseKey": "location-map-mn",
        "start-page": 4,
        "end-page": 4
      }
    ]
  }
]
```

**Local Usage:**
```bash
cd ai-pdf-classification/lambdas/document_finder

# Run with default test PDFs
python app.py

# Fast mode (TF-IDF similarity)
python app.py fast

# Accurate mode (OpenAI embeddings - default)
python app.py accurate
```

**Lambda Configuration:**
- Memory: 2048 MB
- Timeout: 300s (5 minutes)
- Runtime: Python 3.12
- Layers: hostedLayers, hostedLayerAI (new), sharedLayers

### Performance

**Fingerprint Generation:**
- ~2-3 seconds per document
- $0.02 per 1M tokens (~$0.00002 per page)
- 42 documents currently fingerprinted

**Document Search (Parallel Processing):**
| Scenario | Sequential | Parallel | Speedup |
|----------|-----------|----------|---------|
| 2 PDFs, 3 documents | ~40s | ~11-15s | **3x faster** |
| 5 PDFs, 5 documents | ~100s | ~20-30s | **4x faster** |
| 10 PDFs, 10 documents | ~200s | ~30-40s | **5x faster** |

### Configuration

**Environment Variables (in serverless.yaml):**
```yaml
# S3 Configuration
FINGERPRINTS_BUCKET: click-ai-fingerprints-${CurrentEnvironment}

# Performance Settings
CONFIDENCE_THRESHOLD: "0.75"  # Accurate mode threshold
KEYWORD_FILTER_THRESHOLD: "0.6"  # Fast mode threshold
MAX_PDF_WORKERS: "5"  # Parallel PDF processing (max: 10)
MAX_CONCURRENT_SEARCHES: "10"  # Parallel document search (max: 20)
MAX_EMBEDDING_WORKERS: "5"  # OpenAI embedding workers

# OpenAI Configuration
OPENAI_API_KEY: "{{resolve:secretsmanager:OPENAI_API_KEY:SecretString:api_key}}"
OPENAI_EMBEDDING_MODEL: "text-embedding-3-small"
```

### S3 Bucket Structure

```
click-template-documents-{environment}/
└── real-estate-agents/state/{STATE}/{s3baseKey}-{version}.pdf

click-ai-fingerprints-{environment}/
├── fingerprints_cache.json  # All fingerprints combined
└── individual/{s3baseKey}.json  # Individual fingerprints

click-ai-classification-input-pdfs/
└── {user-uploaded-pdfs}.pdf  # User documents to search
```

### Dependencies

**Core Python Packages (~65 MB total):**
- `PyPDF2>=3.0.1` - PDF text extraction
- `scikit-learn>=1.3.0` - TF-IDF for fast mode
- `numpy>=1.24.0` - Vector operations
- `boto3>=1.34.0` - AWS SDK
- `pymysql>=1.1.0` - Database connectivity
- `openai>=1.0.0` - OpenAI embeddings (~1 MB)
- `requests>=2.31.0` - HTTP utilities
- `python-dotenv>=1.0.0` - Environment management

**Lambda Layers:**
- `hostedLayers` - Standard layers (pymysql, layer_init) - shared with other services
- `hostedLayerAI` - **NEW** AI-specific layer (PyPDF2, scikit-learn, numpy, openai) - only used by AI service
- `sharedLayers` - Common utilities - shared with other services

### Development Workflow

**Uses same .venv as main project (Python 3.12):**
```bash
# Activate project virtual environment
cd D:\code\click\click-webapp
.\.venv\Scripts\activate

# Run AI lambdas locally
cd ai-pdf-classification/lambdas/fingerprint_generator
python app.py ai

cd ai-pdf-classification/lambdas/document_finder
python app.py
```

**Environment Configuration:**
- Root `.env` file contains all environment variables (DB, S3, OpenAI)
- Each lambda has local `layer_init.py` for development (ignored in deployments via `.gitignore`)
- SAM template: `ai-pdf-classification/environments/serverless.yaml`

### Deployment

**Deployment Pipeline:**
- Uses AWS CodeBuild via `buildspec.yaml`
- Stack name: `click-ai`
- Region: `us-east-1`
- Capabilities: `CAPABILITY_IAM`

**Build Process:**
```bash
sam build --template-file environments/serverless.yaml --build-dir build-output
sam package --s3-bucket click-ai-${CURRENT_ENVIRONMENT} --output-template-file package-template.yml
sam deploy --stack-name click-ai --capabilities CAPABILITY_IAM
```

### Cost Analysis

**Very cost-effective:**
- OpenAI embeddings: $0.02 per 1M tokens (~$0.00001 per page)
- Lambda execution: ~$0.00011 per search
- **Total for 1,000 searches/month**: ~$0.16

### Frontend Integration (Future)

**Status**: Not yet integrated in frontend

**Planned Integration:**
1. Add `REACT_APP_CLICK_AI_API` to environment variables
2. Add service mapping in `frontend/src/utils/lambdas.js`:
   ```javascript
   'clickAI' → process.env.REACT_APP_CLICK_AI_API
   ```
3. Create lambda functions:
   - `lambdas.findDocuments(idToken, userId, pdfUrls, searchList, mode, callback)`
   - `lambdas.generateFingerprints(idToken, userId, updateType, callback)`

### Important Notes

- **Separate Microservice**: Independent deployment from main backend services
- **Shared Infrastructure**: Uses same database, S3 buckets, and Lambda layers (except hostedLayerAI)
- **Python 3.12**: Uses project's shared `.venv` for local development
- **OpenAI Only**: Simplified from multi-provider system for Lambda compatibility (65 MB vs 1.1 GB)
- **S3 Storage**: All fingerprints stored in S3, no database tables for fingerprint data
- **Dynamic Document List**: Fetches AI document IDs from `click.forms` table dynamically

### Error Handling

**Common Issues:**
1. Empty fingerprints → Run `python app.py ai_force` to regenerate
2. Document not found → Verify s3baseKey matches database exactly
3. S3 access denied → Check AWS credentials configuration
4. OpenAI quota exceeded → Check billing at platform.openai.com

### Reference Documentation

**See**: `ai-pdf-classification/README.md` for comprehensive documentation including:
- Detailed architecture diagrams
- Performance benchmarks
- Troubleshooting guide
- Testing procedures
- Future enhancements

## 📋 Style & Conventions

### Python Style Guide

- **Follow PEP8** with these specific choices:
  - Use double quotes for strings
  - Use trailing commas in multi-line structures
- **Always use type hints** for function signatures and class attributes
- **Format with `Black`**

### Docstring Standards

Use Google-style docstrings for all public functions, classes, and modules:

```python
def calculate_discount(
    price: Decimal,
    discount_percent: float,
    min_amount: Decimal = Decimal("0.01")
) -> Decimal:
    """
    Calculate the discounted price for a product.

    Args:
        price: Original price of the product
        discount_percent: Discount percentage (0-100)
        min_amount: Minimum allowed final price

    Returns:
        Final price after applying discount

    Raises:
        ValueError: If discount_percent is not between 0 and 100
        ValueError: If final price would be below min_amount

    Example:
        >>> calculate_discount(Decimal("100"), 20)
        Decimal('80.00')
    """
```

### Naming Conventions Python

- **Variables and functions**: `snake_case`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Private attributes/methods**: `_leading_underscore`
- **Type aliases**: `PascalCase`
- **Enum values**: `UPPER_SNAKE_CASE`

### Naming Conventions JavaScript and React

- **Variables and functions**: `camelCase`
- **React components**: `PascalCase`

## Testing and Deployment

### Testing
- Frontend: React Testing Library via `npm test`
- Backend: Python unit tests (patterns vary by service)
- API testing via Thunder Client collections in `api-collections/`

## 🧪 Testing Strategy Python

### Test-Driven Development (TDD)

1. **Write the test first** - Define expected behavior before implementation
2. **Watch it fail** - Ensure the test actually tests something
3. **Write minimal code** - Just enough to make the test pass
4. **Refactor** - Improve code while keeping tests green
5. **Repeat** - One test at a time

### Testing Best Practices

```python
# Always use pytest fixtures for setup
import pytest
from datetime import datetime

@pytest.fixture
def sample_user():
    """Provide a sample user for testing."""
    return User(
        id=123,
        name="Test User",
        email="test@example.com",
        created_at=datetime.now()
    )

# Use descriptive test names
def test_user_can_update_email_when_valid(sample_user):
    """Test that users can update their email with valid input."""
    new_email = "newemail@example.com"
    sample_user.update_email(new_email)
    assert sample_user.email == new_email

# Test edge cases and error conditions
def test_user_update_email_fails_with_invalid_format(sample_user):
    """Test that invalid email formats are rejected."""
    with pytest.raises(ValidationError) as exc_info:
        sample_user.update_email("not-an-email")
    assert "Invalid email format" in str(exc_info.value)
```

### Test Organization

- Unit tests: Test individual functions/methods in isolation
- Integration tests: Test component interactions
- End-to-end tests: Test complete user workflows
- Keep test files next to the code they test
- Use `conftest.py` for shared fixtures
- Aim for 80%+ code coverage, but focus on critical paths

### Deployment
- Frontend: AWS CodeBuild via `buildspec.yml`
- Backend: Serverless (SAM templates under `environments/serverless.yaml`)
- Each service has independent deployment pipeline

## Code Conventions

### Python
- Black formatting (120 character line limit)
- Layer-based architecture for shared utilities
- Consistent error handling patterns
- Environment-specific configurations

### JavaScript/React
- Prettier formatting
- Functional components with hooks
- TailwindCSS utility classes
- Environment variables:
  - CRA app: `REACT_APP_*` (via `frontend/environments/*.env` and `env-cmd`)

### File Organization
- Lambda functions: One function per directory under `lambdas/`
- Shared code: Organized in `layers/` directories
- Configuration: `environments/` for deployment configs
- Assets: Organized by type (templates, signatures, etc.)

## 🔄 Git Workflow

### ⚠️ CRITICAL GIT RULES - NON-NEGOTIABLE

**🚫 ABSOLUTE PROHIBITIONS:**
- **NEVER COMMIT DIRECTLY TO MASTER** - Always work on branches
- **NEVER PUSH DIRECTLY TO MASTER** - Use pull request workflow
- **NEVER INCLUDE CLAUDE ATTRIBUTION** - Never add "Co-Authored-By: Claude" or similar
- **NEVER BYPASS CODE REVIEW** - Even for critical fixes, use proper workflow

### Branch Strategy

- `master` - Production-ready code (protected - no direct commits)
- `feature/*` - New features
- `fix/*` - Bug fixes (including critical production fixes)
- `docs/*` - Documentation updates
- `refactor/*` - Code refactoring
- `test/*` - Test additions or fixes

### Proper Workflow

1. **Create branch**: `git checkout -b fix/description-of-fix`
2. **Make changes**: Edit files and commit to branch
3. **Push branch**: `git push origin fix/description-of-fix`
4. **Create PR**: Use web interface to create pull request
5. **Get review**: Wait for approval (even for critical fixes)
6. **Merge**: Only after PR approval

### Commit Message Format

**PROHIBITED:** Never include Claude attribution in commit messages

```
<type>(<scope>): <subject>

<body>

<footer>
```
Types: feat, fix, docs, style, refactor, test, chore

**✅ CORRECT Example:**
```
fix(subscription): resolve blueprint syntax error

- Remove incomplete else blocks in blueprint_builder.py
- Fix Lambda initialization failure
- Restore plan change functionality
```

**❌ PROHIBITED Examples:**
```
# NEVER do this:
fix: something Co-Authored-By: Claude <noreply@anthropic.com>
fix: claude code generated fix
feat: written by claude code
```

## 📋 Transactions Feature

**Repository**: `click-backend-transactions` (independent Git repository)

The Transactions feature provides real estate transaction management with SkySlope integration for external sync.

**📖 See: `docs/transactions/README.md`** for complete documentation including:
- Full architecture and data flow diagrams
- All Lambda functions and API operations
- Database schema (`click.sales`, `click.accountChecklist`)
- Frontend components and pages
- SkySlope integration details (authentication, data mapping, sync workflow)
- Error handling and troubleshooting
- Development and deployment guides

**Quick Reference:**

| Component | Location |
|-----------|----------|
| Backend Lambda handlers | `click-backend-transactions/lambdas/` |
| Frontend pages | `click-frontend/src/pages/transactions/` |
| Frontend components | `click-frontend/src/components/transactions/` |
| API functions | `click-frontend/src/utils/lambdas.js` (service: 'transactions') |
| SkySlope client | `click-backend-transactions/lambdas/transactions_handler/skyslope_client.py` |

**Key Operations:**
- Transaction CRUD: `create_transaction`, `get_transactions`, `update_transaction`, `delete_transaction`
- SkySlope sync: `sync_to_skyslope`, `get_skyslope_sync_status`, `sync_documents_to_skyslope`
- Checklists: `create_checklist`, `get_account_checklist`, `update_checklist`

## 💳 Stripe Integration

**⚠️ REPOSITORY NOTE**: The active subscription and Stripe integration code is in the separate **`click-backend-subscription`** GitHub repository. The `backend-subscription/` directory in this repository is legacy code.

### Payment Methods

**IMPORTANT**: When working with Stripe payment integration, payment method creation, customer setup, or signup payment flows, always refer to the comprehensive documentation:

**📖 See: `docs/stripe_integration/payment_methods.md`** (in this monorepo for reference)
**📖 Active Code: `click-backend-subscription` repository** (for implementation work)

This documentation contains:
- Modern Payment Element implementation details
- Two distinct flows (new customer vs existing customer)
- Payment method attachment process
- Error handling patterns
- Free plan and trial plan considerations
- Code examples with file references and line numbers

**Key Points for LLMs:**
- We use **Payment Element** (modern) NOT SetupIntent (legacy)
- Two flows: PaymentIntent with setup_future_usage (signup) vs direct PaymentMethod creation (existing customers)
- Customer creation and payment method attachment happen together in backend
- Free plans bypass payment collection entirely
- Trial plans offer optional payment method addition with incentives

### Database Tables and APIs

**IMPORTANT**: Always use the correct database tables for Stripe product/price data:

#### ✅ **Modern Tables (USE THESE):**
- `click.stripePrices` - Contains current pricing information
- `click.stripeProducts` - Contains current product information

#### ❌ **Deprecated Tables (DO NOT USE):**
- `click.price` - Legacy pricing table (deprecated, no longer read by frontend)
- `click.product` - Legacy product table (deprecated, no longer read by frontend)

**Note**: The Stripe webhook handler (`stripe_webhook_handler`) still writes to both legacy and modern tables for backward compatibility, but no frontend code reads from the legacy tables.

#### **Frontend API Usage:**

```javascript
// Use for getting products from current stripePrices/stripeProducts tables
lambdas.getStripeProducts('get_products', (response, success) => {
  if (success) {
    // Products from modern database tables
    const products = Array.isArray(response) ? response : response.products
  }
})
```

#### **Implementation Notes:**
- Always use `lambdas.getStripeProducts('get_products')` for fetching plan data
- Filter results by `product.industry === 'real-estate-agents'` for real estate plans
- Filter by `product.productType === 'tier_plan'` for subscription plans (not add-ons)
- Basic plans typically have `hierarchy === 1` or `price === 0`
- The deprecated `lambdas.getIndustryPrices()` function is no longer used and can be removed

## 🛡️ Security Best Practices

### Security Guidelines

- Never commit secrets - use environment variables / AWS Secrets Manager
- Validate all user input and sanitize outputs
- Use parameterized queries for database operations
- Implement rate limiting for APIs
- Use HTTPS for all external communications
- Implement proper authentication and authorization

## 📚 Useful Resources

### Essential Tools

- Black: https://github.com/psf/black
- Pytest: https://docs.pytest.org/

### Python Best Practices

- PEP 8: https://pep8.org/
- PEP 484 (Type Hints): https://www.python.org/dev/peps/pep-0484/
- The Hitchhiker's Guide to Python: https://docs.python-guide.org/

## ⚠️ Important Notes

- **NEVER ASSUME OR GUESS** - When in doubt, ask for clarification
- **Always verify file paths and module names** before use
- **Keep CLAUDE.md updated** when adding new patterns or dependencies
- **Test your code** - No feature is complete without tests
- **Document your decisions** - Future developers (including yourself) will thank you
- **Check reference documentation** - When working with subscriptions/Stripe, review `docs/stripe_integration/`. When working with transactions/SkySlope, review `docs/transactions/README.md`. When working with integrations portal, review `docs/integrations/README.md`

## 📄 Object examples

These examples have been extracted into standalone JSON files under `docs/examples/` for reuse by tooling and tests.

References
- Session/User example: docs/examples/session/user.json
- Stripe products response (sample): docs/examples/subscription/stripe_products.sample.json
- New subscription object: docs/examples/subscription/new_subscription.json
- Legacy subscription object: docs/examples/subscription/legacy_subscription.json
- Change plan instructions: docs/stripe_integration/change-plan-instructions.md
- Other stripe examples and references: docs/stripe_integration/
- **Transactions feature and SkySlope integration: docs/transactions/README.md**
- **Integrations portal (SkySlope, FUB, etc.): docs/integrations/README.md**