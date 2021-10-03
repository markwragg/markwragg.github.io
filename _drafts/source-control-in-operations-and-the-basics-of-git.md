---
layout: post
title: Source control in operations and the basics of Git
---

It is quickly becoming apparent that implementing and using a source control system is an important skill to master. This has long been true in development, but within (Windows) Operations it is (seemingly for many) a relatively new or uknown concept. Many organisations have a change control policy and process which demands that changes be recorded and approved before implemented. However its still generally possible for this policy to be bypassed (deliberately or accidentally). In times of crisis it's often the first thing to go out the window.

For source control in Operations to be relevant, we need to ensure that changes in the environment are only possible via the source control system. It must not be possible to make changes to the servers directly (or at the very least, it must not be possible to make the changes we've source controlled directly).

## Benefits of source control to Ops
- Centralised configuration, a precursor to enabling configuration management/auto scaling technologies
- The ability to see who committed changes, exactly what those changes were and to easily revert changes to the previous revision of the configuration if needed.

Git - "a version control system that is widely used for software development and other version control tasks"

The most widely used system for managing and collaborating on source code is Git. This blog post describes my understanding of some of the basic concepts.

If you are new to Git yourself, I strongly recommend you follow this guide which takes you through the basic concepts: https://guides.github.com/activities/hello-world/