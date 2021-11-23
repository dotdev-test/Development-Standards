# Development Approach

## Connector Database

* Build tables to store all mandatory objects in Shopify with their native structure and relation
* Build tables to store all mandatory objects in tender platform with their native structure and relation
* Build mapping tables store identifiers of object in two platforms
* Enable logs tables, should be automatically created by Monolog system upon every tasks

## Field Mapping

* Mapping fields in payload to database fields based on [Field Mapping](technical-specification.md) document
* Extract payload and save a copy in connector database before sending ahead to the other platform

## Postman Setup

* Setup Shopify requests in production/dev environments
* Setup requests in production/stage/dev environments of tender platform (client side needs to support multi-environments)
* Share to team directory and ensure access permission is open to whole team

## **Webhook Endpoints**

* If a task in [Sequence Diagram](technical-specification.md) is dispatched from Shopify, then it needed to be implement as a Webhook
* Setup Webhooks as respond endpoints of Shopify events
* Allows public access to your webhooks (example: monmon.dotdev.build:8080)

## Cron Job / Scheduling

* If a task in [Sequence Diagram](technical-specification.md) is dispatched from connector, then it needed to be implement as a unit command or service
* Schedule tasks upon frequency requirement, can leverage framework scheduling function or using native linux cron system
* Be aware of timezone and dependency between each job

## Logging

Currently using [Monolog](https://github.com/Seldaek/monolog) as our log system. The following listing events should always be logged. Developers can insert additional logs upon project requirement and development convenience.

### API

* When triggered
* Original incoming payload
* Decoded incoming payload (readable)
* Original outgoing payload
* Decoded outgoing payload (readable)
* Each single process result in loop
* All Exception/Error with error code, error message
* When task finished

### Backend Cron Job

* When triggered
* Each single process result in loop
* All Exception/Error with error code, error message
* When task finished

### Admin Monitor Action

* When and what action triggered and by whom

## Admin Monitor

* Functions to force object syncing
* Functions to monitor processing logs
* Functions to lookup identifier and details of objects in two platform
* Provide different permission of functions for development team and client team
