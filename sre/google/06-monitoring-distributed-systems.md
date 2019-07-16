# Monitoring Distributed Systems

This chapter offers guidelines for which issues should interrupt a human via a page,
and how to deal with issues that aren't serious enough to trigger a page.

## Table of contents

- [Definitions](#definitions)
- [Why Monitor?](#why-monitor?)
- [Setting Reasonable Expectations for Monitoring](#setting-reasonable-expectations-for-monitoring)
- [System Versus Causes](#system-versus-causes)
- [Black-Box Versus White-Box Monitoring](#black-box-versus-white-box-monitoring)
- [The Four Golden Signals](#the-four-golden-signals)

## Definitions

- *Monitoring*

  Collecting, processing, aggregating and displaying quantitative data about a system.
- *White-box monitoring*
   Monitoring based on metrics exposed by the internals of the system (logs, interfaces, HTTP handlers, etc).
- *Black-box monitoring*

  Testing behavior as a user would experience it.
- *Dashboard*
   An application that provides a summary of a system's metrics. It may also display metrics for the team responsible for maintaining the system, such as tickets.
- *Alert*

  A (push) notification, sent to a ticket queue, as an email or as a page.
- *Root cause*

  A defect in in a software that, if repaired, raises confidence its effects won't happen again in the same way.
- *Node and machine*

Used interchangeably to state a single instance of a running kernel. There may be many *services* worth monitoring on a single machine. These can either be:

    - Related to each other (eg. caching server and web server)
    - Unrelated services (eg. a code repository and a master for a configuration system)
  
- *Push*

  Any change to a service's running software or its configuration.

## Why Monitor?

- *Analysing long-term trends*

  How big is the system's database and how fast is it growing?
- *Comparing over time or experiment groups*

  Is my site slower than it was last week?
- *Alerting*

  Something is broken and needs to be fixed (right now!)
- *Building dashboards*

  Should answer basic questions about the service and (normally) include a form of [the Four Golden Signals](#the-four-golden-signals).
- *Conducting ad hoc retrospective analysis*

  If the system's latency suddenly spikes; what else happened around the same time?

In conclusion, monitoring and alerting enables a system to tell us when it's broken, or when it's about to break. When a system can't recover from this, we want a human to investigate the alert. This includes mitigating the error and determining the root cause(s).

Paging (an automated call to an employee due to an alert), is quite an expensive operation, both for the employee and employer. If the employee is working, it will interrupt their workflow. If the employe is at home, it will interrupt their personal time, possibly even their sleep. Furthermore, when pages occur too frequently, there is an increased risk that they carelessly are skimmed over. There is also an increased risk that a *real* page is ignored if pages are too noisy.

In comparison an effective system should have good signal and very low noise.

## Setting Reasonable Expectations for Monitoring

Google suggests to have clear and simple alert rules.
Further, SRE teams should skip complex dependency hierarchies,
where possible. An example of a complex dependency hierarchy for an
alert rule could be "If I know the database is slow,
alert for a slow database; otherwise, alert for the website being generally slow".

As such, it's important that monitoring systems and their alert rules
are simple and comprehensible for everyone on the team.
Especially the critical path and its related pages:

1. Alert raised from production
2. Page to human
3. Basic triage (fix shallow issues)
4. Deep debugging

In conclusion, to keep signal high and noise low,
the elements of the system that direct to a pager need to be simple and robust.
Additionally, rules that generate alerts for humans should be kept simple
and represent *clear* failure.

## System Versus Causes

Monitoring a system should address two questions: "what's broken?", and "why?".

"What's broken?" indicates the symptom; "why" indicates the cause. 
[Table 6-1](#table_6-1) lists some (hypothetical) symptoms and causes.
    
*Table 6-1: Example symptoms and causes*
  
| Symptoms (What's broken?)                                      | Cause  (Why?)                                                        |
|-----------------------------------------------|----------------------------------------------------------------|
| I'm serving HTTP 500s or 404s                 | Database servers are refusing connections                      |
| My responses are slow                         | CPUs are overloaded                                            |
| Users in Antarctica aren't receiving cat GIFs | The Content Distribution Network hates scientists and felines. |

<a name="table_6-1"/>

The distinction between "what" and "why" is important for writing monitoring with maximum signal and minimum noise.

## Black-Box Versus White-Box Monitoring

Black-box monitoring is symptom-oriented and represents *active* problems. 
For instance, it may detect that "The system isn't working, *right now*".

White-box monitoring, on the other hand, count on the ability to inspect the 
system's internals.
Examples of such internals are HTTP endpoints or logs.
So, one of white-box monitoring's purposes is to detect imminent problems,
such as failures masked by retries. 

Note that in a multi-layered system, one teams's symptom can be another team's cause.
For instance, if database reads are slow, it would be a symptom for the database SRE
who detects it.
But,  for a frontend SRE observing a slow website, that symptom would be a cause.
As such, white-box monitoring can sometimes be symptom-oriented,
and sometimes cause-oriented. Depending on how informative your white-box is. 

In conclusion, black-box and white-box monitoring each serve their own purposes.
White-box monitoring is essential for debugging a system.
For instance, if servers are slow on database-heavy requests,
you need to know both *how fast servers perceive the database to be*, and
*how fast the database believes itself to be*.
Otherwise, it's difficult to distinguish if the cause is a slow database,
or a network problem between your components.
The benefit of black-box monitoring is that it only is aware of if ongoing failures,
rather than imminent ones. So it's a great candidate for paging (interrupting) a human. 
Keeping these benefits in mind may decrease the sound that your alerting produces.

## The Four Golden Signals

- **Latency**
  
  The time it takes to service a request. 
  
  Note that monitoring should distinguish between latency of *successful* and *failed*
requests. The collected metrics may be misleading if that distinction is not made. 
For instance, a failed request (500) due to a lost database connection may be served 
very quickly, which would skew the metrics.  
  
- **Traffic**

  This metric may be different depending on the type of system that is being monitored.
For instance, if it's a web service, the SRE team would mainly be interested in HTTP 
requests per second. The request may be split into categories depending on the nature of 
the request (e.g., static versus dynamic content). 

- **Errors**

  The rate of requests that fail. 
  
  This may either be *explicitly* due to an
internal server error, or *implicitly* due to serving a request to the user,
but with incorrect content.
Further, it is possible to consider requests as failures if they break policy
(e.g., "If you committed to one-second response times, any request over
one second is an error").
  
  Depending on the type of failure, monitoring different layers may be necessary.
For instance, monitoring a load-balancer may catch all HTTP 500s.
But end-to-end monitoring tests may be the only way to detect that
wrong content is being served.

- **Saturation**

  How "full" your service is.
  
  A measure of your *system fraction*, emphasising the resources that are most
constrained by your system. For instance, memory is the system's main concern,
the SRE team should focus on measuring the memory.
Consider having a utilisation target, as most systems degrade
their performance before reaching 100% *saturation*.
  
  In complex systems, *saturation* can be supplemented with *higher-level
load measurements*. Meaning, can the service handle double the traffic,
handle only 10% more or can it only *reasonably* handle less than it
currently receives?
  
Finally, a service can be considered *decently* covered by monitoring
if you are measuring the four golden signals. Further, as a rule of thumb,
if a signal fails (or in the case of saturation, *nearly* is failing)
the system should page a human. This may help with reducing a system's overall noise.
 