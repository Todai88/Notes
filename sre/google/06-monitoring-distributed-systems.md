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
- [Worrying About Your Tail (or, Instrumentation and Performance)](#worrying-about-your-tail-or-instrumentation-and-performance)
- [As Simple as Possible, No Simpler](#as-simple-as-possible-no-simpler)
- [Tying These Principles Together](#tying-these-principles-together)
- [Monitoring for the Long Term](#monitoring-for-the-long-term)
    - [Bigtable SRE: A Tale of Over-Alerting](#bigtable-sre-a-tale-of-over-alerting)
    - [Gmail: Predictable, Scriptable Responses from Humans](#gmail-predictable-scriptable-responses-from-humans)
    - [The Long Run](#the-long-run)
- [Conclusion](#conclusion)

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
 
## Worrying About Your Tail (or, Instrumentation and Performance)

As mentioned in chapter 4, it's tempting to monitor the mean of a metric.
The chapter refers to how measuring the average request latency may hide details
of a system. For instance, it doesn't tell you anything about the distribution of
the system's latencies. The average latency may be 50ms, but 1% of
users may be experience latencies far higher than that. 

To mitigate this, consider collecting request counts categorised in levels by latencies.
The levels depends on the system's SLIs and SLOs.
But an example could be requests that take:

- 0-10ms
- 11-30ms
- 31-100ms
- 100-300ms

Distributing the boundaries by a set factor (in this case 3)
makes it easier to visualise the distribution.  

## Choosing an Appropriate Resolution for Measurements

Take care when you structure the granularity of your measurements. 
For example:

- Observing CPU load over a minute won't reveal long-lived spikes that may drive
high tail latencies.

- But for a web service with a target of ≤ 9 hours of downtime / year (99.9% annual
downtime) to collect HTTP requests more than twice / minute is too frequent. 

- Similarly, it's unnecessary to check hard drive saturation for a service
targeting 99.9% availability more than once every 1-2 minutes.

Some system's goals call for high resolution and low latency.
The problem is that frequent measurements is expensive to collect,
store and analyse. To help with this, a system may perform internal sampling
on a server, then configure an external system to collect that data.
Then aggregate the data and distribute it across servers. For instance:

- Record the current CPU utilisation each second.
- Increment the appropriate bucket (of 5% granularity,
like the request latency buckets above) each second.
- Aggregate those values every minute.

## As Simple as Possible, No Simpler

Like all software systems, monitoring can become so complex that it
becomes a maintenance burden. Common complexities are:

- Alerts on different latency thresholds, at different percentiles,
on all kinds of different metrics.

- Extra code to detect and expose possible causes.

- Dashboards for these possible causes.

To help with these complexities, Google has outlined some guidelines to help
design and steer a system towards simplicity:

- The alert rules that catch real incidents should be as simple,
predictable and reliable as possible.

- Rarely used data collection, aggregation and alerting configuration
(less than once / quarter) should be up for removal. 

- Signals that are collected, but not exposed to a dashboard or
used by an alert, are candidates for removal.

## Tying These Principles Together

Asking the following questions when creating rules for alerting can help you avoid
false positives and pager burnout:

- Does this rule detect **an otherwise undetectable** condition that is urgent,
actionable and user-visible?

- Will I ever be able to ignore this alert? When and why can I, and how
do I avoid this?

- Does this alert definitely indicate that users are being negatively affected?
If not, are there cases that should be filtered out?

- Can I take action in response to this alert? Is the action urgent,
or can it wait until morning? Can the action be *safely* automated to reduce toil?
Will the action be a long-term fix, or a short-term workaround?

- How many people should be paged for this issue?

These questions reflect a fundamental philosophy on pages and pagers:

- I should be able to react with a sense of urgency if a pager goes off.
However, I can only react with a sense of urgency a few times per day before
I become fatigued.

- Every page should be actionable.

- Every page response should require intelligence.
If a page merely merits a robotic response, it shouldn't be a page.

- Pages should be about a novel problem, or an even that hasn't been seen before.

If a page satisfies the four bullets above, it doesn't matter if the page is
triggered by white-box or black-box monitoring. As such, it's better to spend
(much) *more effort on catching symptoms than causes*. When it comes to causes,
only worry about very definite, very imminent causes.

## Monitoring for the Long Term

Due to how fast software development moves, tech debt is fairly common.
As such, an alert that was previously rare, may become frequent.
The cause may even merit executing a hacked-together script to resolve it.
At this point, the root cause should be eliminated, and if a resolution isn't
possible, the alert response should be automated. 

Making decisions on monitoring should have the long-term in mind.
Every page will distract a human from improving the system for tomorrow.
So there is often a case for taking a short-term hit to availability or performance
to improve the long-term outlook of the system. 

Following are two case studies at Google.

### Bigtable SRE: A Tale of Over-Alerting

TBA

### Gmail: Predictable, Scriptable Responses from Humans

TBA 

### The Long Run

TBA

## Conclusion

A monitoring and alerting pipeline is considered healthy when it's simple and easy to reason about.
It should primarily focus on symptoms for paging, whilst reserving cause-oriented
metrics to aid with debugging causes. 

Symptoms or easier to monitor the further "up" the stack you are. However, monitoring
saturation and performance of a system, such as a database, might have to be performed
on the subsystem itself. 

Email alerts hold limited value, and increases noise. Instead, favour a dashboard 
that monitors all ongoing, sub-critical problems. A dashboard can also be paired
with a log, to analyse historical correlations.

Achieving a successful on-call rotation and product includes
choosing to alert on symptoms or imminent problems. Adapt your targets
to goals that are achievable, and make sure that your monitoring supports 
rapid diagnosis.

