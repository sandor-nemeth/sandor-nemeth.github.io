---
published: false
title: Monitoring dashboards
---
On Monday my boss come up with some very pointed questions about our monitoring wallboards. We talked about it for some time and then I had thought about them some more, and also about how we can organize them. 

But before all I wanted to categorize them, because that would give me a tangible idea when I am building the dashboards on what exactly do I want to visualize. 

In my view there are 3 different kind of dashboards depending on the target of the dashboard - that we could/should/would utilize:

1. Wall dashboards
2. Overview dashboards
3. Telemetry dashboards

Wall dashboards are dashboards which go up to our monitors. IMO they should communicate what we want to communicate with them in a single blick. I think the main point for now would be to get an overview of the state of the system and see if there are any problems. I do not think we should have documentation for this kind of dashboard, because it should be obvious even when you haven't read the docs in 6 months, and we should aim for that. Of course there are some prerequisites to know our system and what components are there, but that should be the only requirement, and after that introduction everybody should be able to see with 1 look onto this board the overall system state. So to summarize:

Intended audience: HoT / Devs / everybody
Goal: immediate recognition of issues in our systems
Avg usage duration: 5s
Scope: system-wide
Documentation level: minimal, MUST be understandable with some domain knowledge on a blick

Overview dashboards are dashboards which live in grafana, and provide us more detailed insight into our systems. They are dashboards which provide insight to how our system is behaving, providing also some high-level telemetry data so that when analysing we are able to ascertain the health and performance of our system. The intended consumption method is to look at them when sitting in front of a screen and being able to look at this for longer than 10 seconds. They could be documented on a high level, though IMO still most of the devs working on could understand it just by looking at the dashboard (even if that understanding is not immediate). 

Intended audience: HoT / Devs
Goal: recognition of issues in our systems, ascertain performance of system
Avg usage duration: ~1-10min
Scope: group of applications / single application 
Documentation level: can be documented per metric in detail

Telemetry dashboards are dashboards which communicate the telemetry of an application in detail, and from them we should be able to look at any application in detail, determine their health and also make prognoses based on the telemetry data. This is the most detailed of all the boards we should/could/would make and also provides the most detailed look into a single application. The documentation on this should be detailed because here the devs would spend the most time, and it should also be very detailed. 

Intended audience: Devs
Goal: provide deep insight into the system, debugging
Avg usage duration: >10min
Scope: single application 
Documentation level: should be documented in detail



Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
