# AppD &amp; Prometheus Integration

## Introduction

This extension connects to a Prometheus endpoint and runs the specified queries.
Responses are then parsed and then passed to AppDynamics as analytics events.




## Pre-requisites

1. (Optional) Homebrew - for easier installation and management on MacOS
2. Node.JS - currently targeting latest LTS version (10.16.3)

```
$ brew install node
```

3. (Optional) AWS Account with access to IAM and Lambda - only required if deploying to Lambda

4. (Optional) Claudia.js - only required if deploying to Lambda

```
$ npm install claudia -g
```

5. AppDynamics controller with appropriate Analytics licence.



## Installation

### Clone package

```
$ git clone git@github.com:lspacagna/appd-prometheus.git
$ cd appd-prometheus
```
### Choose to run extension locally or in Lambda

This extension default configuration is to run locally. If you would like to run the
extension inside a Lambda function. You need to edit src/index.js and comment
out the last line in the file. It should look like this:

```
// runLocal()
```

### Rebuild project (only if deploying to Lambda)

If you are deploying to Lambda and have commented out 'runLocal()' you will need to rebuild the project. Rebuilding will parse the source code in /src and store the built version in /dist.

```
npm run build
```

## Configuration

### Configure extension controller connection

Open the the conf/config.json file for editing. The default configuration is below

```
{
  "read_local": false ,
  "prometheus_url": "http://localhost:9090",
  "appd_analytics_url": "https://analytics.api.appdynamics.com",
  "appd_global_account_name": "",
  "appd_events_api_key": "",
  "schema_name": "prometheus_events",
  "local_file": "data/sample.json"
}
```


Parameter | Function | Default Value
--------- | -------- | -------------
read_local | Choose to read from local data file instead of pulling data from Prometheus API. Useful during debugging. | `false`
prometheus_url | The URL of your Prometheus deployment | `http://localhost:9090`
appd_analytics_url | URL to connect to the AppD controller events service. See [our documentation](https://docs.appdynamics.com/display/PRO45/Analytics+Events+API#AnalyticsEventsAPI-AbouttheAnalyticsEventsAPI) for the URL for your controller. | (blank)
appd_global_account_name | Account name to connect to the AppD controller See Settings > License > Account for the value for your controller | (blank)
appd_events_api_key | API Key to connect to AppD controller events service See [our documentation](https://docs.appdynamics.com/display/PRO45/Managing+API+Keys) | (blank)
schema_name | Reporting data to analytics requires a schema to be created. Change this value if you are connecting more than one of these extensions to more than one Prometheus deployment | `prometheus_events`
local_file | The location of the local file used for data when `read_local` is set to `true` | `data/sample.json`

### Configure Schema

To be able to publish Prometheus data to AppD a custom schema needs to be created in your controller. This schema must match the data types of your Prometheus data. The default schema configuration matches the schema required for the default queries in conf/queries.txt.

Open conf/schema.json for editing.

```
{
  "name": "string",
  "instance": "string",
  "job": "string",
  "quantile": "string",
  "code": "string",
  "handler": "string",
  "value": "float"
}

```

Ensure the following:

* There is a paramter for each value returned from every Prometheus query.
* `name` is required and should not be changed.
* `value` is required and shold not be changed.

The extension cannot modify or delete existing schemas. If you have an existing schema which needs editing follow instructions [in our documentation](https://docs.appdynamics.com/display/PRO45/Analytics+Events+API#AnalyticsEventsAPI-update_schemaUpdateEventSchema)

## Run Extension

### Run extension - locally
If running locally the extension is ready to run. Run the extension with the
following command.

```
$ yarn run run
```

### Run extension - Lambda

Create AWS profile with IAM full access, Lambda full access, and API Gateway
Admin privileges.

Add the keys to your .aws/credentials file

```
[claudia]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_ACCESS_SECRET
```

#### Send function to AWS via Claudia

```
$ claudia create --region us-east-1 --handler index.handler
```

When the deployment completes, Claudia will save a new file claudia.json in
your project directory, with the function details, so you can invoke and
update it easily.

For more detailed instructions see: https://claudiajs.com/tutorials/hello-world-lambda.html

#### Running in AWS

You can either use the AWS UI to trigger the function. Or you can setup a trigger.
A common trigger would be to run this extension once per minute.



## Define Prometheus Queries

The extension has been designed to run Prometheus queries in series. By default
the extension will run two sample queries and send the data to AppD.

Currently, the queries are configured within the extension code. Future versions
of this extension will configure in an external configuration file.

To change the default queries find the getDataFromPrometheus() method. For each
query you would like to run add a new 'prometheusRequest' line. The query should
be passed as the variable to the prometheusRequest call.

Default config:
```
data.push(await prometheusRequest('prometheus_target_interval_length_seconds'))
data.push(await prometheusRequest('prometheus_http_requests_total'))
```

Default config with custom query:
```
data.push(await prometheusRequest('prometheus_target_interval_length_seconds'))
data.push(await prometheusRequest('prometheus_http_requests_total'))
data.push(await prometheusRequest('CUSTOM QUERY HERE'))
```

Remember to re-compile the project after making changes.
