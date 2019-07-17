Based on [SLI workshop](https://gds-way.cloudapps.digital/standards/slis.html).

----

As suggested by [Google's SRE book](https://landing.google.com/sre/sre-book/toc/index.html),
which most people consider industry standards, defining SLOs and SLIs are best done in a
backwards fashion.

This means that a team may have an easier time defining SLOs and SLIs if they first consider
what user journeys are most important for their users. The workflow can be outlined as:

- Outline user journeys (the entire team can / should be doing this).
- Prioritise the user journeys based on importance (everyone should do this, but project / product manager has last say).
- Decide on one user journey (likely the highest priority) to define SLIs and SLOs for.
- On a high-level, define a SLO, its SLI(s) and its target(s). 

  If this is too difficult, use keywords such as:
  - request latency; how long time does it take to return a response to a request.
  - error rate; a percentile or fraction of received requests.
  - system throughput; measured as request per second.
  - availability; the fraction of time in which a service is usable.
  - yield; the fraction of well-formed and successful requests.
  - durability; the likelihood of data-retention over a long period of time. Particularly important for storage systems.

  An example can be "The request latency of the 99.99th percentile should lower than 2 seconds".
  This entire sentence is the SLO, the SLI is "request latency" (for the 99.99th percentile)
  and the SLIs target is "lest than 2 seconds".

- Create tasks and actions for the SLIs to be put into production
- Repeat for the other journeys (likely based on priority).