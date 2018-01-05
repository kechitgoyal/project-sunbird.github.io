---
type: landing
directory: developer-docs/telemetry
title: Consuming Telemetry Data
page_title: Consuming Telemetry Data
description: Consuming Telemetry Data
published: true
allowSearch: true
---

## Telemetry Exhaust APIs

The Sunbird telemetry services are made available through a daily exhaust, supported by Ekstep infra from multiple channels. It is common that, these channels will not have the data pipeline to process the telemetry data. Hence, telemetry are made available to channels.

Steps involved in consuming Data Exhaust:

1. [Automated] A User need to register using [Register User API](https://github.com/ekstep/Common-Design/wiki/Data-Exhaust-API-Specification#data-exhaust-register-user-api). This API returns a license key, which needs to be used when requesting telemetry data. This creates an entry for the user in ```data_exhaust_users``` table. 
2. [Manual] Access needs to be granted, manually, to this user to the dataset resource that they want to consume. This access is granted by making an entry, for corresponding user and resource, in ```data_exhaust_users_resources``` table.
3. [Automated] User can then use [Datasets API](https://github.com/ekstep/Common-Design/wiki/Data-Exhaust-API-Specification#data-exhaust-dataset-api) to download telemetry data. User has to pass the license key, got in Step 1, for authentication. Internally, Datasets API will invoke [Authentication API](https://github.com/ekstep/Common-Design/wiki/Data-Exhaust-API-Specification#data-exhaust-authenticate-api) and [Authorisation API](https://github.com/ekstep/Common-Design/wiki/Data-Exhaust-API-Specification#data-exhaust-authorize-api) to check if the user has is valid and has access to the requested resource, if the checks pass, dataset is returned.

```Note```: Database schema of Data Exhaust is described [here](https://github.com/ekstep/Common-Design/wiki/TDD-DataSets#database-schema-changes).

### Using channel ID and API key to request channel telemetry

## Understanding typical workflows from Telemetry data

This section details **Standard Telemetry Workflows** for different access channels.

### Mobile App

<pre>
START(type: "app")
    ...| --> app events such as IMPRESSION, FEEDBACK, etc may happen
    START(type: "session")
        ...
        | --> IMPRESSION - For the pages that the user visits
        | --> INTERACT --> one of the content is clicked
            | --> START(type: "player") --> events generated by specific content
                ...|
                ...| --> in-content events such as ASSESS, INTERACT, IMPRESSION, LEVEL_SET etc.
                ...|
            | --> END(type: "player")
        | --> IMPRESSION - Returned back to mobile app for content player
        ...| --> app events such as IMPRESSION, INTERACT, etc. may happen
        | --> INTERACT --> one of the content is clicked
            | --> START(type: "player") --> events generated by specific content
                ...|
                ...| --> in-content events such as ASSESS, INTERACT, IMPRESSION, LEVEL_SET etc.
                ...|
            | --> END(type: "player")
        | --> IMPRESSION - Returned back to mobile app for content player
        ...| --> app events such as IMPRESSION, INTERACT, etc. may happen
    END(type: "session")
    ...| --> app events such as APP_UPDATE, FEEDBACK, etc may happen
END(type: "app")
</pre>

### Web Portal

<pre>
AUDIT (object: user) --> (Optional if a user is created for the first time)
START(type: "session") --> User session starts
    ...
    | --> IMPRESSION - For the pages that the user visits
    | --> INTERACT - For the interactions on the page
    // there is no explicit logout/timeout
</pre>

### Content editor

<pre>
START (type: "session") - User logs in
    ...
    | --> IMPRESSION (Portal) - User visits content creation page (cdata session)
    | --> INTERACT (Portal) - User initiates content creation
    | --> AUDIT (Platform) - User creates a new content
    | --> START (type: "editor", mode: "content")
        ...
        | --> INTERACT (Editor) - In editor, user is loading the asset browser
        | --> SEARCH (Platform) - In AT, user is searching for assets (cdata session, search result id)
        | --> PLUGIN_LIFECYCLE (Editor) - User selects the asset for content (which search result was used)
        | --> PLUGIN_LIFECYCLE (Editor) - User removes the asset from content
        | --> INTERACT (Editor) - User clicks save in AT
        | --> ACCESS (Platform) - Platform save API is called
        | --> INTERACT (Editor) - User clicks on submit to review
        | --> AUDIT (object: Content, state: "Review", prevstate: "Draft") - Platform sends the content to review state
    | --> END (type: "editor", mode: "content") - User closes the editor and goes back to portal
    </pre>
    
### Backend-services

<pre> 
    AUDIT (object: Service, state: "Ready") --> State transition to READY
    ...| --> ACCESS events for API requests
    ACCESS
        LOG --> Log events in the context of the incoming request (by request correlation ID)
        ...
    METRICS --> Health/business metrics (e.g. number of jobs executed)
    AUDIT (object: Service, state: "Stopped") --> State transition to STOPPED
    </pre>