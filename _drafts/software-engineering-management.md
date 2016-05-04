---
layout: post
title:  "Thoughts on Software Engineering Management"
date:   2016-05-03 12:20:00 -0800
categories: software engineering
---

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

[git-blame]: https://git-scm.com/docs/git-blame
