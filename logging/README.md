# Logging, Logging, Logging
## What do you log?
What are your inputs and outputs? What are the different ways these inputs and outputs can go? Are
you suppressing an error? Then count the error.
## Errors
Errors should go through some sort of service. Sentry is amazing changed my life.
## Metrics
Metrics are great. Measure effectiveness of a change or measure how a service is performing. A
logging service may track the number of events being inserted into the pipeline and the number of
events being read to track the processing lag. A change to the icon used for favorites on an
application may track # of clicks on the icon between A/B testing groups. A network glue application
may track # of network errors
## Logging service (time based log service)
Remember to also log the information required to debug an issue. Counting number of errors for the
network glue application, also log the endpoint and the error code returned.
## Tracing
### Request Tracing
UUIDs to track a request through multiple services
### Execution Tracing
Tracking execution times of different function calls and where a request stalls in a service