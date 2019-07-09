# Service Layer Objectives

Uses the [Shakespeare service example](00-shakespeare-service.md) as an example service.

## Table of contents:
* [Service Level Terminology](#service-level-terminology)
    * [Service Level Indicators (SLI)]
    * [Service Level Objectives (SLO)]
    * [Service Level Agreements (SLA)]

----

### Service Level Terminology
####  Service Level Indicators (SLI)
A defined quantitative measure of the level of a provided service.

Examples: 
* request latency; how long time it takes to return a response to a request.
* error rate; a percentile or fraction of received requests.
* system throughput; measured as request per second.
* availability; the fraction of the time that a service is usable.
* yield; the fraction of well-formed and successful requests. 
* durability; the likelihood of data-retention over a long period of time. Particularly important for storage systems.

The measurements are often aggregated. Meaning it's collected over a period of time. Depending on the type of the measurement, it is then turned into rates, averages or percentiles. 

#### Service Level Objectives (SLO)
A target value or range of values for a service level, measured by an SLI. e.g: *SLI*  ≤ *target* or *lower bound* ≤ *SLI* ≤ *upper bound*. 

In some cases it is difficult to define a SLO. In particular you might not always be able to define its value. For instance, to measure incoming HTTP requests from the outside world to your service, you may use the queries per seconds (QPS) metrics. 
This might seem like a good idea. But you may realise that it can't reliably be measured due to the QPS being dependant on your users' desire to use your service.
But, there are cases where you can add a value to a SLO. For instance and SLO with a value could be "the Shakespeare service should have an average *request latency* lower than 100 ms".  
This SLO may motivate your team to write the front-end with low-latency behaviour. Or it may motivate you to purchase low-latency equipment. 

Yet, it may be more subtle than only having the latter SLI (request latency) in the SLO. The QPS and request latency may be connected behind the scenes. A higher QPS or throughput would likely increase the request latency. Besides, it's common for services to have a performance cliff beyond some load threshold.

If possible, publishing the SLOs to end-users may be useful. This may aid with managing users' expectations on the service, such as availability. Users are likely to develop their own expectations if an explicit SLO isn't available. 
This dynamic can lead to both over-reliance on the service and under-reliance. 

An example is the Global Chubby (Google's lock service) Planned Outage: 

The lock service was too reliable, to the point where users would rely on it to never be down. This lead to external services being dependant on it being available (tight coupling). 
The solution was to have planned outages. In that way, Google was able to flush out any *unreasonable dependencies*. Doing so ensured that any external services weren't too reliant on Chubby. 

#### Service Level Agreements (SLA) 
An explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLO(s) they contain.
Most commonly these consequences are financial and may incur a penalty. As such it's easy to distinguish between an SLA and SLO. 
If there is no explicit consequence if the SLO isn't met, it's not an SLA. 

SRE typically don't get involved in constructing SLAs, because they are too closely tied to business and product decisions. But, SRE gets involved in helping not triggering the consequences of missed SLOs. Additionally, SRE identifies the SLIs that are the foundation for the SLOs. 
