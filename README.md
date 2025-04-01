# OpenSearch Log Deletion via ISM Policy

## Overview
This repository contains details on how we automatically delete logs older than **30 days** in OpenSearch using **Index State Management (ISM)** policies.

## How It Works
- We delete **entire indices**, not individual logs.
- OpenSearch **does not support deleting logs inside an index directly** via ISM.
- Deleting an index **automatically deletes all logs** within it.

## Steps to Set Up ISM Policy
### 1. Go to Dev Tools in OpenSearch Dashboard
Use the following steps to create and apply the ISM policy.

### 2. Create ISM Policy
```json
PUT _opendistro/_ism/policies/test-30days-logs
{
  "policy": {
    "policy_id": "test-30days-logs",
    "description": "A policy to delete indices older than 30 days.",
    "default_state": "hot",
    "states": [
      {
        "name": "hot",
        "actions": [],
        "transitions": [
          {
            "state_name": "delete",
            "conditions": {
              "min_index_age": "30d"
            }
          }
        ]
      }
    ],
    "ism_template": [
      {
        "index_patterns": [
          "<index_pattern>-*"  //eg: fluentd-*
        ],
        "priority": 100
      }
    ]
  }
}
```

### 3. Apply Policy to Indices
```json
POST _opendistro/_ism/add/<index_pattern>-* //eg: fluentd-*
{
  "policy_id": "test-30days-logs"
}
```

### 4. Verify Policy is Applied
```json
GET _opendistro/_ism/policies/test-30days-logs
```

## Checking ISM Job Interval
To check how often OpenSearch runs ISM policies:
```sh
GET _cluster/settings?include_defaults=true
```
Look for:
```json
{
  "defaults": {
    "opendistro.index_state_management.job_interval": "30s"
  }
}
```
This means **indices older than 30 days will be deleted automatically** when the ISM job runs.

## Conclusion
- We **delete entire indices**, not individual logs.
- **Deleted indices cannot be recovered.**
- OpenSearch runs **ISM jobs periodically** based on the configured interval.
