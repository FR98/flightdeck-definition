# Requirements Document

## Introduction

FlightDeck is a comprehensive web application designed to manage multiple aircraft operations for a commercial airline. The system handles pilot management, aircraft fleet management, flight operations, passenger manifests, and flight logs. It supports passenger flight operations with complete tracking from flight planning to completion, including regulatory compliance through standardized flight log formats.

## Glossary

- **System**: FlightDeck - The airline management web application built with Python AWS Lambda backend and React frontend
- **Pilot**: A certified aircraft operator employed by the airline
- **Aircraft**: A commercial airplane owned or operated by the airline
- **Flight**: A scheduled passenger service between two airports
- **Passenger_Manifest**: A complete list of passengers for a specific flight
- **Flight_Log**: A detailed record of flight operations following BITACORA format standards
- **Flight_Hours**: Accumulated time spent operating aircraft, tracked monthly and total
- **Aircraft_Registration**: Unique identifier (matricula) assigned to each aircraft
- **Flight_Number**: Unique identifier assigned to each scheduled flight
- **Captain**: The pilot-in-command responsible for flight operations
- **Route**: The planned path between departure and arrival airports

## Requirements

### Requirement 1

**User Story:** As an airline administrator, I want to manage pilot information and qualifications, so that I can ensure proper crew assignment and regulatory compliance.

#### Acceptance Criteria

1. THE FlightDeck_System SHALL store pilot personal information including first name, last name, and age
2. THE FlightDeck_System SHALL maintain pilot qualification records including all current certifications
3. THE FlightDeck_System SHALL track monthly flight hours for each pilot
4. THE FlightDeck_System SHALL calculate and display total accumulated flight hours for each pilot
5. THE FlightDeck_System SHALL validate that pilot data is complete before allowing flight assignments

### Requirement 2

**User Story:** As an airline administrator, I want to manage aircraft fleet information, so that I can track aircraft capabilities and assign appropriate aircraft to flights.

#### Acceptance Criteria

1. THE FlightDeck_System SHALL store aircraft manufacturer information
2. THE FlightDeck_System SHALL maintain unique aircraft registration numbers for each aircraft
3. THE FlightDeck_System SHALL record maximum takeoff weight for each aircraft
4. THE FlightDeck_System SHALL specify required crew size for each aircraft type
5. THE FlightDeck_System SHALL prevent duplicate aircraft registrations in the fleet

### Requirement 3

**User Story:** As a flight operations manager, I want to create and manage flight schedules, so that I can coordinate passenger services and aircraft utilization.

#### Acceptance Criteria

1. THE FlightDeck_System SHALL assign unique flight numbers to each scheduled flight
2. THE FlightDeck_System SHALL define flight routes with departure and arrival locations
3. THE FlightDeck_System SHALL link each flight to a specific aircraft from the fleet
4. THE FlightDeck_System SHALL record scheduled departure and estimated arrival times
5. THE FlightDeck_System SHALL track actual arrival times when flights are completed
6. THE FlightDeck_System SHALL assign a captain from qualified pilots to each flight
7. THE FlightDeck_System SHALL calculate and store total flight weight including passengers and cargo

### Requirement 4

**User Story:** As a flight operations manager, I want to maintain passenger manifests for each flight, so that I can comply with aviation regulations and track passenger information.

#### Acceptance Criteria

1. THE FlightDeck_System SHALL link each passenger record to a specific flight
2. THE FlightDeck_System SHALL store complete passenger names
3. THE FlightDeck_System SHALL record passenger identification numbers
4. THE FlightDeck_System SHALL maintain passenger nationality information
5. THE FlightDeck_System SHALL track individual passenger weights for weight and balance calculations

### Requirement 5

**User Story:** As a pilot, I want to create flight logs following BITACORA format standards, so that I can maintain required flight documentation and regulatory compliance.

#### Acceptance Criteria

1. THE FlightDeck_System SHALL generate flight logs using the BITACORA.xlsx format structure
2. THE FlightDeck_System SHALL populate flight log data from flight and aircraft information
3. THE FlightDeck_System SHALL allow pilots to complete operational details during flight
4. THE FlightDeck_System SHALL maintain flight log records for regulatory audit purposes
5. THE FlightDeck_System SHALL ensure flight logs contain all required fields per aviation standards

### Requirement 6

**User Story:** As a system user, I want to access the airline management system through a modern web interface, so that I can efficiently perform my tasks with an intuitive user experience.

#### Acceptance Criteria

1. THE FlightDeck_System SHALL provide a responsive web interface built with React and Vite
2. THE FlightDeck_System SHALL use Tailwind CSS for consistent and modern styling
3. THE FlightDeck_System SHALL serve data through a Python AWS Lambda REST API backend
4. THE FlightDeck_System SHALL ensure cross-browser compatibility for modern web browsers
5. THE FlightDeck_System SHALL provide real-time data updates between frontend and backend