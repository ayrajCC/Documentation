# NICE CXone Healthcare Integration API

## Overview

The NICE CXone Healthcare Integration API enables seamless integration between NICE CXone contact center platform and healthcare systems such as EHR (Electronic Health Records), patient portals, scheduling systems, and billing platforms. This API is designed specifically for healthcare organizations to create personalized and HIPAA-compliant patient experiences.

## Authentication

The CXone Healthcare Integration API uses OAuth 2.0 for authentication. All API requests must include a valid access token in the Authorization header.

### Obtaining an Access Token

```http
POST https://auth.cxone.com/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET
```

### Example Response

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "healthcare-api.read healthcare-api.write"
}
```

### Using the Access Token

```http
GET https://api.cxone.com/healthcare/v1/patients
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Base URL

All API endpoints are relative to:

```
https://api.cxone.com/healthcare/v1
```

## Rate Limiting

The API enforces rate limiting to ensure fair usage and system stability:

| Plan | Rate Limit |
|------|------------|
| Basic | 100 requests per minute |
| Professional | 500 requests per minute |
| Enterprise | 2000 requests per minute |

Rate limit information is included in the response headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1619723400
```

## HIPAA Compliance

All data transferred through this API is encrypted in transit using TLS 1.2+. The API endpoints are designed to be HIPAA-compliant and adhere to secure PHI (Protected Health Information) handling practices. Audit logging is enabled for all API interactions involving PHI.

## Patient Resource

The Patient resource represents a patient in the healthcare system and is the core object for integration with CXone telephony systems.

### Patient Object Structure

```json
{
  "id": "string",
  "mrn": "string",
  "firstName": "string",
  "lastName": "string",
  "dateOfBirth": "date",
  "gender": "string",
  "contactInfo": {
    "email": "string",
    "phoneNumber": "string",
    "alternatePhoneNumber": "string",
    "preferredContactMethod": "string"
  },
  "insuranceInfo": {
    "providerId": "string",
    "memberNumber": "string",
    "groupNumber": "string"
  },
  "preferredLanguage": "string",
  "communicationPreferences": {
    "allowEmail": boolean,
    "allowSMS": boolean,
    "allowVoice": boolean
  },
  "tags": [
    "string"
  ],
  "createdAt": "datetime",
  "updatedAt": "datetime"
}
```

### Get Patient by ID

Retrieves a patient by their unique identifier.

```http
GET /patients/{id}
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id | string | Yes | Unique patient identifier |

**Response:**

```json
{
  "id": "pt-12345",
  "mrn": "MRN-987654",
  "firstName": "John",
  "lastName": "Doe",
  "dateOfBirth": "1980-05-15",
  "gender": "male",
  "contactInfo": {
    "email": "john.doe@example.com",
    "phoneNumber": "+12025551234",
    "alternatePhoneNumber": "+12025556789",
    "preferredContactMethod": "phone"
  },
  "insuranceInfo": {
    "providerId": "INS-001",
    "memberNumber": "MEM-12345",
    "groupNumber": "GRP-6789"
  },
  "preferredLanguage": "en",
  "communicationPreferences": {
    "allowEmail": true,
    "allowSMS": true,
    "allowVoice": true
  },
  "tags": [
    "primary-care",
    "hypertension"
  ],
  "createdAt": "2023-01-15T12:30:45Z",
  "updatedAt": "2023-03-20T09:15:30Z"
}
```

### Search Patients

Search for patients based on various criteria.

```http
GET /patients
```

**Query Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| mrn | string | No | Medical Record Number |
| phoneNumber | string | No | Patient's phone number |
| email | string | No | Patient's email address |
| lastName | string | No | Patient's last name |
| dateOfBirth | date | No | Patient's date of birth (format: YYYY-MM-DD) |
| limit | integer | No | Maximum number of results to return (default: 20, max: 100) |
| offset | integer | No | Number of results to skip (default: 0) |

**Response:**

```json
{
  "total": 1,
  "offset": 0,
  "limit": 20,
  "data": [
    {
      "id": "pt-12345",
      "mrn": "MRN-987654",
      "firstName": "John",
      "lastName": "Doe",
      "dateOfBirth": "1980-05-15",
      "contactInfo": {
        "phoneNumber": "+12025551234",
        "preferredContactMethod": "phone"
      }
      // Other fields omitted for brevity
    }
  ]
}
```

## Appointment Resource

The Appointment resource represents scheduled appointments for patients.

### Appointment Object Structure

```json
{
  "id": "string",
  "patientId": "string",
  "providerId": "string",
  "locationId": "string",
  "appointmentType": "string",
  "status": "string",
  "startTime": "datetime",
  "endTime": "datetime",
  "notes": "string",
  "createdAt": "datetime",
  "updatedAt": "datetime",
  "reminderSent": boolean,
  "reminderStatus": "string"
}
```

### Get Upcoming Appointments by Patient ID

```http
GET /patients/{patientId}/appointments
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| patientId | string | Yes | Patient identifier |
| startDate | date | No | Filter appointments starting from this date (format: YYYY-MM-DD) |
| endDate | date | No | Filter appointments until this date (format: YYYY-MM-DD) |
| status | string | No | Filter by appointment status (scheduled, cancelled, completed) |

**Response:**

```json
{
  "total": 2,
  "data": [
    {
      "id": "appt-54321",
      "patientId": "pt-12345",
      "providerId": "dr-789",
      "locationId": "loc-456",
      "appointmentType": "follow-up",
      "status": "scheduled",
      "startTime": "2023-05-10T14:30:00Z",
      "endTime": "2023-05-10T15:00:00Z",
      "notes": "Follow-up for hypertension medication adjustment",
      "createdAt": "2023-04-25T10:15:30Z",
      "updatedAt": "2023-04-25T10:15:30Z",
      "reminderSent": true,
      "reminderStatus": "delivered"
    },
    {
      "id": "appt-98765",
      "patientId": "pt-12345",
      "providerId": "dr-123",
      "locationId": "loc-789",
      "appointmentType": "annual-physical",
      "status": "scheduled",
      "startTime": "2023-06-15T10:00:00Z",
      "endTime": "2023-06-15T11:00:00Z",
      "notes": "Annual physical examination",
      "createdAt": "2023-04-20T14:45:22Z",
      "updatedAt": "2023-04-20T14:45:22Z",
      "reminderSent": false,
      "reminderStatus": null
    }
  ]
}
```

### Schedule New Appointment

```http
POST /appointments
Content-Type: application/json
```

**Request Body:**

```json
{
  "patientId": "pt-12345",
  "providerId": "dr-789",
  "locationId": "loc-456",
  "appointmentType": "follow-up",
  "startTime": "2023-05-10T14:30:00Z",
  "endTime": "2023-05-10T15:00:00Z",
  "notes": "Follow-up for hypertension medication adjustment"
}
```

**Response:**

```json
{
  "id": "appt-54321",
  "patientId": "pt-12345",
  "providerId": "dr-789",
  "locationId": "loc-456",
  "appointmentType": "follow-up",
  "status": "scheduled",
  "startTime": "2023-05-10T14:30:00Z",
  "endTime": "2023-05-10T15:00:00Z",
  "notes": "Follow-up for hypertension medication adjustment",
  "createdAt": "2023-04-25T10:15:30Z",
  "updatedAt": "2023-04-25T10:15:30Z",
  "reminderSent": false,
  "reminderStatus": null
}
```

## CXone Integration Endpoints

These endpoints allow integration with NICE CXone contact center features and are specific to healthcare use cases.

### Create Patient Callback Request

Schedule a callback for a patient, creating a task in the CXone platform.

```http
POST /integrations/cxone/callbacks
Content-Type: application/json
```

**Request Body:**

```json
{
  "patientId": "pt-12345",
  "phoneNumber": "+12025551234",
  "callbackTime": "2023-05-02T15:30:00Z",
  "callbackReason": "prescription-refill",
  "priority": "medium",
  "notes": "Patient needs refill of hypertension medication",
  "skillGroupId": "skill-123",
  "attributes": {
    "patientMRN": "MRN-987654",
    "callbackAttempts": 0
  }
}
```

**Response:**

```json
{
  "id": "callback-9876",
  "patientId": "pt-12345",
  "status": "scheduled",
  "phoneNumber": "+12025551234",
  "callbackTime": "2023-05-02T15:30:00Z",
  "callbackReason": "prescription-refill",
  "priority": "medium",
  "notes": "Patient needs refill of hypertension medication",
  "skillGroupId": "skill-123",
  "attributes": {
    "patientMRN": "MRN-987654",
    "callbackAttempts": 0
  },
  "createdAt": "2023-05-01T09:45:22Z"
}
```

### Send Appointment Reminder

Send an automated appointment reminder via the CXone platform (SMS, email, or voice).

```http
POST /integrations/cxone/reminders
Content-Type: application/json
```

**Request Body:**

```json
{
  "appointmentId": "appt-54321",
  "reminderType": "sms",
  "reminderTime": "2023-05-09T14:30:00Z",
  "message": "Reminder: You have an appointment scheduled with Dr. Smith tomorrow at 2:30 PM. Reply C to confirm or R to reschedule.",
  "includeDirections": true
}
```

**Response:**

```json
{
  "id": "reminder-1234",
  "appointmentId": "appt-54321",
  "reminderType": "sms",
  "reminderTime": "2023-05-09T14:30:00Z",
  "status": "scheduled",
  "createdAt": "2023-05-01T10:00:15Z"
}
```

### Get Patient Contact History

Retrieve a patient's contact history from CXone platform.

```http
GET /integrations/cxone/patients/{patientId}/contacts
```

**Parameters:**

| Name | Type | Required | Description |
|------|------|----------|-------------|
| patientId | string | Yes | Patient identifier |
| startDate | date | No | Filter contacts from this date (format: YYYY-MM-DD) |
| endDate | date | No | Filter contacts until this date (format: YYYY-MM-DD) |
| channel | string | No | Filter by contact channel (voice, chat, email, sms) |
| limit | integer | No | Maximum number of results to return (default: 20, max: 100) |
| offset | integer | No | Number of results to skip (default: 0) |

**Response:**

```json
{
  "total": 2,
  "offset": 0,
  "limit": 20,
  "data": [
    {
      "id": "contact-123",
      "patientId": "pt-12345",
      "channel": "voice",
      "direction": "inbound",
      "agentId": "agent-456",
      "skillGroupId": "skill-123",
      "startTime": "2023-04-15T10:30:45Z",
      "endTime": "2023-04-15T10:45:22Z",
      "duration": 875,
      "disposition": "resolved",
      "notes": "Patient called regarding prescription refill. Request processed successfully.",
      "tags": ["prescription", "resolved"],
      "recording": {
        "id": "rec-789",
        "url": "https://recordings.cxone.com/rec-789",
        "duration": 875
      }
    },
    {
      "id": "contact-456",
      "patientId": "pt-12345",
      "channel": "sms",
      "direction": "outbound",
      "agentId": null,
      "skillGroupId": "skill-456",
      "startTime": "2023-04-09T14:15:30Z",
      "endTime": "2023-04-09T14:16:05Z",
      "duration": 35,
      "disposition": "automated-reminder",
      "notes": "Automated appointment reminder sent",
      "tags": ["appointment-reminder"],
      "transcript": "Reminder: You have an appointment with Dr. Smith on 04/10/2023 at 2:30 PM. Reply C to confirm or R to reschedule."
    }
  ]
}
```

## IVR Customization

These endpoints allow customization of Interactive Voice Response (IVR) flows for healthcare use cases.

### Get Available IVR Templates

```http
GET /integrations/cxone/ivr/templates
```

**Response:**

```json
{
  "data": [
    {
      "id": "ivr-temp-123",
      "name": "Healthcare Appointment Management",
      "description": "IVR flow for appointment scheduling, rescheduling, and cancellation",
      "category": "appointment",
      "createdAt": "2022-10-15T08:30:45Z"
    },
    {
      "id": "ivr-temp-456",
      "name": "Prescription Refill Request",
      "description": "IVR flow for prescription refill requests",
      "category": "pharmacy",
      "createdAt": "2022-11-20T14:45:30Z"
    },
    {
      "id": "ivr-temp-789",
      "name": "Patient Billing Inquiry",
      "description": "IVR flow for billing inquiries and payment processing",
      "category": "billing",
      "createdAt": "2023-01-10T11:15:22Z"
    }
  ]
}
```

### Create Custom IVR Flow

```http
POST /integrations/cxone/ivr/flows
Content-Type: application/json
```

**Request Body:**

```json
{
  "name": "Specialty Clinic Appointment Flow",
  "description": "Custom IVR flow for cardiology department appointment management",
  "baseTemplateId": "ivr-temp-123",
  "parameters": {
    "departmentName": "Cardiology",
    "collectInsuranceInfo": true,
    "offerCallbackOption": true,
    "maxCallbackDays": 3,
    "workingHoursStart": "08:00",
    "workingHoursEnd": "17:00",
    "workingDays": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
    "holidayDates": ["2023-05-29", "2023-07-04"],
    "greetingMessage": "Thank you for calling the cardiology department. Your call is important to us."
  },
  "integrations": {
    "ehrSystemId": "ehr-456",
    "schedulingSystemId": "schedule-789",
    "authenticationRequired": true
  }
}
```

**Response:**

```json
{
  "id": "ivr-flow-321",
  "name": "Specialty Clinic Appointment Flow",
  "description": "Custom IVR flow for cardiology department appointment management",
  "baseTemplateId": "ivr-temp-123",
  "status": "created",
  "createdAt": "2023-05-01T15:30:45Z",
  "updatedAt": "2023-05-01T15:30:45Z",
  "deploymentStatus": "pending"
}
```

## Webhooks

The CXone Healthcare API supports webhooks for real-time event notifications.

### Register Webhook

```http
POST /webhooks
Content-Type: application/json
```

**Request Body:**

```json
{
  "url": "https://your-healthcare-app.com/api/cxone-webhooks",
  "events": ["appointment.created", "appointment.canceled", "patient.updated", "contact.completed"],
  "description": "Production webhook for real-time appointment and contact updates",
  "secret": "your-webhook-secret"
}
```

**Response:**

```json
{
  "id": "webhook-123",
  "url": "https://your-healthcare-app.com/api/cxone-webhooks",
  "events": ["appointment.created", "appointment.canceled", "patient.updated", "contact.completed"],
  "description": "Production webhook for real-time appointment and contact updates",
  "status": "active",
  "createdAt": "2023-05-01T16:45:30Z"
}
```

### Webhook Payload Example

When an appointment is created, the following payload will be sent to your webhook URL:

```json
{
  "eventId": "evt-456789",
  "eventType": "appointment.created",
  "timestamp": "2023-05-01T17:30:45Z",
  "data": {
    "appointmentId": "appt-54321",
    "patientId": "pt-12345",
    "providerId": "dr-789",
    "appointmentType": "follow-up",
    "status": "scheduled",
    "startTime": "2023-05-10T14:30:00Z",
    "endTime": "2023-05-10T15:00:00Z"
  }
}
```

## Error Handling

The API uses standard HTTP status codes and returns error details in the response body.

### Error Response Format

```json
{
  "error": {
    "code": "invalid_parameter",
    "message": "Invalid parameter: patientId",
    "details": "Patient ID must be in the format 'pt-' followed by numbers",
    "requestId": "req-abc123"
  }
}
```

### Common Error Codes

| Code | Description |
|------|-------------|
| invalid_parameter | One or more parameters are invalid |
| resource_not_found | The requested resource was not found |
| authentication_failed | Authentication failed |
| authorization_failed | The authenticated user does not have permission |
| rate_limit_exceeded | Rate limit has been exceeded |
| internal_error | An internal server error occurred |

## Appendix: HIPAA Compliance Guidelines

When using the CXone Healthcare API, follow these guidelines to ensure HIPAA compliance:

1. Always use TLS 1.2+ for all API communications
2. Store PHI only as needed and in compliance with HIPAA requirements
3. Implement appropriate access controls for API credentials
4. Enable audit logging for all API interactions involving PHI
5. Implement timeout mechanisms for sessions
6. Use secure, temporary storage for any downloaded PHI
7. Implement data minimization practices

For complete HIPAA compliance documentation, refer to our [HIPAA Compliance Guide](https://docs.cxone.com/healthcare/hipaa-compliance-guide).
