# Service Layer Objectives

Uses the [Shakespeare service example](00-shakespeare-service.md) as an example service.

----

### Service Level Terminology:
#####  Service Level Indicators (SLI)
A defined quantitative measure of the level of a provided service.

Examples: 
* request latency; how long time it takes to return a response to a request.
* error rate; a percentile or fraction of received requests.
* system throughput; measured as request per second.
* availability; the fraction of the time that a service is usable.
* yield; the fraction of well-formed and successful requests. 
* durability; the likelihood of data-retention over a long period of time. Particularly important for storage systems.

The measurements are often aggregated. Meaning it's collected over a period of time. Depending on the type of the measurement, it is then turned into rates, averages or percentiles. 

##### Service Level Objectives (SLO)
* Service Level Agreements (SLA). 

