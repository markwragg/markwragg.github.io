---
title: Operational metrics
---
How should we measure success in Operations? Why is it important to track metrics? What metrics are informative and what metrics are not? What are traps/pitfalls? We need to be careful not to assume doing more with less is necessarily an indicator of success. How do you measure quality?

Number of 24x7 paging alerts is a good starting point, per recent tweet from John Arundel https://twitter.com/bitfield/status/726344907741401088

(you can display a tweet in line)
https://support.twitter.com/articles/20169559

What metrics are dangerous?
Number of tickets
Time taken to resolve tickets (without thought to type/categorisation)
Number of changes - rate of change is not a measure of success
Change success?? Depends on the cause of failure. What constitutes a failed changed - topic for another post.

Good metrics
Commits to source control? % of configuration in source control? Hard to get a firm number on this tho. Oh % of changes made in source control vs non source control?
%age of tasks automated (full, partial) vs manual
Percentage of production on latest version
Time between builds
Time between release and production 
Turnover
Capability
Response time
Time taken to complete builds
Time taken to complete upgrades
Uptime
Mean time to recovery
Project tasks completed or % of time devoted to projects?
Customer ticket volume -- not total tickets but cust incidents indicates how much pain customers suffer 


https://www.pagerduty.com/blog/operational-metrics/
Raw incident count
mttr
Time to respond (in hrs / out hrs?)
Escalations -- definitely want to start counting these 

Metrics must be
Important to the business
Actionable
Measured frequently
Relevant to the audience 

Dashboards must be
Simple to interpret
Provide context

https://www.klipfolio.com/resources/articles/kpi-dashboard-operational-metrics-top-10-guidelines

ITIL suggests measuring processes in two ways: quantitative and qualitative. Quantity is easy, it's things like how many tickets, how many failed changes. Quality is harder, often requiring you to speak to people on their perception of the service.

Quality measures:

Assess and score the quality of a random sample of changes.
Assess and score the quality of a random sample of incidents.
Assess a build request, whether it was delivered on time and how many updates were provided : could do this one programmatically.

Could script the selection of sample tickets.

