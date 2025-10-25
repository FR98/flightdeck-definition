# FlightDeck Technology Stack

## Architecture
- **Cloud Platform**: AWS (us-east-2 region)
- **Infrastructure as Code**: AWS CDK with TypeScript
- **API**: AWS Lambda + API Gateway (REST)
- **Database**: DynamoDB with multi-tenant design
- **Authentication**: AWS Cognito with JWT tokens
- **Frontend Hosting**: AWS Amplify with CI/CD
- **Monitoring**: CloudWatch + X-Ray tracing

## Backend (Python)
- **Runtime**: Python 3.x on AWS Lambda
- **Framework**: Custom base handler pattern with decorators
- **Key Dependencies**:
  - `boto3` - AWS SDK
  - `pyjwt` - JWT token handling
  - `requests` - HTTP client
  - `openpyxl` - Excel export for BITACORA logs
  - `cryptography` - Cryptographic operations

## Infrastructure (TypeScript)
- **CDK Version**: 2.215.0
- **Node.js**: 18+
- **TypeScript**: ~5.6.3
- **Testing**: Jest with ts-jest

## Common Commands

### Infrastructure Deployment
```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Deploy all stacks
./deploy.sh

# Deploy individual stacks
npx cdk deploy FlightDeckCore
npx cdk deploy FlightDeckApi
npx cdk deploy FlightDeckFrontend
```

### Development
```bash
# Watch mode for TypeScript
npm run watch

# Run tests
npm test

# CDK commands
npx cdk diff
npx cdk synth
npx cdk destroy
```

### Python API Development
```bash
# Install dependencies (in flightdeck-core-api/)
pip install -r requirements.txt

# Run tests
python -m pytest tests/
```

## Code Patterns
- **Lambda Handlers**: Inherit from `BaseHandler` class
- **Error Handling**: Custom exception hierarchy with automatic HTTP status mapping
- **Multi-tenancy**: Organization-based isolation through JWT claims
- **Role-based Access**: Decorator pattern for permission checking
- **Soft Deletes**: All deletions go to `deleted_records` table for recovery