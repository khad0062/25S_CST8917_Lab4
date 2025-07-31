# CST8917 Lab 4 - Project Report: Real-Time Trip Event Analysis

## Executive Summary

This lab implements a real-time event-driven system for monitoring and analyzing taxi trip data. The solution leverages Azure Event Hub for data ingestion, Azure Functions for intelligent analysis, Logic Apps for workflow orchestration, and Microsoft Teams for instant notifications. The system successfully identifies suspicious patterns in trip data and alerts operations staff in real-time.

## Architecture Overview

### System Components

1. **Azure Event Hub (triipeventhub)**
   - Ingests trip events in JSON format
   - Configured with default consumer group
   - Batch processing with up to 10 events per pull

2. **Azure Logic App**
   - Polls Event Hub every minute
   - Orchestrates the analysis workflow
   - Routes notifications based on analysis results

3. **Azure Function (taxi-app-analyze_trip)**
   - Performs intelligent trip analysis
   - Identifies patterns and suspicious activities
   - Returns enriched data with insights

4. **Microsoft Teams Integration**
   - Receives Adaptive Card notifications
   - Three distinct card types for different scenarios
   - Direct chat integration with Flow bot

### Data Flow
```
Event Hub → Logic App Trigger → Azure Function → For Each Loop → Conditional Logic → Teams Notifications
```

## Logic App Workflow Details

### 1. Event Hub Trigger
```json
"When_events_are_available_in_Event_Hub": {
    "recurrence": {
        "interval": 1,
        "frequency": "Minute"
    },
    "queries": {
        "maximumEventsCount": 10,
        "consumerGroupName": "$Default"
    }
}
```

- **Frequency**: Polls every minute for new events
- **Batch Size**: Processes up to 10 events per execution
- **Content Type**: JSON format for structured data

### 2. Azure Function Call
```json
"taxi-app-analyze_trip": {
    "type": "Function",
    "inputs": {
        "body": "@triggerBody()"
    }
}
```

- Passes the entire event batch to the function
- Function processes each trip and returns analysis results

### 3. For Each Loop Processing

Iterates through each analyzed trip result and implements nested conditional logic:

#### Primary Condition: `isItInteresting`
- **True Branch**: Further evaluates if it's suspicious
- **False Branch**: Posts "No Issues" notification

#### Secondary Condition: `IsSus` (Is Suspicious)
- **True Branch**: Posts "Suspicious Vendor Activity" card
- **False Branch**: Posts "Interesting Trip Detected" card

## Azure Function Logic Description

The Azure Function (`analyze_trip`) implements the following analysis logic:

### Input Processing
- Accepts batch of trip events from Event Hub
- Extracts trip data from ContentData property
- Handles both single events and arrays

### Analysis Rules

1. **Long Trip Detection**
   - **Condition**: `tripDistance > 10` miles
   - **Insight**: "LongTrip"

2. **Group Ride Identification**
   - **Condition**: `passengerCount > 4`
   - **Insight**: "GroupRide"

3. **Cash Payment Tracking**
   - **Condition**: `paymentType == "2"`
   - **Insight**: "CashPayment"

4. **Suspicious Vendor Activity**
   - **Condition**: `paymentType == "2" AND tripDistance < 1`
   - **Insight**: "SuspiciousVendorActivity"
   - **Purpose**: Flags potential fare manipulation

### Output Structure
```json
{
    "vendorID": "string",
    "tripDistance": "number",
    "passengerCount": "number",
    "paymentType": "string",
    "insights": ["array of insights"],
    "isInteresting": "boolean",
    "summary": "string description",
    "isSuspicious": "boolean"
}
```

## Microsoft Teams Integration

### Adaptive Card Types

1. **No Issues Card**
   - Posted when `isItInteresting = false`
   - Simple status notification
   - Green color scheme

2. **Interesting Trip Detected Card**
   - Posted when `isItInteresting = true` and `isSuspicious = false`
   - Contains trip details and insights
   - Yellow/orange color scheme

3. **Suspicious Vendor Activity Card**
   - Posted when `isItInteresting = true` and `isSuspicious = true`
   - High-priority alert format
   - Red color scheme
   - Includes escalation options

### Card Content Structure
Each card includes:
- Trip identification (Vendor ID)
- Key metrics (distance, passengers, payment type)
- Analysis insights
- Timestamp
- Action buttons (for suspicious activities)

## Technical Implementation Details

### Event Hub Configuration
- **Partition Count**: Default (typically 4)
- **Message Retention**: 1 day (minimum)
- **Consumer Group**: $Default
- **Event Format**: JSON with ContentData wrapper

### Logic App Configuration
- **Trigger Type**: Event Hub polling
- **Polling Interval**: 1 minute
- **Batch Processing**: Up to 10 events
- **Error Handling**: Built-in retry policies

### Azure Function Configuration
- **Runtime**: Python runtime
- **Trigger Type**: HTTP trigger
- **Authentication**: Function key
- **Timeout**: 5 minutes
- **Memory**: 512 MB
<img width="857" height="708" alt="image" src="https://github.com/user-attachments/assets/c10cac54-d73a-46ed-8814-7665b3b91c48" />
