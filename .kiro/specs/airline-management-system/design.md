# Design Document

## Overview

**FlightDeck** is a serverless SaaS web application that provides comprehensive management capabilities for airline operations. The system follows a modern serverless architecture with Python AWS Lambda functions serving a React frontend, designed to handle multi-tenant pilot management, aircraft fleet operations, flight scheduling, passenger manifests, and regulatory-compliant flight logging with subscription-based access control.

## Architecture

### System Architecture

The application follows a serverless SaaS architecture with multi-tenancy support:

```
┌─────────────────┐    HTTPS/REST API   ┌─────────────────┐
│   React Client  │ ◄─────────────────► │  API Gateway    │
│   (Frontend)    │                     │                 │
└─────────────────┘                     └─────────────────┘
                                                │
                                                ▼
                                        ┌─────────────────┐
                                        │  Lambda Functions│
                                        │   (Python)      │
                                        └─────────────────┘
                                                │
                                                ▼
                                        ┌─────────────────┐
                                        │   DynamoDB      │
                                        │   (Multi-tenant)│
                                        └─────────────────┘
```

### Technology Stack

- **Backend**: Python AWS Lambda functions (serverless)
- **Frontend**: React 18+ with Vite build tool
- **Frontend Deployment**: AWS Amplify for CI/CD and hosting
- **Styling**: Tailwind CSS for responsive design
- **Database**: Amazon DynamoDB (NoSQL)
- **Authentication**: AWS Cognito with JWT tokens
- **API**: AWS API Gateway with Lambda integration
- **Subscriptions**: Stripe for billing
- **Infrastructure**: AWS CDK (Cloud Development Kit) for Infrastructure as Code
- **Primary Region**: us-east-2 (Ohio)

### Deployment Architecture

- **Development**: Local development with CDK synth/deploy and Vite dev server
- **Infrastructure as Code**: AWS CDK for all resource provisioning and management
- **Frontend Deployment**: AWS Amplify for automated CI/CD pipeline and hosting
- **Primary Region**: us-east-2 (Ohio) for all AWS resources
- **Production**: AWS serverless deployment with:
  - AWS Amplify for React frontend hosting and CI/CD
  - API Gateway for API endpoints
  - Lambda functions for business logic
  - DynamoDB for data storage
  - Cognito for authentication
  - CloudWatch for monitoring
  - All resources defined and managed through CDK stacks

## Components and Interfaces

### Backend Components (AWS Lambda + DynamoDB)

#### Data Models (DynamoDB Tables)

1. **Organizations Table** (Multi-tenancy)
   - PK: org_id
   - Attributes: name, subscription_type, subscription_status, billing_info, created_at

2. **Users Table** (Authentication and basic user information)
   - PK: user_id (Cognito sub)
   - Attributes: email, first_name, last_name, global_role (sudo only), subscription_type (basic/premium), subscription_status, age, qualifications, total_hours, license_number, created_at

4. **UserOrganizations Table** (User-organization-role mapping)
   - PK: org_id, SK: user_id
   - Attributes: role (administrator, pilot, counter), status (active/inactive), monthly_hours, hire_date, organization_pilot_id, created_at

5. **Aircraft Table**
   - PK: org_id, SK: aircraft_registration_id
   - Attributes: manufacturer, aircraft_registration_id, max_weight, required_crew, created_at, updated_at

6. **Flights Table**
   - PK: org_id, SK: flight_id
   - Attributes: flight_number, flight_type, aircraft_type, route, aircraft_id, captain_id, copilot_id, total_block_time, total_flight_time, total_cycles, customer_name, observations, maintenance_inspection, pilot_acceptance, fuel_information, created_at, updated_at

7. **PassengerManifests Table**
   - PK: org_id#flight_id, SK: passenger_id
   - Attributes: passenger_name, identification_number, nationality, weight, created_at, updated_at

8. **FlightLogs Table**
   - PK: org_id#flight_id, SK: log_id
   - Attributes: date, route, engine_start_time, engine_stop_time, engine_block_time, departure_time, arrival_time, flight_time, cycles, customer_name, engine_1_oat, engine_1_alt, engine_1_vel, engine_1_torque, engine_1_propeller_rpm, engine_1_itt, engine_1_ng, engine_1_oil_pressure, engine_1_oil_temp, engine_1_fuel_flow, created_at, updated_at

9. **DeletedRecords Table** (Soft delete storage)
   - PK: table_name, SK: record_pk#record_sk#deleted_timestamp
   - Attributes: original_record (JSON), deleted_by, org_id, created_at

10. **ActivityLogs Table** (Audit trail)
    - PK: table_name, SK: record_pk#record_sk#timestamp#log_id
    - Attributes: user_id, action_type (CREATE/UPDATE/DELETE), table_name, record_id, old_values (JSON), new_values (JSON), ip_address, user_agent, created_at

#### Lambda Functions

1. **users_handler**: User management and organization assignments
2. **pilots_handler**: Pilot-specific data and operations
3. **aircraft_handler**: Fleet management operations  
4. **flights_handler**: Flight scheduling and management
5. **passengers_handler**: Passenger manifest management
6. **flight_logs_handler**: Flight log creation and export
7. **auth_handler**: Authentication and authorization
8. **billing_handler**: Subscription and billing management (organization and individual pilot subscriptions)
9. **organizations_handler**: Multi-tenant organization management
10. **activity_logs_handler**: Audit trail and activity logging
11. **soft_delete_handler**: Soft delete operations and rollback functionality
12. **auth_middleware**: Role-based access control and permissions

#### Services (Python Modules)

1. **bitacora_service**: Handles BITACORA format generation and Excel export
2. **weight_calculation_service**: Calculates total flight weight from passengers and cargo
3. **flight_hours_service**: Updates pilot flight hours tracking
4. **validation_service**: Cross-model validation logic
5. **subscription_service**: Manages SaaS billing and feature access (organization and individual pilot subscriptions)
6. **multi_tenant_service**: Handles organization isolation and data access
7. **soft_delete_service**: Manages soft delete operations and rollback functionality
8. **activity_log_service**: Records all data changes and user actions for audit trail
9. **rollback_service**: Handles data rollback operations using activity logs and deleted records

### Frontend Components (React)

#### Core Layout Components

1. **App**: Main application wrapper with routing
2. **Navigation**: Top navigation bar with menu items
3. **Sidebar**: Side navigation for different modules
4. **Layout**: Main layout wrapper for pages

#### Feature Components

1. **Pilot Management**
   - PilotList: Display all pilots with search and filter
   - PilotForm: Create/edit pilot information
   - PilotDetail: View detailed pilot information and flight history

2. **Aircraft Management**
   - AircraftList: Fleet overview with specifications
   - AircraftForm: Add/edit aircraft information
   - AircraftDetail: Detailed aircraft view with flight history

3. **Flight Operations**
   - FlightList: All flights with status and filters
   - FlightForm: Create/edit flight schedules
   - FlightDetail: Comprehensive flight information
   - FlightBoard: Real-time flight status dashboard

4. **Passenger Management**
   - PassengerManifest: Flight passenger list management
   - PassengerForm: Add/edit passenger information
   - ManifestExport: Export passenger lists

5. **Flight Logs**
   - FlightLogForm: Create and edit flight logs with BITACORA format fields
   - BitacoraViewer: Display flight log in standard BITACORA format
   - EngineParametersForm: Record engine parameters during flight
   - MaintenanceSection: Handle discrepancies and maintenance release
   - LogExport: Export logs to Excel in BITACORA format

#### Shared Components

1. **DataTable**: Reusable table component with sorting and pagination
2. **FormComponents**: Input fields, selects, date pickers
3. **Modal**: Reusable modal dialogs
4. **LoadingSpinner**: Loading state indicator
5. **ErrorBoundary**: Error handling wrapper

### API Interfaces

#### API Gateway Endpoints (Lambda Integration)

```
# User Management
GET    /api/users                  # List organization users (users_handler)
POST   /api/users                  # Create/invite user to org (users_handler)
GET    /api/users/{id}             # Get user details (users_handler)
PUT    /api/users/{id}             # Update user profile (users_handler)
POST   /api/users/{id}/invite      # Invite existing user to org (users_handler)
DELETE /api/users/{id}/remove      # Soft delete user from org (users_handler)
GET    /api/users/search           # Search global user registry (users_handler)

# Pilot Management
GET    /api/pilots                 # List organization pilots (pilots_handler)
POST   /api/pilots                 # Create/invite pilot to org (pilots_handler)
GET    /api/pilots/{id}            # Get pilot details (pilots_handler)
PUT    /api/pilots/{id}            # Update pilot profile (pilots_handler)
POST   /api/pilots/{id}/invite     # Invite existing pilot to org (pilots_handler)
DELETE /api/pilots/{id}/remove     # Soft delete pilot from org (pilots_handler)
GET    /api/pilots/search          # Search global pilot registry (pilots_handler)
POST   /api/pilots/{id}/subscribe  # Manage individual pilot subscription (pilots_handler)

# Aircraft Management  
GET    /api/aircraft               # List organization aircraft (aircraft_handler)
POST   /api/aircraft               # Create new aircraft (aircraft_handler)
GET    /api/aircraft/{id}          # Get aircraft details (aircraft_handler)
PUT    /api/aircraft/{id}          # Update aircraft (aircraft_handler)
DELETE /api/aircraft/{id}          # Delete aircraft (aircraft_handler)

# Flight Operations
GET    /api/flights                # List organization flights (flights_handler)
POST   /api/flights                # Create new flight (flights_handler)
GET    /api/flights/{id}           # Get flight details (flights_handler)
PUT    /api/flights/{id}           # Update flight (flights_handler)
DELETE /api/flights/{id}           # Delete flight (flights_handler)

# Passenger Management
GET    /api/flights/{id}/passengers # Get flight passengers (passengers_handler)
POST   /api/flights/{id}/passengers # Add passenger (passengers_handler)
PUT    /api/passengers/{id}         # Update passenger (passengers_handler)
DELETE /api/passengers/{id}         # Remove passenger (passengers_handler)

# Flight Logs
GET    /api/flights/{id}/log        # Get flight log (flight_logs_handler)
POST   /api/flights/{id}/log        # Create flight log (flight_logs_handler)
PUT    /api/flight_logs/{id}        # Update flight log (flight_logs_handler)
GET    /api/flight_logs/{id}/export # Export BITACORA Excel (flight_logs_handler)

# Organization & Billing
GET    /api/organization            # Get org details (organizations_handler)
PUT    /api/organization            # Update org settings (organizations_handler)
POST   /api/billing/subscribe       # Create subscription (billing_handler)
GET    /api/billing/usage           # Get usage metrics (billing_handler)

# Activity Logs & Audit Trail
GET    /api/activity-logs           # Get organization activity logs (activity_logs_handler)
GET    /api/activity-logs/{id}      # Get specific log details (activity_logs_handler)

# Soft Delete & Rollback
GET    /api/deleted-records         # List soft deleted records (soft_delete_handler)
POST   /api/deleted-records/{id}/restore # Restore soft deleted record (soft_delete_handler)
POST   /api/rollback/{log_id}       # Rollback specific change (soft_delete_handler)
```

## Soft Delete and Activity Logging

### Soft Delete Implementation

All data modifications in the system implement soft delete functionality to ensure data integrity and enable rollback capabilities.

#### Soft Delete Strategy

1. **Soft Delete Fields**: All tables include `deleted_at` timestamp field
2. **Deleted Records Storage**: Complete record snapshots stored in `DeletedRecords` table
3. **Query Filtering**: All queries automatically filter out soft-deleted records
4. **Rollback Capability**: Deleted records can be restored from the `DeletedRecords` table

#### Soft Delete Process

```python
def soft_delete_record(table_name, record_id, user_id, reason="User requested"):
    """
    Soft delete a record and store it for potential rollback
    """
    # Get the original record
    original_record = get_record(table_name, record_id)
    
    # Store in DeletedRecords table
    deleted_record = {
        'table_name': table_name,
        'record_id': f"{record_id}#{int(time.time())}",
        'original_record': json.dumps(original_record),
        'deleted_by': user_id,
        'deletion_reason': reason,
        'org_id': original_record.get('org_id'),
        'created_at': datetime.utcnow().isoformat()
    }
    
    # Save to DeletedRecords table
    save_deleted_record(deleted_record)
    
    # Remove original record from main table
    delete_record(table_name, record_id)
    
    # Log the deletion activity
    log_activity(user_id, 'DELETE', table_name, record_id, original_record, None)
```

#### Rollback Functionality

```python
def restore_deleted_record(deleted_record_id, user_id):
    """
    Restore a soft-deleted record from DeletedRecords table
    """
    # Get the deleted record
    deleted_record = get_deleted_record(deleted_record_id)
    original_data = json.loads(deleted_record['original_record'])
    
    # Remove deleted_at timestamp
    original_data.pop('deleted_at', None)
    
    # Restore the record
    table_name = deleted_record['table_name']
    restore_record(table_name, original_data)
    
    # Log the restoration activity
    log_activity(user_id, 'RESTORE', table_name, original_data['id'], None, original_data)
    
    # Remove from DeletedRecords table
    delete_deleted_record(deleted_record_id)
```

### Activity Logging System

Comprehensive audit trail system that tracks all data changes and user actions.

#### Activity Log Structure

```python
activity_log = {
    'org_id': 'organization_id',
    'timestamp_log_id': f"{timestamp}#{log_id}",
    'user_id': 'user_who_performed_action',
    'action_type': 'CREATE|UPDATE|DELETE|RESTORE',
    'table_name': 'affected_table',
    'record_id': 'affected_record_id',
    'old_values': json.dumps(previous_state),  # For UPDATE operations
    'new_values': json.dumps(new_state),       # For CREATE/UPDATE operations
    'ip_address': 'user_ip_address',
    'user_agent': 'browser_user_agent',
    'created_at': 'iso_timestamp'
}
```

#### Activity Logging Implementation

```python
def log_activity(user_id, action_type, table_name, record_id, old_values=None, new_values=None, request_context=None):
    """
    Log all data changes and user actions
    """
    timestamp = int(time.time() * 1000)  # Millisecond precision
    log_id = str(uuid.uuid4())
    
    activity_log = {
        'org_id': get_user_org_id(user_id),
        'timestamp_log_id': f"{timestamp}#{log_id}",
        'user_id': user_id,
        'action_type': action_type,
        'table_name': table_name,
        'record_id': record_id,
        'old_values': json.dumps(old_values) if old_values else None,
        'new_values': json.dumps(new_values) if new_values else None,
        'ip_address': request_context.get('ip_address') if request_context else None,
        'user_agent': request_context.get('user_agent') if request_context else None,
        'created_at': datetime.utcnow().isoformat()
    }
    
    save_activity_log(activity_log)
```

#### Data Change Rollback

```python
def rollback_change(activity_log_id, user_id):
    """
    Rollback a specific data change using activity log
    """
    activity_log = get_activity_log(activity_log_id)
    
    if activity_log['action_type'] == 'CREATE':
        # Soft delete the created record
        soft_delete_record(activity_log['table_name'], activity_log['record_id'], user_id, "Rollback operation")
    
    elif activity_log['action_type'] == 'UPDATE':
        # Restore previous values
        old_values = json.loads(activity_log['old_values'])
        update_record(activity_log['table_name'], old_values)
        log_activity(user_id, 'ROLLBACK', activity_log['table_name'], activity_log['record_id'], None, old_values)
    
    elif activity_log['action_type'] == 'DELETE':
        # Find and restore from DeletedRecords
        deleted_record = find_deleted_record(activity_log['table_name'], activity_log['record_id'])
        restore_deleted_record(deleted_record['record_id'], user_id)
```

### Query Filtering for Soft Deletes

All database queries automatically filter out soft-deleted records:

```python
def get_active_records(table_name, org_id, filters=None):
    """
    Get all active (non-deleted) records for an organization
    Records are active if they exist in the main table (not in DeletedRecords)
    """
    return query_table(table_name, org_id, filters)

def get_deleted_records(org_id, table_name=None):
    """
    Get soft-deleted records from DeletedRecords table
    """
    query_filters = {'org_id': org_id}
    if table_name:
        query_filters['table_name'] = table_name
    
    return query_table('DeletedRecords', query_filters)
```

## Individual Pilot Subscriptions

### Pilot Premium Features

Pilots can have individual subscriptions for premium features independent of organization subscriptions.

#### Pilot Subscription Tiers

1. **Basic Pilot** (Free)
   - Access to assigned flights and basic logging
   - Standard BITACORA format export
   - Basic flight hour tracking

2. **Premium Pilot** ($19/month)
   - Advanced flight analytics and reporting
   - Personal flight logbook with detailed statistics
   - Mobile app access with offline capabilities
   - Custom BITACORA templates
   - Flight performance analytics
   - Career progression tracking

#### Pilot Subscription Management

```python
def manage_pilot_subscription(user_id, subscription_type, payment_method=None):
    """
    Manage individual pilot subscription
    """
    user = get_user(user_id)
    
    # Update user subscription
    user['subscription_type'] = subscription_type
    user['subscription_status'] = 'active' if payment_method else 'pending'
    user['subscription_updated_at'] = datetime.utcnow().isoformat()
    
    if payment_method:
        # Process payment through Stripe
        process_pilot_subscription_payment(user_id, subscription_type, payment_method)
    
    update_user(user)
    log_activity(user_id, 'UPDATE', 'Users', user_id, None, {'subscription_type': subscription_type})
```

#### Feature Gating for Pilot Subscriptions

```python
def check_pilot_feature_access(user_id, feature):
    """
    Check if pilot has access to premium features
    """
    user = get_user(user_id)
    subscription_type = user.get('subscription_type', 'basic')
    
    premium_features = [
        'advanced_analytics',
        'mobile_app',
        'custom_templates',
        'performance_analytics',
        'career_tracking'
    ]
    
    if feature in premium_features:
        return subscription_type == 'premium' and user.get('subscription_status') == 'active'
    
    return True  # Basic features available to all
```

## Data Models

### DynamoDB Schema Design

#### Table Structure

```python
# Organizations Table
{
  "TableName": "Organizations",
  "KeySchema": [
    {"AttributeName": "org_id", "KeyType": "HASH"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "org_id", "AttributeType": "S"},
    {"AttributeName": "subscription_status", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "subscription-status-index",
      "KeySchema": [{"AttributeName": "subscription_status", "KeyType": "HASH"}]
    }
  ],
  "Attributes": {
    "org_id": "String (Primary Key)",
    "name": "String",
    "subscription_type": "String",
    "subscription_status": "String",
    "billing_info": "Map",
    "created_at": "String (ISO timestamp)"
  }
}

# Users Table (Authentication and basic user information)
{
  "TableName": "Users",
  "KeySchema": [
    {"AttributeName": "user_id", "KeyType": "HASH"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "user_id", "AttributeType": "S"},
    {"AttributeName": "email", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "email-index",
      "KeySchema": [{"AttributeName": "email", "KeyType": "HASH"}]
    }
  ],
  "Attributes": {
    "user_id": "String (Primary Key - Cognito sub)",
    "email": "String",
    "first_name": "String",
    "last_name": "String",
    "global_role": "String (sudo only)",
    "subscription_type": "String (basic/premium)",
    "subscription_status": "String",
    "created_at": "String (ISO timestamp)"
  }
}

# Pilots Table (Pilot-specific information)
{
  "TableName": "Pilots",
  "KeySchema": [
    {"AttributeName": "pilot_id", "KeyType": "HASH"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "pilot_id", "AttributeType": "S"},
    {"AttributeName": "license_number", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "license-number-index",
      "KeySchema": [{"AttributeName": "license_number", "KeyType": "HASH"}]
    }
  ],
  "Attributes": {
    "pilot_id": "String (Primary Key - same as user_id for pilots)",
    "age": "Number",
    "qualifications": "List",
    "total_hours": "Number",
    "license_number": "String",
    "created_at": "String (ISO timestamp)"
  }
}

# UserOrganizations Table (User-organization relationships)
{
  "TableName": "UserOrganizations",
  "KeySchema": [
    {"AttributeName": "user_id", "KeyType": "HASH"},
    {"AttributeName": "org_id", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "user_id", "AttributeType": "S"},
    {"AttributeName": "org_id", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "org-users-index",
      "KeySchema": [
        {"AttributeName": "org_id", "KeyType": "HASH"},
        {"AttributeName": "user_id", "KeyType": "RANGE"}
      ]
    }
  ],
  "Attributes": {
    "user_id": "String (Primary Key)",
    "org_id": "String (Sort Key)",
    "role": "String (administrator, pilot, counter)",
    "status": "String (active/inactive)",
    "assigned_by": "String",
    "monthly_hours": "Number (for pilots)",
    "hire_date": "String (ISO date)",
    "organization_pilot_id": "String",
    "created_at": "String (ISO timestamp)"
  }
}

# PassengerManifests Table
{
  "TableName": "PassengerManifests",
  "KeySchema": [
    {"AttributeName": "flight_composite_key", "KeyType": "HASH"},
    {"AttributeName": "passenger_id", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "flight_composite_key", "AttributeType": "S"},
    {"AttributeName": "passenger_id", "AttributeType": "S"}
  ],
  "Attributes": {
    "flight_composite_key": "String (Primary Key - org_id#flight_id)",
    "passenger_id": "String (Sort Key)",
    "passenger_name": "String",
    "identification_number": "String",
    "nationality": "String",
    "weight": "Number",
    "created_at": "String (ISO timestamp)",
    "updated_at": "String (ISO timestamp)"
  }
}

# FlightLogs Table
{
  "TableName": "FlightLogs",
  "KeySchema": [
    {"AttributeName": "flight_composite_key", "KeyType": "HASH"},
    {"AttributeName": "log_id", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "flight_composite_key", "AttributeType": "S"},
    {"AttributeName": "log_id", "AttributeType": "S"}
  ],
  "Attributes": {
    "flight_composite_key": "String (Primary Key - org_id#flight_id)",
    "log_id": "String (Sort Key)",
    "date": "String (ISO date)",
    "route": "String",
    "engine_start_time": "String (ISO timestamp)",
    "engine_stop_time": "String (ISO timestamp)",
    "engine_block_time": "Number",
    "departure_time": "String (ISO timestamp)",
    "arrival_time": "String (ISO timestamp)",
    "flight_time": "Number",
    "cycles": "Number",
    "customer_name": "String",
    "engine_1_oat": "Number",
    "engine_1_alt": "Number",
    "engine_1_vel": "Number",
    "engine_1_torque": "Number",
    "engine_1_propeller_rpm": "Number",
    "engine_1_itt": "Number",
    "engine_1_ng": "Number",
    "engine_1_oil_pressure": "Number",
    "engine_1_oil_temp": "Number",
    "engine_1_fuel_flow": "Number",
    "created_at": "String (ISO timestamp)",
    "updated_at": "String (ISO timestamp)"
  }
}

# DeletedRecords Table (Soft delete storage)
{
  "TableName": "DeletedRecords",
  "KeySchema": [
    {"AttributeName": "table_name", "KeyType": "HASH"},
    {"AttributeName": "record_id_timestamp", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "table_name", "AttributeType": "S"},
    {"AttributeName": "record_id_timestamp", "AttributeType": "S"},
    {"AttributeName": "org_id", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "org-deleted-records-index",
      "KeySchema": [
        {"AttributeName": "org_id", "KeyType": "HASH"},
        {"AttributeName": "record_id_timestamp", "KeyType": "RANGE"}
      ]
    }
  ],
  "Attributes": {
    "table_name": "String (Primary Key)",
    "record_id_timestamp": "String (Sort Key - record_id#deleted_timestamp)",
    "original_record": "String (JSON)",
    "deleted_by": "String",
    "deletion_reason": "String",
    "org_id": "String",
    "created_at": "String (ISO timestamp)"
  }
}

# ActivityLogs Table (Audit trail)
{
  "TableName": "ActivityLogs",
  "KeySchema": [
    {"AttributeName": "org_id", "KeyType": "HASH"},
    {"AttributeName": "timestamp_log_id", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "org_id", "AttributeType": "S"},
    {"AttributeName": "timestamp_log_id", "AttributeType": "S"},
    {"AttributeName": "user_id", "AttributeType": "S"},
    {"AttributeName": "table_name", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "user-activity-index",
      "KeySchema": [
        {"AttributeName": "user_id", "KeyType": "HASH"},
        {"AttributeName": "timestamp_log_id", "KeyType": "RANGE"}
      ]
    },
    {
      "IndexName": "table-activity-index",
      "KeySchema": [
        {"AttributeName": "table_name", "KeyType": "HASH"},
        {"AttributeName": "timestamp_log_id", "KeyType": "RANGE"}
      ]
    }
  ],
  "Attributes": {
    "org_id": "String (Primary Key)",
    "timestamp_log_id": "String (Sort Key - timestamp#log_id)",
    "user_id": "String",
    "action_type": "String (CREATE/UPDATE/DELETE/RESTORE)",
    "table_name": "String",
    "record_id": "String",
    "old_values": "String (JSON)",
    "new_values": "String (JSON)",
    "ip_address": "String",
    "user_agent": "String",
    "created_at": "String (ISO timestamp)"
  }
}

# Aircraft Table
{
  "TableName": "Aircraft", 
  "KeySchema": [
    {"AttributeName": "org_id", "KeyType": "HASH"},
    {"AttributeName": "aircraft_id", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "org_id", "AttributeType": "S"},
    {"AttributeName": "aircraft_id", "AttributeType": "S"},
    {"AttributeName": "registration", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "registration-index",
      "KeySchema": [{"AttributeName": "registration", "KeyType": "HASH"}]
    }
  ],
  "Attributes": {
    "org_id": "String (Primary Key)",
    "aircraft_id": "String (Sort Key)",
    "manufacturer": "String",
    "registration": "String",
    "max_weight": "Number",
    "required_crew": "Number",
    "created_at": "String (ISO timestamp)",
    "updated_at": "String (ISO timestamp)"
  }
}

# Flights Table
{
  "TableName": "Flights",
  "KeySchema": [
    {"AttributeName": "org_id", "KeyType": "HASH"},
    {"AttributeName": "flight_id", "KeyType": "RANGE"}
  ],
  "AttributeDefinitions": [
    {"AttributeName": "org_id", "AttributeType": "S"},
    {"AttributeName": "flight_id", "AttributeType": "S"},
    {"AttributeName": "flight_number", "AttributeType": "S"},
    {"AttributeName": "departure_date", "AttributeType": "S"}
  ],
  "GlobalSecondaryIndexes": [
    {
      "IndexName": "flight-number-index",
      "KeySchema": [{"AttributeName": "flight_number", "KeyType": "HASH"}]
    },
    {
      "IndexName": "date-index", 
      "KeySchema": [
        {"AttributeName": "org_id", "KeyType": "HASH"},
        {"AttributeName": "departure_date", "KeyType": "RANGE"}
      ]
    }
  ],
  "Attributes": {
    "org_id": "String (Primary Key)",
    "flight_id": "String (Sort Key)",
    "flight_number": "String",
    "flight_type": "String",
    "aircraft_type": "String",
    "route": "String",
    "aircraft_id": "String",
    "captain_id": "String",
    "copilot_id": "String",
    "total_block_time": "Number",
    "total_flight_time": "Number",
    "total_cycles": "Number",
    "customer_name": "String",
    "observations": "String",
    "maintenance_inspection": "String",
    "pilot_acceptance": "String",
    "fuel_information": "Map",
    "created_at": "String (ISO timestamp)",
    "updated_at": "String (ISO timestamp)"
  }
}
```

### Data Relationships (DynamoDB Design Patterns)

- Pilot ↔ Organizations (many-to-many via PilotOrganizations table)
- Organization → Aircraft (one-to-many, same partition key)  
- Organization → Flights (one-to-many, same partition key)
- Flight → PassengerManifests (one-to-many, composite key with flight_id)
- Flight → FlightLog (one-to-one, composite key with flight_id)
- Pilot → Flights (one-to-many across organizations, tracked via captain_id)
- Organization isolation maintained while allowing pilot sharing

## Error Handling

### Backend Error Handling (Lambda Functions)

1. **Validation Errors**: Return 422 with detailed field errors
2. **Not Found**: Return 404 for missing DynamoDB items
3. **Authentication**: Return 401 for invalid Cognito tokens
4. **Authorization**: Return 403 for organization access violations
5. **Subscription Limits**: Return 402 for exceeded plan limits
6. **DynamoDB Errors**: Catch and transform to user-friendly messages
7. **Lambda Timeouts**: Return 504 with retry instructions

### Frontend Error Handling

1. **API Errors**: Display user-friendly error messages
2. **Network Errors**: Show connection status and retry options
3. **Validation Errors**: Real-time form validation feedback
4. **Component Errors**: Error boundaries to prevent app crashes
5. **Loading States**: Clear loading indicators and timeouts

### Error Response Format

```json
{
  "error": {
    "message": "Validation failed",
    "details": {
      "first_name": ["can't be blank"],
      "age": ["must be between 18 and 70"]
    }
  }
}
```

## Testing Strategy

### Backend Testing (Python Lambda)

1. **Unit Tests**: Service module functions and validation logic
2. **Lambda Handler Tests**: Function entry points and responses
3. **Integration Tests**: DynamoDB operations and data flow
4. **Service Tests**: Business logic in service modules
5. **Multi-tenancy Tests**: Organization isolation validation
6. **Subscription Tests**: Feature gating and billing logic

### Frontend Testing (React)

1. **Component Tests**: Individual component rendering and behavior
2. **Integration Tests**: Component interaction and data flow
3. **E2E Tests**: Complete user workflows
4. **API Integration Tests**: Frontend-backend communication
5. **Accessibility Tests**: WCAG compliance validation

### Test Coverage Goals

- Backend: 90%+ code coverage
- Frontend: 80%+ component coverage
- Critical paths: 100% coverage (authentication, data persistence)

### Testing Tools

- **Backend**: pytest, moto (AWS mocking), boto3 testing utilities
- **Frontend**: Jest, React Testing Library, Cypress
- **API Testing**: Postman/Newman for API Gateway endpoint testing
- **Infrastructure**: AWS SAM CLI for local Lambda testing
- **Load Testing**: Artillery or Locust for serverless performance testing

## Security Considerations

### Authentication & Authorization

1. **AWS Cognito**: Managed authentication service with JWT tokens
2. **Organization-based Access**: Multi-tenant isolation and permissions
3. **Role-based Access**: Sudo, Administrator, Pilot, and Counter roles with specific permissions
4. **API Gateway Authorizers**: Lambda authorizers for request validation
5. **Session Management**: Cognito handles token refresh and security

### Data Security

1. **Input Validation**: Sanitize all user inputs in Lambda functions
2. **NoSQL Injection Prevention**: Validate DynamoDB query parameters
3. **XSS Protection**: Escape output and use CSP headers
4. **CORS Configuration**: API Gateway CORS policies
5. **Data Encryption**: DynamoDB encryption at rest and in transit
6. **Organization Isolation**: Strict partition key validation

### Infrastructure Security

1. **HTTPS**: CloudFront and API Gateway enforce SSL/TLS
2. **Environment Variables**: AWS Systems Manager Parameter Store
3. **IAM Roles**: Least privilege access for Lambda functions
4. **VPC**: Optional VPC deployment for enhanced security
5. **Rate Limiting**: API Gateway throttling and usage plans
6. **WAF**: Web Application Firewall for additional protection

## Performance Considerations

### Backend Optimization

1. **DynamoDB Indexing**: GSI and LSI for efficient queries
2. **Lambda Caching**: In-memory caching and connection reuse
3. **Pagination**: DynamoDB pagination with LastEvaluatedKey
4. **Async Processing**: Step Functions for complex workflows
5. **Cold Start Optimization**: Provisioned concurrency for critical functions
6. **Connection Pooling**: Reuse DynamoDB connections across invocations
7. **Jitter Algorithm**: Exponential backoff with jitter for retries and timeouts

### Frontend Optimization

1. **Code Splitting**: Lazy load components and routes
2. **Bundle Optimization**: Minimize JavaScript bundle size
3. **Image Optimization**: Compress and lazy load images
4. **Caching Strategy**: Browser and service worker caching

### Monitoring

1. **CloudWatch**: Lambda metrics, logs, and custom dashboards
2. **X-Ray**: Distributed tracing for request flow analysis
3. **DynamoDB Metrics**: Read/write capacity and throttling monitoring
4. **API Gateway Metrics**: Request latency and error rates
5. **Cost Monitoring**: AWS Cost Explorer and billing alerts
6. **User Analytics**: CloudWatch RUM for frontend performance
## BITA
CORA Flight Log Format

### Required Sections

The flight log must follow the standard BITACORA format with the following sections:

#### Header Information
- Flight type: Comercial, Entrenamiento, Comprobación, Otros
- Aircraft type and registration (matricula)
- Correlative number

#### Flight Data Section
- Route (From/To airports)
- Flight number and date
- Engine times (on/off)
- Flight times (takeoff/landing)
- Total flight time and cycles

#### Engine Parameters Recording
Multiple readings during flight of:
- OAT (Outside Air Temperature)
- Battery Voltage
- Start ITT (Interstage Turbine Temperature)
- Altitude
- Velocity
- Torque
- Propeller RPM
- ITT
- NG (Gas Generator Speed)
- Oil Pressure
- Oil Temperature
- Fuel Flow

#### Maintenance Section
- Observations field for general notes
- Discrepancies table with Item, ATA code, and description
- Corrective actions taken

#### Signatures and Approvals
- Maintenance release certification
- Pilot acceptance with captain and copilot information
- Electronic signatures and timestamps
- License numbers and dates

### Data Validation Rules

1. **Flight Times**: Takeoff must be after engine on, landing before engine off
2. **Engine Parameters**: All readings must be within acceptable ranges for aircraft type
3. **Cycles**: Automatically calculated based on takeoff/landing events
4. **Signatures**: Required fields for maintenance release and pilot acceptance
5. **Correlative Number**: Must be unique and sequential## Saa
S Multi-Tenancy Design

### Organization Isolation

Each airline customer is treated as a separate organization with complete data isolation:

- **Partition Key Strategy**: All tables use `org_id` as partition key or part of composite key
- **Data Access Control**: Lambda functions validate organization membership before data access
- **Resource Isolation**: Each organization's data is logically separated in DynamoDB

### Subscription Management

#### Subscription Tiers

1. **Basic Plan** ($99/month)
   - Up to 5 aircraft
   - Up to 20 pilots
   - 100 flights per month
   - Basic reporting

2. **Professional Plan** ($299/month)
   - Up to 25 aircraft
   - Up to 100 pilots
   - 500 flights per month
   - Advanced reporting and analytics
   - API access

3. **Enterprise Plan** ($799/month)
   - Unlimited aircraft and pilots
   - Unlimited flights
   - Custom integrations
   - Priority support
   - White-label options

#### Feature Gating

```python
def check_subscription_limits(org_id, resource_type, action):
    """
    Validates if organization can perform action based on subscription
    """
    org = get_organization(org_id)
    subscription = org['subscription_type']
    
    limits = {
        'basic': {'aircraft': 5, 'pilots': 20, 'monthly_flights': 100},
        'professional': {'aircraft': 25, 'pilots': 100, 'monthly_flights': 500},
        'enterprise': {'aircraft': -1, 'pilots': -1, 'monthly_flights': -1}  # Unlimited
    }
    
    if action == 'create' and resource_type in limits[subscription]:
        current_count = get_resource_count(org_id, resource_type)
        limit = limits[subscription][resource_type]
        
        if limit != -1 and current_count >= limit:
            raise SubscriptionLimitExceeded(f"Limit reached for {resource_type}")
    
    return True
```

### Billing Integration

- **Payment Processing**: Stripe integration for subscription billing
- **Usage Tracking**: Monitor resource usage for billing calculations
- **Automatic Provisioning**: Instant access upon successful payment
- **Dunning Management**: Handle failed payments and account suspension

### AWS Cost Optimization

1. **Lambda Optimization**
   - Use provisioned concurrency for frequently accessed functions
   - Optimize memory allocation based on usage patterns
   - Implement connection pooling for DynamoDB

2. **DynamoDB Optimization**
   - Use on-demand billing for variable workloads
   - Implement TTL for temporary data
   - Optimize partition key distribution

3. **Storage Optimization**
   - Use S3 for file storage (flight logs, documents)
   - Implement lifecycle policies for data archival
   - Compress large payloads

## Multi-Organization Pilot Management

### Pilot Registration and Assignment

#### Global Pilot Registry
- Pilots are registered once in the global Pilots table
- Each pilot has a unique pilot_id and email address
- Pilot profile includes personal information, qualifications, and total flight hours

#### Organization Assignment Process
1. **New Pilot Creation**: Organization creates a new pilot profile
2. **Existing Pilot Invitation**: Organization invites existing pilot by email/license
3. **Pilot Acceptance**: Pilot accepts invitation to join organization
4. **Role Assignment**: Organization assigns role (Captain, First Officer, etc.)

#### Multi-Organization Flight Hours Tracking
```python
def update_pilot_hours(pilot_id, org_id, flight_hours):
    """
    Updates both organization-specific and total pilot hours
    """
    # Update organization-specific hours
    pilot_org = get_pilot_organization(pilot_id, org_id)
    pilot_org['monthly_hours'] = update_monthly_hours(pilot_org['monthly_hours'], flight_hours)
    
    # Update global total hours
    pilot = get_pilot(pilot_id)
    pilot['total_hours'] += flight_hours
    
    # Save both records
    save_pilot_organization(pilot_org)
    save_pilot(pilot)
```

#### Access Control and Data Isolation
- Pilots can only see flight data for organizations they belong to
- Organizations can only assign their own pilots to flights
- Pilot personal information is shared across organizations (with consent)
- Flight hours are tracked per organization and globally

#### Subscription Impact on Pilot Limits
- Pilot limits apply to active pilot-organization relationships
- Same pilot working for multiple organizations counts toward each organization's limit
- Organizations pay for their active pilot slots regardless of pilot sharing

### Business Rules

1. **Pilot Availability**: Pilots can be assigned to flights only in organizations they belong to
2. **Hour Tracking**: Flight hours accumulate both per-organization and globally
3. **Qualification Sharing**: Pilot qualifications are global and shared across organizations
4. **Data Privacy**: Organizations cannot see pilot activity in other organizations
5. **Billing**: Each organization pays for their pilot slots independently

## Resilience and Retry Strategies

### Jitter Algorithm Implementation

To prevent thundering herd problems and improve system resilience, the application implements exponential backoff with jitter for all retry operations.

#### Exponential Backoff with Full Jitter
```python
import random
import time

def exponential_backoff_with_jitter(attempt, base_delay=1, max_delay=60):
    """
    Calculate delay with full jitter to prevent thundering herd
    """
    # Exponential backoff: base_delay * 2^attempt
    exponential_delay = min(base_delay * (2 ** attempt), max_delay)
    
    # Full jitter: random value between 0 and exponential_delay
    jittered_delay = random.uniform(0, exponential_delay)
    
    return jittered_delay

def retry_with_jitter(func, max_attempts=3, base_delay=1):
    """
    Retry function with exponential backoff and jitter
    """
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise e
            
            delay = exponential_backoff_with_jitter(attempt, base_delay)
            time.sleep(delay)
    
    return None
```

#### Use Cases for Jitter Algorithm

1. **DynamoDB Throttling**: Retry failed requests with jitter to avoid coordinated retries
2. **Lambda Timeouts**: Stagger function invocations to prevent simultaneous cold starts
3. **API Rate Limiting**: Distribute retry attempts across time to respect rate limits
4. **External Service Calls**: Prevent overwhelming third-party services during outages
5. **Batch Processing**: Stagger batch job executions to smooth resource usage

#### Implementation in Lambda Functions

```python
# DynamoDB operations with jitter
@retry_with_jitter(max_attempts=3, base_delay=0.1)
def get_pilot_with_retry(pilot_id):
    return dynamodb.get_item(
        TableName='Pilots',
        Key={'pilot_id': {'S': pilot_id}}
    )

# API Gateway timeout handling
def lambda_handler(event, context):
    # Calculate remaining time with jitter buffer
    remaining_time = context.get_remaining_time_in_millis()
    jitter_buffer = random.uniform(500, 1000)  # 0.5-1 second buffer
    
    if remaining_time < jitter_buffer:
        return {
            'statusCode': 504,
            'body': json.dumps({'error': 'Request timeout with jitter protection'})
        }
    
    # Continue with normal processing
    return process_request(event)
```

#### Circuit Breaker Pattern with Jitter

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
    
    def call(self, func):
        if self.state == 'OPEN':
            # Add jitter to recovery timeout
            jittered_timeout = self.recovery_timeout + random.uniform(0, 10)
            if time.time() - self.last_failure_time > jittered_timeout:
                self.state = 'HALF_OPEN'
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func()
            if self.state == 'HALF_OPEN':
                self.state = 'CLOSED'
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = 'OPEN'
            
            raise e
```

### Timeout Configuration

#### Lambda Function Timeouts
- **API Functions**: 30 seconds with 5-second jitter buffer
- **Data Processing**: 5 minutes with 30-second jitter buffer  
- **Report Generation**: 15 minutes with 2-minute jitter buffer

#### DynamoDB Timeouts
- **Read Operations**: 5 seconds with exponential backoff
- **Write Operations**: 10 seconds with exponential backoff
- **Batch Operations**: 30 seconds with jitter between batch items

#### Frontend Timeouts
- **API Calls**: 30 seconds with automatic retry
- **File Uploads**: 5 minutes with progress indication
- **Real-time Updates**: 10 seconds with reconnection jitter

## Role-Based Access Control (RBAC)

### User Roles and Permissions

#### 1. Sudo Role (System Administrator)
**Scope**: Global system access
**Permissions**:
- Manage all organizations and their data
- View system-wide analytics and usage
- Manage subscription plans and billing
- Access all user accounts and roles
- System configuration and maintenance
- Override any access restrictions

#### 2. Administrator Role (Organization Admin)
**Scope**: Single organization
**Permissions**:
- Manage organization settings and billing
- Invite and manage users within organization
- Assign roles to organization members
- Full CRUD access to all organization data:
  - Pilots, Aircraft, Flights, Passengers, Flight Logs
- View organization analytics and reports
- Export all organization data

#### 3. Pilot Role
**Scope**: Organization member with flight operations focus
**Permissions**:
- View assigned flights and aircraft
- Create and edit BITACORA flight logs for assigned flights
- Update flight status and times
- View pilot profiles and qualifications
- Read-only access to passenger manifests for assigned flights
- Cannot manage other pilots or aircraft

#### 4. Counter Role (Ground Operations)
**Scope**: Organization member with passenger operations focus
**Permissions**:
- Manage passenger manifests for all flights
- Add, edit, and remove passengers
- Generate passenger reports and manifests
- View flight schedules and aircraft assignments
- Read-only access to pilot information
- Cannot edit flight logs or aircraft data

### Permission Matrix

| Resource | Sudo | Administrator | Pilot | Counter |
|----------|------|---------------|-------|---------|
| Organizations | Full | Own Org Only | Read Own | Read Own |
| Users/Roles | Full | Own Org Only | Read Only | Read Only |
| Pilots | Full | Full | Read Only | Read Only |
| Aircraft | Full | Full | Read Only | Read Only |
| Flights | Full | Full | Assigned Only | Read Only |
| Passengers | Full | Full | Read Assigned | Full |
| Flight Logs | Full | Full | Edit Assigned | Read Only |
| Billing | Full | Own Org Only | None | None |
| Analytics | Full | Own Org Only | Limited | Limited |

### Access Control Implementation

#### Lambda Authorizer Function
```python
def lambda_authorizer(event, context):
    """
    Custom authorizer for API Gateway
    Validates JWT token and checks permissions
    """
    token = extract_token(event)
    user_claims = validate_cognito_token(token)
    
    # Get user role for the organization
    user_id = user_claims['sub']
    org_id = extract_org_from_path(event['path'])
    
    user_role = get_user_organization_role(user_id, org_id)
    
    # Check if user has permission for this resource/action
    resource = extract_resource(event['path'])
    action = event['httpMethod']
    
    if has_permission(user_role, resource, action, event):
        return generate_policy('Allow', event['methodArn'], {
            'user_id': user_id,
            'org_id': org_id,
            'role': user_role
        })
    else:
        return generate_policy('Deny', event['methodArn'])

def has_permission(role, resource, action, event):
    """
    Check if role has permission for resource/action
    """
    permissions = {
        'sudo': {'*': ['*']},  # Full access
        'administrator': {
            'pilots': ['GET', 'POST', 'PUT', 'DELETE'],
            'aircraft': ['GET', 'POST', 'PUT', 'DELETE'],
            'flights': ['GET', 'POST', 'PUT', 'DELETE'],
            'passengers': ['GET', 'POST', 'PUT', 'DELETE'],
            'flight_logs': ['GET', 'POST', 'PUT', 'DELETE'],
            'organization': ['GET', 'PUT'],
            'users': ['GET', 'POST', 'PUT', 'DELETE']
        },
        'pilot': {
            'pilots': ['GET'],
            'aircraft': ['GET'],
            'flights': ['GET', 'PUT'],  # Only assigned flights
            'passengers': ['GET'],  # Only assigned flights
            'flight_logs': ['GET', 'POST', 'PUT']  # Only assigned flights
        },
        'counter': {
            'pilots': ['GET'],
            'aircraft': ['GET'],
            'flights': ['GET'],
            'passengers': ['GET', 'POST', 'PUT', 'DELETE'],
            'flight_logs': ['GET']
        }
    }
    
    if role == 'sudo':
        return True
    
    if resource in permissions.get(role, {}):
        allowed_actions = permissions[role][resource]
        if action in allowed_actions:
            # Additional checks for resource-specific permissions
            return check_resource_specific_permissions(role, resource, event)
    
    return False

def check_resource_specific_permissions(role, resource, event):
    """
    Additional permission checks for specific resources
    """
    if role == 'pilot':
        if resource in ['flights', 'passengers', 'flight_logs']:
            # Pilots can only access their assigned flights
            flight_id = extract_flight_id(event['path'])
            user_id = event['requestContext']['authorizer']['user_id']
            return is_pilot_assigned_to_flight(user_id, flight_id)
    
    return True
```

#### Role Assignment API
```python
# User Management Endpoints
POST   /api/users/invite           # Invite user to organization (Administrator only)
PUT    /api/users/{id}/role        # Update user role (Administrator only)
DELETE /api/users/{id}/remove      # Remove user from org (Administrator only)
GET    /api/users/me               # Get current user info and permissions
GET    /api/users                  # List organization users (Administrator only)
```

### Role Hierarchy and Inheritance

1. **Sudo** > **Administrator** > **Pilot/Counter**
2. Higher roles can perform all actions of lower roles
3. Cross-organization access only for Sudo role
4. Role changes require Administrator or higher permission
5. Users can have different roles in different organizations

### Security Considerations

1. **Token Validation**: All requests validate Cognito JWT tokens
2. **Organization Isolation**: Strict validation of organization membership
3. **Resource-Level Security**: Additional checks for assigned flights/aircraft
4. **Audit Logging**: All role changes and sensitive actions logged
5. **Session Management**: Automatic token refresh and secure logout

## Infrastructure as Code with AWS CDK

### CDK Stack Architecture

The application infrastructure is organized into multiple CDK stacks for better separation of concerns and deployment flexibility:

#### 1. Core Infrastructure Stack (`CoreInfrastructureStack`)
- **DynamoDB Tables**: All application tables with proper indexes and encryption
- **Cognito User Pool**: Authentication and user management
- **IAM Roles and Policies**: Lambda execution roles and permissions
- **Parameter Store**: Configuration and secrets management

#### 2. API Stack (`ApiStack`)
- **API Gateway**: REST API with custom authorizers
- **Lambda Functions**: All business logic functions
- **Lambda Layers**: Shared dependencies and utilities
- **API Gateway Deployment**: Stages and throttling configuration

#### 3. Frontend Stack (`FrontendStack`)
- **AWS Amplify App**: Automated CI/CD pipeline and hosting for React app
- **Amplify Branch**: Environment-specific deployments (dev/staging/prod)
- **Custom Domain**: Domain configuration through Amplify
- **Build Settings**: Automated build and deployment configuration

#### 4. Monitoring Stack (`MonitoringStack`)
- **CloudWatch Dashboards**: Application and infrastructure metrics
- **CloudWatch Alarms**: Error rates, latency, and cost alerts
- **X-Ray Tracing**: Distributed tracing configuration
- **Log Groups**: Centralized logging with retention policies

### CDK Implementation Structure

```typescript
// Main CDK App
import * as cdk from 'aws-cdk-lib';
import { CoreInfrastructureStack } from './stacks/core-infrastructure-stack';
import { ApiStack } from './stacks/api-stack';
import { FrontendStack } from './stacks/frontend-stack';
import { MonitoringStack } from './stacks/monitoring-stack';

const app = new cdk.App();

// Environment configuration
const env = {
  account: process.env.CDK_DEFAULT_ACCOUNT,
  region: process.env.CDK_DEFAULT_REGION || 'us-east-2'  // Primary region: Ohio
};

// Core infrastructure (databases, auth, IAM)
const coreStack = new CoreInfrastructureStack(app, 'FlightDeckCore', { env });

// API and Lambda functions
const apiStack = new ApiStack(app, 'FlightDeckApi', {
  env,
  userPool: coreStack.userPool,
  dynamoTables: coreStack.dynamoTables,
  executionRole: coreStack.lambdaExecutionRole
});

// Frontend hosting with AWS Amplify
const frontendStack = new FrontendStack(app, 'FlightDeckFrontend', {
  env,
  apiGateway: apiStack.apiGateway,
  userPool: coreStack.userPool,
  repository: 'https://github.com/organization/flightdeck-frontend'  // GitHub repository for Amplify
});

// Monitoring and observability
const monitoringStack = new MonitoringStack(app, 'FlightDeckMonitoring', {
  env,
  apiGateway: apiStack.apiGateway,
  lambdaFunctions: apiStack.lambdaFunctions,
  dynamoTables: coreStack.dynamoTables
});
```

### DynamoDB Tables CDK Definition

```typescript
// Core Infrastructure Stack - DynamoDB Tables
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

export class CoreInfrastructureStack extends Stack {
  public readonly dynamoTables: { [key: string]: dynamodb.Table };

  constructor(scope: Construct, id: string, props: StackProps) {
    super(scope, id, props);

    // Organizations Table
    const organizationsTable = new dynamodb.Table(this, 'Organizations', {
      tableName: 'flightdeck-organizations',
      partitionKey: { name: 'org_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.ON_DEMAND,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      removalPolicy: RemovalPolicy.RETAIN
    });

    organizationsTable.addGlobalSecondaryIndex({
      indexName: 'subscription-status-index',
      partitionKey: { name: 'subscription_status', type: dynamodb.AttributeType.STRING }
    });

    // Pilots Table (Global Registry)
    const pilotsTable = new dynamodb.Table(this, 'Pilots', {
      tableName: 'flightdeck-pilots',
      partitionKey: { name: 'pilot_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.ON_DEMAND,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      removalPolicy: RemovalPolicy.RETAIN
    });

    pilotsTable.addGlobalSecondaryIndex({
      indexName: 'email-index',
      partitionKey: { name: 'email', type: dynamodb.AttributeType.STRING }
    });

    // PilotOrganizations Table
    const pilotOrganizationsTable = new dynamodb.Table(this, 'PilotOrganizations', {
      tableName: 'flightdeck-pilot-organizations',
      partitionKey: { name: 'pilot_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'org_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.ON_DEMAND,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      removalPolicy: RemovalPolicy.RETAIN
    });

    pilotOrganizationsTable.addGlobalSecondaryIndex({
      indexName: 'org-pilots-index',
      partitionKey: { name: 'org_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'pilot_id', type: dynamodb.AttributeType.STRING }
    });

    // Aircraft Table
    const aircraftTable = new dynamodb.Table(this, 'Aircraft', {
      tableName: 'flightdeck-aircraft',
      partitionKey: { name: 'org_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'aircraft_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.ON_DEMAND,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      removalPolicy: RemovalPolicy.RETAIN
    });

    aircraftTable.addGlobalSecondaryIndex({
      indexName: 'registration-index',
      partitionKey: { name: 'registration', type: dynamodb.AttributeType.STRING }
    });

    // Flights Table
    const flightsTable = new dynamodb.Table(this, 'Flights', {
      tableName: 'flightdeck-flights',
      partitionKey: { name: 'org_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'flight_id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.ON_DEMAND,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
      removalPolicy: RemovalPolicy.RETAIN
    });

    flightsTable.addGlobalSecondaryIndex({
      indexName: 'flight-number-index',
      partitionKey: { name: 'flight_number', type: dynamodb.AttributeType.STRING }
    });

    flightsTable.addGlobalSecondaryIndex({
      indexName: 'date-index',
      partitionKey: { name: 'org_id', type: dynamodb.AttributeType.STRING },
      sortKey: { name: 'departure_date', type: dynamodb.AttributeType.STRING }
    });

    // Store table references
    this.dynamoTables = {
      organizations: organizationsTable,
      pilots: pilotsTable,
      pilotOrganizations: pilotOrganizationsTable,
      aircraft: aircraftTable,
      flights: flightsTable
    };
  }
}
```

### Lambda Functions CDK Definition

```typescript
// API Stack - Lambda Functions
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';

export class ApiStack extends Stack {
  public readonly apiGateway: apigateway.RestApi;
  public readonly lambdaFunctions: { [key: string]: lambda.Function };

  constructor(scope: Construct, id: string, props: ApiStackProps) {
    super(scope, id, props);

    // Shared Lambda layer for common dependencies
    const sharedLayer = new lambda.LayerVersion(this, 'SharedLayer', {
      code: lambda.Code.fromAsset('lambda-layers/shared'),
      compatibleRuntimes: [lambda.Runtime.PYTHON_3_11],
      description: 'Shared utilities and dependencies'
    });

    // Lambda function configuration
    const lambdaConfig = {
      runtime: lambda.Runtime.PYTHON_3_11,
      timeout: Duration.seconds(30),
      memorySize: 512,
      layers: [sharedLayer],
      environment: {
        ORGANIZATIONS_TABLE: props.dynamoTables.organizations.tableName,
        PILOTS_TABLE: props.dynamoTables.pilots.tableName,
        AIRCRAFT_TABLE: props.dynamoTables.aircraft.tableName,
        FLIGHTS_TABLE: props.dynamoTables.flights.tableName,
        USER_POOL_ID: props.userPool.userPoolId
      },
      role: props.executionRole
    };

    // Create Lambda functions
    const pilotsHandler = new lambda.Function(this, 'PilotsHandler', {
      ...lambdaConfig,
      code: lambda.Code.fromAsset('lambda/pilots'),
      handler: 'handler.main'
    });

    const aircraftHandler = new lambda.Function(this, 'AircraftHandler', {
      ...lambdaConfig,
      code: lambda.Code.fromAsset('lambda/aircraft'),
      handler: 'handler.main'
    });

    const flightsHandler = new lambda.Function(this, 'FlightsHandler', {
      ...lambdaConfig,
      code: lambda.Code.fromAsset('lambda/flights'),
      handler: 'handler.main'
    });

    // Grant DynamoDB permissions
    Object.values(props.dynamoTables).forEach(table => {
      table.grantReadWriteData(pilotsHandler);
      table.grantReadWriteData(aircraftHandler);
      table.grantReadWriteData(flightsHandler);
    });

    // API Gateway with custom authorizer
    const authorizerHandler = new lambda.Function(this, 'AuthorizerHandler', {
      ...lambdaConfig,
      code: lambda.Code.fromAsset('lambda/authorizer'),
      handler: 'handler.main'
    });

    const authorizer = new apigateway.TokenAuthorizer(this, 'CustomAuthorizer', {
      handler: authorizerHandler,
      tokenSource: apigateway.IdentitySource.header('Authorization')
    });

    // Create API Gateway
    this.apiGateway = new apigateway.RestApi(this, 'FlightDeckApi', {
      restApiName: 'FlightDeck API',
      description: 'API for FlightDeck airline management system',
      defaultCorsPreflightOptions: {
        allowOrigins: apigateway.Cors.ALL_ORIGINS,
        allowMethods: apigateway.Cors.ALL_METHODS,
        allowHeaders: ['Content-Type', 'Authorization']
      }
    });

    // API Routes
    const pilotsResource = this.apiGateway.root.addResource('pilots');
    pilotsResource.addMethod('GET', new apigateway.LambdaIntegration(pilotsHandler), {
      authorizer
    });
    pilotsResource.addMethod('POST', new apigateway.LambdaIntegration(pilotsHandler), {
      authorizer
    });

    this.lambdaFunctions = {
      pilots: pilotsHandler,
      aircraft: aircraftHandler,
      flights: flightsHandler,
      authorizer: authorizerHandler
    };
  }
}
```

### Environment Management

```typescript
// CDK Context for different environments
{
  "environments": {
    "development": {
      "account": "123456789012",
      "region": "us-east-2",
      "domainName": "dev.flightdeck.com",
      "certificateArn": "arn:aws:acm:us-east-2:123456789012:certificate/dev-cert"
    },
    "staging": {
      "account": "123456789012", 
      "region": "us-east-2",
      "domainName": "staging.flightdeck.com",
      "certificateArn": "arn:aws:acm:us-east-2:123456789012:certificate/staging-cert"
    },
    "production": {
      "account": "987654321098",
      "region": "us-east-2", 
      "domainName": "flightdeck.com",
      "certificateArn": "arn:aws:acm:us-east-2:987654321098:certificate/prod-cert"
    }
  }
}
```

### AWS Amplify Frontend Stack

```typescript
// Frontend Stack with AWS Amplify
import * as amplify from 'aws-cdk-lib/aws-amplify';
import * as iam from 'aws-cdk-lib/aws-iam';

export class FrontendStack extends Stack {
  constructor(scope: Construct, id: string, props: FrontendStackProps) {
    super(scope, id, props);

    // Amplify App for React frontend
    const amplifyApp = new amplify.CfnApp(this, 'FlightDeckAmplifyApp', {
      name: 'FlightDeck',
      description: 'FlightDeck Airline Management System Frontend',
      repository: props.repository,
      platform: 'WEB',
      environmentVariables: [
        {
          name: 'REACT_APP_API_URL',
          value: props.apiGateway.url
        },
        {
          name: 'REACT_APP_USER_POOL_ID',
          value: props.userPool.userPoolId
        },
        {
          name: 'REACT_APP_USER_POOL_CLIENT_ID',
          value: props.userPool.userPoolClientId
        },
        {
          name: 'REACT_APP_REGION',
          value: 'us-east-2'
        }
      ],
      buildSpec: `
        version: 1
        applications:
          - frontend:
              phases:
                preBuild:
                  commands:
                    - npm ci
                build:
                  commands:
                    - npm run build
              artifacts:
                baseDirectory: dist
                files:
                  - '**/*'
              cache:
                paths:
                  - node_modules/**/*
            appRoot: frontend
      `
    });

    // Development branch
    const devBranch = new amplify.CfnBranch(this, 'DevBranch', {
      appId: amplifyApp.attrAppId,
      branchName: 'development',
      stage: 'DEVELOPMENT',
      enableAutoBuild: true,
      environmentVariables: [
        {
          name: 'REACT_APP_ENVIRONMENT',
          value: 'development'
        }
      ]
    });

    // Production branch
    const prodBranch = new amplify.CfnBranch(this, 'ProdBranch', {
      appId: amplifyApp.attrAppId,
      branchName: 'main',
      stage: 'PRODUCTION',
      enableAutoBuild: true,
      environmentVariables: [
        {
          name: 'REACT_APP_ENVIRONMENT',
          value: 'production'
        }
      ]
    });

    // Custom domain (optional)
    if (props.domainName) {
      new amplify.CfnDomain(this, 'FlightDeckDomain', {
        appId: amplifyApp.attrAppId,
        domainName: props.domainName,
        subDomainSettings: [
          {
            branchName: prodBranch.branchName,
            prefix: ''
          },
          {
            branchName: devBranch.branchName,
            prefix: 'dev'
          }
        ]
      });
    }
  }
}
```

### CDK Deployment Commands

```bash
# Install dependencies
npm install -g aws-cdk
npm install

# Bootstrap CDK in us-east-2 region (first time only)
cdk bootstrap --region us-east-2

# Synthesize CloudFormation templates
cdk synth

# Deploy to development (us-east-2)
cdk deploy --all --context environment=development --region us-east-2

# Deploy to production (us-east-2)
cdk deploy --all --context environment=production --region us-east-2

# Destroy resources (development only)
cdk destroy --all --context environment=development --region us-east-2
```

### Benefits of CDK Infrastructure as Code

1. **Version Control**: All infrastructure changes tracked in Git
2. **Reproducible Deployments**: Consistent environments across dev/staging/prod
3. **Type Safety**: TypeScript provides compile-time validation
4. **Resource Management**: Automatic dependency resolution and cleanup
5. **Cost Optimization**: Easy resource tagging and cost allocation
6. **Security**: Consistent IAM policies and security configurations
7. **Rollback Capability**: CloudFormation stack rollback on failures
8. **Documentation**: Infrastructure is self-documenting through code
##
 Application Branding

### FlightDeck Brand Identity

**FlightDeck** serves as the comprehensive command center for airline operations, embodying the concept of a flight deck where pilots have complete control and visibility over their aircraft systems.

#### Brand Positioning
- **Primary Name**: FlightDeck
- **Tagline**: "Your Airline Command Center"
- **Domain Strategy**: 
  - Production: `flightdeck.com`
  - Staging: `staging.flightdeck.com`
  - Development: `dev.flightdeck.com`

#### Technical Branding
- **AWS Resource Naming**: All AWS resources prefixed with `flightdeck-`
- **Database Tables**: `flightdeck-organizations`, `flightdeck-pilots`, etc.
- **CDK Stack Names**: `FlightDeckCore`, `FlightDeckApi`, `FlightDeckFrontend`, `FlightDeckMonitoring`
- **API Gateway**: "FlightDeck API"
- **Application Title**: "FlightDeck - Airline Management System"

#### User Interface Branding
- **Navigation**: FlightDeck logo and branding throughout the interface
- **Page Titles**: All pages prefixed with "FlightDeck -"
- **Email Communications**: Branded with FlightDeck identity
- **Documentation**: All references use FlightDeck naming

#### Brand Values
- **Control**: Complete command over airline operations
- **Visibility**: Clear insight into all flight operations
- **Reliability**: Dependable system for critical aviation operations
- **Compliance**: Meeting all regulatory requirements
- **Efficiency**: Streamlined operations for maximum productivity