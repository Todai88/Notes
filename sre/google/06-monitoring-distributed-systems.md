# Monitoring Distributed Systems

This chapter offers guidelines for which issues should interrupt a human via a page,
and how to deal with issues that aren't serious enough to trigger a page.

## Table of contents
- [Definitions](#definitions)

## Definitions

- *Monitoring*
> Collecting, processing, aggregating and displaying quantitative data about a system.
- *White-box monitoring*
> Monitoring based on metrics exposed by the internals of the system (logs, interfaces, HTTP handlers, etc).
- *Black-box monitoring*
> Testing behavior as a user would experience it.
- *Dashboard*
> An application that provides a summary of a system's metrics. It may also display metrics for the team responsible for maintaining the system, such as tickets.
- *Alert*
> A (push) notification, sent to a ticket queue, as an email or as a page. 
- *Root cause*
> A defect in in a software that, if repaired, raises confidence its effects won't happen again in the same way.
- *Node and machine*

> Used interchangeably to state a single instance of a running kernel. There may be many *services* worth monitoring on a single machine. These can either be:
    
    - Related to each other (eg. caching server and web server)
    - Unrelated services (eg. a code repository and a master for a configuration system)
- *Push*

> Any change to a service's running software or its configuration.

