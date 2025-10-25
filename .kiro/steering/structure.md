# FlightDeck Project Structure

## Repository Organization
The repository contains two main components:

### flightdeck-core-api/ (Python Lambda Functions)
```
flightdeck-core-api/
├── requirements.txt          # Python dependencies
├── shared/                   # Shared utilities and base classes
│   ├── base_handler.py      # Base handler with error handling & CORS
│   ├── response.py          # HTTP response helpers
│   ├── exceptions.py        # Custom exception hierarchy
│   ├── utils.py             # Common utility functions
│   └── auth_middleware.py   # JWT validation middleware
├── services/                # Business logic services
│   ├── dynamodb_service.py  # DynamoDB operations wrapper
│   ├── multi_tenant_service.py # Organization isolation logic
│   ├── permission_service.py # Role-based access control
│   ├── subscription_service.py # Billing and subscription logic
│   └── user_invitation_service.py # User invitation workflow
├── handlers/                # Lambda function handlers (one per resource)
│   ├── auth_handler.py      # JWT authorizer for API Gateway
│   ├── users_handler.py     # User management CRUD
│   ├── pilots_handler.py    # Pilot-specific operations
│   ├── aircraft_handler.py  # Aircraft fleet management
│   ├── flights_handler.py   # Flight operations
│   ├── passengers_handler.py # Passenger manifest management
│   ├── flight_logs_handler.py # BITACORA flight logs
│   ├── organizations_handler.py # Organization management
│   ├── billing_handler.py   # Subscription and billing
│   ├── activity_logs_handler.py # Audit trail operations
│   └── soft_delete_handler.py # Data recovery operations
└── tests/                   # Unit tests
    ├── test_auth_handler.py
    ├── test_users_handler.py
    └── test_otp_auth.py
```

### flightdeck-iac/ (AWS CDK Infrastructure)
```
flightdeck-iac/
├── package.json             # Node.js dependencies and scripts
├── cdk.json                 # CDK configuration
├── tsconfig.json           # TypeScript configuration
├── bin/                    # CDK app entry point
├── lib/                    # CDK stack definitions
│   ├── flightdeck-core-stack.ts     # DynamoDB, Cognito, IAM
│   ├── flightdeck-api-stack.ts      # Lambda functions, API Gateway
│   ├── flightdeck-frontend-stack.ts # Amplify hosting
│   └── flightdeck-monitoring-stack.ts # CloudWatch dashboards
├── test/                   # Infrastructure tests
├── deploy.sh               # Automated deployment script
├── deploy-api.sh          # API-only deployment
├── deploy-frontend.sh     # Frontend-only deployment
└── cdk.out/               # Generated CloudFormation templates
```

## Architectural Patterns

### Handler Pattern
- All Lambda handlers inherit from `BaseHandler`
- HTTP method routing (`handle_get`, `handle_post`, etc.)
- Automatic error handling and CORS
- JWT token extraction and validation
- Role-based access control decorators

### Service Layer
- Business logic separated from handlers
- DynamoDB operations abstracted in `DynamoDBService`
- Multi-tenant isolation enforced in `MultiTenantService`
- Reusable across multiple handlers

### Multi-Tenant Design
- Organization-based data isolation
- JWT tokens contain organization context
- All database operations scoped to user's organization
- Role hierarchy: Sudo > Administrator > Pilot > Counter

### Error Handling
- Custom exception hierarchy in `shared/exceptions.py`
- Automatic HTTP status code mapping
- Structured error responses
- Comprehensive logging and tracing

## Naming Conventions
- **Files**: snake_case for Python, kebab-case for TypeScript
- **Classes**: PascalCase
- **Functions/Methods**: snake_case (Python), camelCase (TypeScript)
- **Constants**: UPPER_SNAKE_CASE
- **DynamoDB Tables**: `flightdeck-{resource}` format
- **Lambda Functions**: `flightdeck-{resource}-handler` format