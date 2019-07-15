# Monitoring Distributed Systems

This chapter offers guidelines for which issues should interrupt a human via a page,
and how to deal with issues that aren't serious enough to trigger a page.

## Table of contents

- [Definitions](#definitions)
- [Why Monitor?](#why-monitor?)

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

  Should answer basic questions about the service and (normally) include a form of the [Four Golden Signals](#four-golden-signals).
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
