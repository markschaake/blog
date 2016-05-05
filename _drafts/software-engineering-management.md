---
layout: post
title:  "Thoughts on Software Engineering Management"
date:   2016-05-03 12:20:00 -0800
categories: software engineering
---

# The Importance of a Work Log

Sowftware engineering teams typically iterate through cycles of planning followed by implementation. These cycles are commonly called "sprints". A sprint begins with a spring planning, and ends with some sort of deliverable software. Finally, before starting the next sprint a common practice is to have a "retrospective" meeting in which the team reflects on what went well, what didn't go so well, and opportunities for team process improvements. Several sprints may be rolled into something bigger - say a set of major milestones that have been committed to for delivery by a certain date. You can imagine that these might be quarterly commitments, with their own planning at the beginning of the quarter and a major retrospective at the end. The methodology I've sketched out here is designed to facilitate continuous team process improvement. The team should be able to identify productivity bottlenecks and it should be able to propose solutions to try out in the next sprint. Sounds great, right? So how can a team identify bottlenecks? Are retrospectives supposed to be meetings where individuals exchange subjective opinions? Or can retrospectives include some objective data analysis?

At a minimum, if a team is serious about continuous improvement, they must maintain an accurate __work log__. A work log is a record of every work item that changed the software. You can think of this as code changes plus quality-impacting changes (such as change in deployment if it improves performance). Ideally, your work log will contain __pre-planned__ work items as well as __gametime__ work items. Pre-planned work items are those that were scheduled during sprint or major milestone planning. Gametime work items are unplanned changes made in the middle of a sprint. Why is this distinction important? Because if you really want to get better (continuously improve), then you need to understand what actually happened in the past. Did you do everything you planned to do? If not, why not? What was the cost of the gametime changes? Could we have done better if some gametime changes were not made? What types of errors did we make in planning? Analyzing gametime work items can provide valuable insight into the types of work items that you failed to account for during planning. It can also reveal that the team may occasionally work on things that are not necessary to deliver the planned work items. This is a [wasteful][muda] practice and should be identified and eliminated.

# The Importance of Traceability

__Traceability__ in software engineering is the ability to relate engineering events to each other. When your project has a high level of traceability, then it is easy to discover the events that led to changes in software. For example, you may have a PivotalTracker feature that was implemented in a Github repository, and via webhooks and special commit message syntax you estabilish bidrectional linkage between the source code and the PivotalTracker feature request. If you are browsing the PivotalTracker feature, you can click a link to go to the code changeset that delivered it. Likewise, if you are browsing the pull request that delivered the feature, you can click a link to take you to the PivotalTracker feature.

- Work item requests (features, bugs, chores)
- Work item deliveries (feature implementation, bug fix, chore completion)
- Regressions

At the code level, your SCM system may provide some basic traceability features. For example, Git provides the [`git-blame`][git-blame] command which can tell you the last commit that changed every line of code in a file. This is useful if you discover a bug and want to find out when and by whom it was introduced...

# Examples of Traceability

- Hyperlinks in commit messages
- Hyperlinks to commits / pull requests in issue tracking system
- Links in source code comments to issues

# Traceability Leads to Professionalism

The "easy" way to develop software is to just write code and not worry about team process. This is __not__ professional software engineering...

# The Importance of Leadership

- Maintaining process: someone needs to shephared in new process. Someone needs to be a watchdog every now and then.

[muda]: https://en.wikipedia.org/wiki/Lean_software_development#Eliminate_waste
[git-blame]: https://git-scm.com/docs/git-blame
