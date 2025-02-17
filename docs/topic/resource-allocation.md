# Resource Allocation on Profile Lists

This document lays out general guidelines on how to think about what goes into
the list of choices about resource allocation that are presented to the user
in the profile List, so they can make an informed choice about what they want
without getting overwhelmed.

This primarily applies just to research hubs, not educational hubs.

## Factors to balance

1. **Server startup time**

   If everyone gets an instance just for themselves, servers
   take forever to start. Usually, many users are active at the same time, and we
   can decrease server startup time by putting many users on the same machine in a
   way they don't step on each others' foot.

2. **Cloud cost**

   If we pick really large machines, fewer scale up events need to be
   triggered, so server startup is much faster. However, we pay for instances
   regardless of how 'full' they are, so if we have a 64GB instance that only has
   1GB used, we're paying extra for that. So a trade-off has to be chosen for
   *machine size*. This can be quantified though, and help make the tradeoff.

3. **Resource *limits*, which the end user can consistently observe**.

   Memory limits are easy to explain to end users - if you go over the memory limit, your
   kernel dies. If you go over the CPU limit, well, you can't - you get throttled.
   If we set limits appropriately, they will also helpfully show up in the status
   bar, with
   [jupyter-resource-usage](https://github.com/jupyter-server/jupyter-resource-usage)

4. **Resource *requests* are harder for end users to observe** , as they are primarily
   meant for the *scheduler*, on how to pack user nodes together for higher
   utilization. This has an 'oversubscription' factor, relying on the fact that
   most users don't actually use resources upto their limit. However, this factor
   varies community to community, and must be carefully tuned. Users may use more
   resources than they are guaranteed *sometimes*, but then get their kernels
   killed or CPU throttled at *some other times*, based on what *other* users are
   doing. This inconsistent behavior is confusing to end users, and we should be
   careful to figure this out.

So in summary, there are two kinds of factors:

1. **Noticeable by users**
   1. Server startup time
   2. Memory Limit
   3. CPU Limit

2. **Noticeable by infrastructure & hub admins**:
   1. Cloud cost (proxied via utilization %)

The *variables* available to Infrastructure Engineers and hub admins to tune
are:

1. Size of instances offered

2. "Oversubscription" factor for memory - this is ratio of memory limit to
   memory guarantee. If users are using memory > guarantee but < limit, they *may*
   get their kernels killed. Based on our knowledge of this community, we can tune
   this variable to reduce cloud cost while also reducing disruption in terms of
   kernels being killed

3. "Oversubscription" factor for CPU. This is easier to handle, as CPUs can be
   *throttled* easily. A user may use 4 CPUs for a minute, but then go back to 2
   cpus next minute without anything being "killed". This is unlike memory, where
   memory once given can not be taken back. If a user is over the guarantee and
   another user who is *under* the guarantee needs the memory, the first users's
   kernel *will* be killed. Since this doesn't happen with CPUs, we can be more
   liberal in oversubscribing CPUs.

## UX Goals

The goal when generating list of resource allocation choices is the following:

1. Profile options should be *automatically* generated by a script, with various
   options to be tuned by the whoever is running it. Engineers should have an easy
   time making these choices.

2. The *end user* should be able to easily understand the ramifications of the
    options they choose, and it should be visible to them *after* they start their
    notebook as well.

3. It's alright for users who want *more resources* to have to wait longer for a
   server start than users who want fewer resources. This is incentive to start
   with fewer resources and then size up.