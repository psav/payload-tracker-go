# Payload Tracker

## Overview
The Payload Tracker is a centralized location for tracking payloads through the Platform. Finding the status (current, or past) of a payload is difficult as logs are spread amongst various services and locations. Furthermore, Prometheus is meant purely for an aggregation of metrics and not for individualized transactions or tracking.

The Payload Tracker aims to provide a mechanism to query for a `request_id,` `inventory_id,` or `system_uuid` (physical machine-id) and see the current, last or previous X statuses of this upload through the platform. In the future, hopefully it will allow for more robust filtering based off of `service,` `account,` and `status.`

The ultimate goal of this service is to say that the upload made it through X services and was successful, or that the upload made it through X services was a failure and why it was a failure.

## Architecture
Payload Tracker is a service that lives in `platform-<env>`. This service has its own database representative of the current payload status in the platform. There are REST API endpoints that give access to the payload status. This service listens to messages on the Kafka MQ topic `platform.payload-status.` There is now a front-end UI for this service located in the same `platform-<env>`. It is respectively titled "payload-tracker-frontend."

## REST API Endpoints
Please see the Swagger Spec for API Endpoints. The API Swagger Spec is located in `api/api.spec.yaml`.


## Message Formats
Simply send a message on the ‘platform.payload-status’ for your given Kafka MQ Broker in the appropriate environment. Currently, the only required fields are ‘service,’ ‘request_id,‘ ‘status,’ and ‘date‘ however this may change.The format is as follows:

```
{ 	
    'service': 'The services name processing the payload',
    'source': 'This is indicative of a third party rule hit analysis. (not Insights Client)',
    'account': 'The RH associated account',
    'org_ig': 'The RH associated org id',
    'request_id': 'The ID of the payload',
    'inventory_id': 'The ID of the entity in terms of the inventory',
    'system_id': 'The ID of the entity in terms of the actual system',
    'status': 'received|processing|success|error|etc',
    'status_msg': 'Information relating to the above status, should more verbiage be needed (in the event of an error)',
    'date': 'Timestamp for the message relating to the status above'
}
```
The following statuses are required:
```
‘received‘ 
‘success/error‘ # success OR error
```

## Development
#### Prerequisites
```
docker
docker-compose
Golang >= 1.15
```

#### Launching the Service
Launch DB, Zookeeper and Kafka
```
$> docker compose up payload-tracker-db
$> docker compose up zookeeper
$> docker compose up kafka
```
Migrate and seed the DB
```
$> make run-migration
$> make run-seed
```
Compile the source code for API and Consumer into a go binary:
```
$> make build-all
```
Launch the application
```
$> ./pt-api
$> ./pt-consumer
```
The API should now be available on TCP port 8080
```
$> curl http://localhost:8080/api/v1/
$> lubdub
```

## Running Tests
Use `go tests` to test the application
```
$> go test ./...
```
