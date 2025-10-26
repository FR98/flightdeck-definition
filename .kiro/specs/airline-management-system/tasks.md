# Implementation Plan

- [x] 1. Set up project infrastructure with AWS CDK
  - [x] 1.1 Initialize CDK project and core infrastructure stack
    - Create CDK TypeScript project with proper structure for us-east-2 region
    - Set up CoreInfrastructureStack with DynamoDB tables including DeletedRecords and ActivityLogs
    - Configure multi-tenant table design with proper indexes
    - Add Cognito User Pool and Identity Pool configuration
    - Create IAM roles and policies for Lambda functions
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 1.2 Create Lambda function source code structure
    - Create flightdeck-core-api directory with Python handlers
    - Set up shared utilities and service modules
    - Create requirements.txt for Python dependencies
    - Implement base handler structure with error handling
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 1.3 Create API stack with Lambda functions and API Gateway
    - Implement ApiStack with Lambda function definitions
    - Configure API Gateway with custom authorizer
    - Set up Lambda layers for shared dependencies
    - Add proper CORS and throttling configuration
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 1.4 Implement frontend stack with AWS Amplify
    - Create FrontendStack with AWS Amplify app configuration
    - Set up Amplify CI/CD pipeline with GitHub integration
    - Configure environment-specific branches (dev/prod)
    - Add custom domain configuration through Amplify
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 1.5 Set up monitoring stack for observability
    - Create MonitoringStack with CloudWatch dashboards
    - Configure X-Ray tracing for distributed requests
    - Set up CloudWatch alarms for errors and performance
    - Add cost monitoring and billing alerts
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [x] 2. Implement core authentication and authorization system
  - [x] 2.1 Create Cognito user pool and identity pool configuration
    - Configure user pool with email-based otp passwordless authentication
    - Set up user pool client for React frontend
    - Define custom attributes for user roles
    - _Requirements: 6.1, 6.2, 6.5_

  - [x] 2.2 Implement Lambda authorizer for API Gateway
    - Create custom authorizer function for JWT validation
    - Implement role-based permission checking
    - Add organization isolation validation
    - Handle Sudo, Administrator, Pilot, and Counter roles
    - _Requirements: 6.1, 6.2, 6.5_

  - [x] 2.3 Create user management Lambda functions
    - Implement users_handler for user CRUD operations
    - Create user invitation and role assignment logic
    - Add organization membership management
    - Implement permission validation utilities
    - Add individual user subscription management for premium features
    - _Requirements: 6.1, 6.2, 6.5_

- [x] 3. Implement soft delete and activity logging system
  - [x] 3.1 Create DeletedRecords table and soft delete operations
    - Implement DeletedRecords DynamoDB table with proper indexes
    - Create soft_delete_service module for record deletion and storage
    - Add soft delete functionality to all CRUD operations
    - Implement record restoration from DeletedRecords table
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 3.2 Implement ActivityLogs table and audit trail system
    - Create ActivityLogs DynamoDB table with proper indexes
    - Implement activity_log_service module for comprehensive logging
    - Add activity logging to all data modification operations
    - Create activity log querying and filtering functionality
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 3.3 Create soft delete and rollback Lambda functions
    - Implement soft_delete_handler for deletion and restoration operations
    - Create activity_logs_handler for audit trail management
    - Add rollback functionality using activity logs
    - Implement data change rollback operations
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [x] 3.4 Integrate soft delete and logging into all handlers
    - Update all Lambda functions to use soft delete operations
    - Add activity logging to all CRUD operations
    - Implement rollback endpoints for data recovery
    - Create admin interfaces for deleted records management
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 4. Build multi-tenant organization management system
  - [x] 4.1 Implement organization data models and handlers
    - Create Organizations DynamoDB table operations
    - Implement organization creation and management
    - Add subscription tier validation and feature gating
    - Create organization settings management
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 6.1_

  - [x] 4.2 Implement subscription and billing integration
    - Create billing_handler for subscription management
    - Integrate with Recurrente for payment processing
    - Implement usage tracking and limits enforcement
    - Add subscription tier feature gating logic
    - _Requirements: 6.1, 6.2, 6.5_

  - [-] 4.3 Implement subscription and billing integration according Recurrente API
    - Read and understand Recurrente API at Recurrente-API.postman_collection.json
    - Integrate Recurrente API endpoint to create checkout subscription (so user can create a subscription)
    - Integrate Recurrente API endpoint to get a subscription (so user can read its own susbcription information)
    - Integrate Recurrente API endpoint to cancel a subscription (so user can cancel its own susbcription or change subscription plan)
    - Understand Embedded Checkouts and create tasks for frontend implementation
    - _Requirements: 6.1, 6.2, 6.5_

  - [ ] 4.4 Create UserOrganizations relationship management
    - Implement user-organization mapping table operations
    - Create role assignment and validation logic
    - Add organization member invitation system
    - Implement cross-organization pilot support
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 6.1_

- [ ] 5. Implement global pilot registry and individual subscriptions
  - [ ] 5.1 Create separate Users and Pilots tables
    - Implement Users DynamoDB table for authentication and basic info
    - Create Pilots DynamoDB table for pilot-specific information
    - Add pilot search and lookup functionality
    - Implement total flight hours tracking
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ] 5.2 Implement UserOrganizations relationship management
    - Create UserOrganizations table operations for user-org mapping
    - Implement user invitation to organizations
    - Add organization-specific user data (monthly hours, role)
    - Create user assignment validation logic
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ] 5.3 Create pilots_handler Lambda function
    - Implement pilot CRUD operations with multi-org support
    - Add pilot search across organizations
    - Create pilot invitation and acceptance workflow
    - Implement flight hours calculation and updates
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ] 5.4 Implement individual pilot subscription system
    - Add pilot subscription management (basic/premium tiers)
    - Create feature gating for premium pilot features
    - Implement Recurrente integration for individual pilot billing
    - Add pilot subscription status tracking and validation
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

- [ ] 6. Build aircraft fleet management system
  - [ ] 6.1 Implement Aircraft table and operations
    - Create Aircraft DynamoDB table with organization isolation
    - Implement aircraft registration validation (unique per org)
    - Add aircraft specifications management
    - Create aircraft availability tracking
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

  - [ ] 6.2 Create aircraft_handler Lambda function
    - Implement aircraft CRUD operations
    - Add aircraft search and filtering
    - Create aircraft assignment validation
    - Implement subscription limits for aircraft count
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ] 7. Develop flight operations management
  - [ ] 7.1 Implement Flights table and core operations
    - Create Flights DynamoDB table with proper indexes
    - Implement flight scheduling and management
    - Add flight status tracking (scheduled, in-progress, completed)
    - Create flight number uniqueness validation
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7_

  - [ ] 7.2 Create flights_handler Lambda function
    - Implement flight CRUD operations
    - Add flight assignment validation (pilot, aircraft)
    - Create flight search and filtering by date/route
    - Implement weight calculation integration
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7_

  - [ ] 7.3 Implement flight assignment and validation logic
    - Create pilot availability checking
    - Add aircraft availability validation
    - Implement flight time conflict detection
    - Create captain assignment validation
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7_

- [ ] 8. Build passenger manifest management system
  - [ ] 8.1 Implement PassengerManifests table and operations
    - Create PassengerManifests DynamoDB table
    - Implement passenger data validation
    - Add passenger weight tracking for flight calculations
    - Create passenger search and filtering
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

  - [ ] 8.2 Create passengers_handler Lambda function
    - Implement passenger CRUD operations
    - Add bulk passenger import functionality
    - Create passenger manifest export features
    - Implement Counter role access control
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

  - [ ] 8.3 Implement weight calculation service
    - Create weight_calculation_service module
    - Calculate total flight weight from passengers and cargo
    - Add weight validation against aircraft limits
    - Update flight total weight automatically
    - _Requirements: 3.7, 4.5_

- [ ] 9. Develop BITACORA flight log system
  - [ ] 9.1 Implement FlightLogs table and BITACORA structure
    - Create FlightLogs DynamoDB table
    - Design BITACORA format data structure in JSON
    - Implement flight log validation rules
    - Add correlative number generation and tracking
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ] 9.2 Create flight_logs_handler Lambda function
    - Implement flight log CRUD operations
    - Add BITACORA format validation
    - Create flight log search and filtering
    - Implement Pilot role access control for assigned flights
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ] 9.3 Implement BITACORA service and Excel export
    - Create bitacora_service module for format generation
    - Implement Excel export matching BITACORA.csv structure
    - Add engine parameters validation and tracking
    - Create maintenance section and signature handling
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

- [ ] 10. Implement resilience and performance optimizations
  - [ ] 10.1 Add jitter algorithm and retry mechanisms
    - Implement exponential backoff with full jitter
    - Create retry decorators for DynamoDB operations
    - Add circuit breaker pattern for external services
    - Implement timeout handling with jitter buffers
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 10.2 Optimize Lambda performance and cold starts
    - Implement connection pooling for DynamoDB
    - Add provisioned concurrency for critical functions
    - Create Lambda layer for shared dependencies
    - Optimize memory allocation and timeout settings
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 11. Build React frontend application
  - [ ] 11.1 Create React project structure
    - Initialize React project with Vite build tool in frontend directory
    - Configure Tailwind CSS for styling
    - Set up routing with React Router
    - Create basic project structure and configuration files
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 11.2 Set up core layout and authentication components
    - Create responsive layout components
    - Implement AWS Cognito authentication integration
    - Set up protected routes and navigation
    - Create login and registration components
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 11.3 Implement user management and organization UI
    - Add role-based navigation and access control
    - Create user profile and organization management
    - Implement user invitation and role assignment interfaces
    - Add organization settings and subscription management
    - _Requirements: 6.1, 6.2, 6.5_

  - [ ] 11.4 Build pilot management interface
    - Create pilot list and search components
    - Implement pilot profile creation and editing
    - Add multi-organization pilot assignment UI
    - Create pilot invitation and acceptance workflow
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_

  - [ ] 11.5 Create aircraft management interface
    - Build aircraft fleet overview and management
    - Implement aircraft registration and specifications
    - Add aircraft search and filtering
    - Create aircraft assignment and availability views
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

  - [ ] 11.6 Develop flight operations dashboard
    - Create flight scheduling and management interface
    - Implement flight board with real-time status
    - Add flight search and filtering capabilities
    - Create flight assignment and validation UI
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7_

  - [ ] 11.7 Build passenger manifest management
    - Create passenger list and management interface
    - Implement bulk passenger import/export
    - Add passenger search and filtering
    - Create Counter role specific interface
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5_

  - [ ] 11.8 Implement BITACORA flight log interface
    - Create BITACORA format flight log forms
    - Implement engine parameters input and validation
    - Add maintenance section and signature capture
    - Create Excel export functionality
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ] 11.9 Build soft delete and activity logging interfaces
    - Create admin interface for viewing deleted records
    - Implement record restoration functionality
    - Add activity log viewer with filtering and search
    - Create rollback interface for data recovery
    - Add individual pilot subscription management UI
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 12. Add comprehensive error handling and monitoring
  - [ ] 12.1 Implement frontend error handling
    - Create error boundary components
    - Add API error handling and user feedback
    - Implement loading states and retry mechanisms
    - Create offline support and sync capabilities
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 12.2 Set up monitoring and logging
    - Configure CloudWatch dashboards and alarms
    - Implement X-Ray tracing for request flow
    - Add custom metrics for business operations
    - Create cost monitoring and optimization alerts
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 13. Create comprehensive test suite
  - [ ] 13.1 Write backend unit tests
    - Create unit tests for all Lambda functions
    - Test DynamoDB operations with moto library
    - Add service module unit tests
    - Test role-based access control logic
    - _Requirements: All requirements_

  - [ ] 13.2 Write frontend component tests
    - Create unit tests for React components
    - Test user interactions and form validation
    - Add integration tests for API communication
    - Test role-based UI rendering
    - _Requirements: All requirements_

  - [ ] 13.3 Implement end-to-end testing
    - Create E2E tests for complete user workflows
    - Test multi-tenant isolation and security
    - Add performance and load testing
    - Test BITACORA export functionality
    - _Requirements: All requirements_

- [ ] 14. Deploy and configure production environment with CDK
  - [ ] 14.1 Set up CI/CD pipeline with CDK deployment and Amplify
    - Create GitHub Actions workflow for CDK deployment in us-east-2
    - Configure Amplify CI/CD pipeline for frontend deployment
    - Implement automated testing before CDK synth and deploy
    - Add environment-specific CDK context configurations
    - Create CDK deployment stages (dev, production)
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 14.2 Configure multi-environment CDK deployments
    - Set up CDK bootstrap for us-east-2 region in different AWS accounts
    - Deploy development environment with CDK and Amplify
    - Deploy production environment with proper security and scaling
    - Implement CDK rollback procedures and disaster recovery
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 14.3 Implement CDK infrastructure testing and validation
    - Create CDK unit tests for stack configurations
    - Add integration tests for deployed infrastructure including Amplify
    - Implement infrastructure compliance and security scanning
    - Set up automated cost monitoring and optimization
    - Test soft delete and activity logging functionality
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_
## CDK 
Infrastructure Tasks Summary

The implementation plan now includes comprehensive AWS CDK infrastructure as code approach:

### Key CDK Components:
1. **CoreInfrastructureStack**: DynamoDB tables (including DeletedRecords and ActivityLogs), Cognito, IAM roles
2. **ApiStack**: Lambda functions (including soft delete and activity logging handlers), API Gateway, custom authorizers
3. **FrontendStack**: AWS Amplify app with CI/CD pipeline, custom domain configuration
4. **MonitoringStack**: CloudWatch dashboards, X-Ray tracing, alarms

### CDK Benefits:
- **Version Control**: All infrastructure changes tracked in Git
- **Reproducible Deployments**: Consistent environments across dev/prod
- **Type Safety**: TypeScript provides compile-time validation
- **Automated Resource Management**: Dependency resolution and cleanup
- **Cost Optimization**: Resource tagging and cost allocation
- **Security**: Consistent IAM policies and configurations
- **Rollback Capability**: CloudFormation stack rollback on failures

### Deployment Strategy:
- **Primary Region**: us-east-2 (Ohio) for all AWS resources
- **Frontend**: AWS Amplify with automated CI/CD pipeline
- Development environment for testing and iteration
- Production environment with enhanced security and monitoring
- Automated CI/CD pipeline with CDK deployment stages
- **New Features**: Soft delete system, activity logging, individual pilot subscriptions