# Therapist Matcher API

A Flask-based REST API server that facilitates the matching process between patients and therapists by managing appointment scheduling, therapist suggestions, and automated email communications.

## Overview

The Therapist Matcher API simulates a therapy appointment booking system that:
- Sends therapist suggestions to patients based on availability preferences
- Automatically selects therapists and schedules appointments for the following week
- Simulates email notifications to both patients and therapists
- Provides therapist response simulation with weighted probability

## Features

- **Therapist Suggestion Management**: Send lists of available therapists to patients
- **Automated Selection**: Randomly select a therapist from the provided list with datetime calculation
- **Email Simulation**: Mock email system for appointment notifications
- **Weighted Response System**: Simulates therapist availability responses (70% acceptance rate)
- **Flexible Scheduling**: Supports morning/afternoon slots across all weekdays

## Technology Stack

- **Python 3.11**
- **Flask 2.3.3**
- **Datetime** for scheduling calculations
- **Random** for weighted selections

## Setup

This is a Deepnote notebook. To run the server:

1. **Open the notebook in Deepnote:**
   
   [Open Therapist Matcher API Server Notebook](https://deepnote.com/workspace/DHP25-244a274b-59d3-442f-b7ef-3d5d24503cee/project/chocolatexpress-fbdcce36-fd51-4cfb-8676-e6e544158098/notebook/APIServer-cc5e3854092d45b89dd08990f5fce491?secondary-sidebar-autoopen=true&secondary-sidebar=agent)

2. Run all cells sequentially
3. The server will start on `http://0.0.0.0:8080`

All dependencies are pre-installed in the Deepnote environment.

## API Endpoints

### Base URL
```
/therapist-matcher
```

### 1. Send Therapist Suggestions
**Endpoint:** `POST /therapist-matcher/therapists-suggestion`

Sends a list of available therapists to a patient for a specific weekday and time slot.

**Request Body:**
```json
{
  "patient": "Mariah Carey",
  "therapistList": ["Dr. Jordan", "Dr. Smith", "Dr. John"],
  "weekday": "wednesday",
  "slot": "afternoon"
}
```

**Parameters:**
- `patient` (string, required): Patient's full name
- `therapistList` (array, required): List of available therapist names
- `weekday` (string, required): Day of the week (monday-sunday)
- `slot` (string, required): Time slot ("morning" = 8:00, "afternoon" = 14:00)

**Response:**
```json
{
  "status": "success",
  "message": "Message sent successfully."
}
```

**Status Codes:**
- `200`: Success
- `400`: Invalid payload

---

### 2. Get Selection
**Endpoint:** `GET /therapist-matcher/selection`

Returns a randomly selected therapist from the previously submitted list along with the calculated appointment datetime for the next occurrence of the specified weekday.

**Response:**
```json
{
  "therapist": "Dr. Jordan",
  "datetime": "2025-12-17T14:00:00"
}
```

**Parameters:**
- `therapist` (string): Selected therapist name
- `datetime` (string): ISO 8601 formatted datetime for next week's appointment

**Note:** Introduces a 1-second delay to simulate processing time.

---

### 3. Send Appointment Message
**Endpoint:** `POST /therapist-matcher/send-message`

Sends appointment confirmation emails to both the patient and the selected therapist.

**Request Body:**
```json
{
  "patient": "Mariah Carey",
  "therapist": "Dr. Jordan",
  "datetime": "2025-12-17T14:00:00"
}
```

**Parameters:**
- `patient` (string, required): Patient's full name
- `therapist` (string, required): Therapist's full name
- `datetime` (string, required): ISO 8601 formatted appointment datetime

**Response:**
```json
{
  "status": "success",
  "message": "Message sent successfully."
}
```

**Status Codes:**
- `200`: Success
- `400`: Invalid payload

**Note:** Introduces a 1-second delay to simulate email sending.

---

### 4. Get Therapist Response
**Endpoint:** `GET /therapist-matcher/therapist-response`

Simulates therapist's response to an appointment request with weighted probability.

**Response:**
```
yes
```
or
```
no
```

**Probability:**
- 70% chance of "yes"
- 30% chance of "no"

**Note:** Introduces a 1-second delay to simulate response time.

---

### 5. Health Check
**Endpoint:** `GET /therapist-matcher/`

Confirms the server is running.

**Response:**
```
The Therapist Matcher Server is running.
```

## Email System

The API includes a simulated email system that generates email addresses and logs all communications.

### Email Address Format
Email addresses are automatically generated from names:
- Format: `{lowercasename}@chocolat.com`
- Spaces and periods are removed
- Example: "Dr. Smith" → `drsmith@chocolat.com`

### Email Logs
All simulated emails are printed to console with the following format:
```
[EMAIL SIMULATED]
to=mariahcarey@chocolat.com
subject=Therapy Appointment Details
body=2025-12-17T14:00:00
```

## Scheduling Logic

### Weekday Mapping
- Monday: 1
- Tuesday: 2
- Wednesday: 3
- Thursday: 4
- Friday: 5
- Saturday: 6
- Sunday: 7

### Time Slot Mapping
- Morning: 8:00 AM (08:00)
- Afternoon: 2:00 PM (14:00)

### Next Week Calculation
The system automatically calculates the next occurrence of the specified weekday:
- If the target weekday hasn't occurred this week, it schedules for this week
- If the target weekday has passed or is today, it schedules for next week
- Always schedules at least 7 days in advance if the current day matches the target

## Error Handling

### Validation Errors

The API validates all incoming payloads and returns descriptive error messages:

**Missing Required Fields:**
```json
{
  "status": "error",
  "message": "Payload does not contain necessary fields."
}
```

**Invalid Field Type:**
```json
{
  "status": "error",
  "message": "Field 'patient' must be a string."
}
```

**Invalid Weekday:**
```python
ValueError: Invalid weekday: {day_global}
```

## Configuration

### Global Variables

The following variables are set globally and updated through API calls:
- `day_global`: Current target weekday (default: 'monday')
- `slot_global`: Current target time slot (default: 'morning')
- `therapist_list_global`: Current list of available therapists (default: ['Dr. Smith'])

### Server Configuration

- **Host:** 0.0.0.0 (all interfaces)
- **Port:** 8080
- **Waiting Time:** 1 second (simulated processing delay)

## Development Notes

### Limitations
- Email system is mocked and only logs to console
- No persistent storage (state resets on server restart)
- No authentication or authorization
- Single-threaded processing

## Future Enhancements

- [ ] Implement actual email integration (SMTP)
- [ ] Add database support for persistent storage
- [ ] Implement user authentication
- [ ] Add appointment cancellation and rescheduling
- [ ] Support for multiple time slots per day
- [ ] Therapist availability calendar
- [ ] Patient history tracking
- [ ] Real-time notifications (WebSocket)

---

**Note:** This is a simulation/mock API intended for development and testing purposes. The email functionality is simulated and does not send actual emails.
