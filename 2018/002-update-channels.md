 Title

| Field  | Value      |
|:-------|:-----------|
| Status | Draft      |
| Date   | 2018-07-31 |

## Introduction

### Problem description

Updates can be disruptive for a customer's application stack (ex. Kubernetes
upgrade from 1.10.x to 1.11.x, default pod security policy introduction).

Right now there is no visibility of what is inside an update and furthermore
there is no possible way around intrusive updates by having the ability to only
apply critical bug fixes and security updates. An admin needs to apply all updates
that come through a single channel.

In order to separate intrusive updates in SUSE CaaS Platform, we need different
update channels. This allows the separation of possibly disruptive updates
into their own channels, and lets the admin continue applying bugfixes and security
updates without the need to apply intrusive, new features. The admin can also see
what is inside an update to be able to decide when to apply it.

### Proposed change

The admin will have the ability of choosing when to switch to the next channel.
A link to the release notes of the update channel is displayed and can be
evaluated by the admin beforehand.

The UI of velum needs to be adapted to enable a switch to a new channel.

## Detailed RFC

We can separate the updates into different channels. There are 2 types of channels:

* update channel: normal maintenance updates, which are safe and non-intrusive
* new version channel: version updates which can contain intrusive updates

This RFC proposes:

* a front-end change in the velum UI to enable the admin to switch to a new version
channel
* a back-end change in salt to perform the channel switch on all cluster nodes

### Proposed change (Detailed)

#### Tools/Services

We need a set of tools/services to indicate, whenever a maintenance update or a new version
update is ready to be installed. They should run on each worker node to be more independent
and decoupled from velum. If velum would be the initiator of the update check, it might lead
to inconsistent and long execution runs in larger clusters.

* "check for updates" service on each worker
  - reports availability of maintenance updates to velum (via grains?)
  - once per day e.g.
  - decoupled from velum, runs independently, just reports
* "new version" checker service on each worker
  - check for new versions (migration targets)
    - query RMT/SMT/SCC API if there is a new compatible version available for this CaaSP version (SUSEConnect library)
    - report back to velum (via grains?)
  - get the link to the release notes
  - once per week or longer timeframe
  - decoupled from velum, runs independently, just reports

#### User Interface

The user interface of velum needs to be adapted minimally.

* [indirect] display of update content through [a link to the] release notes
* display of update history via velum
* button to switch to the next channel in velum
* separate UI for updates and upgrades (new migration target)

#### API

Furthermore there needs to be an API available through velum to enable easy scripting

* endpoints:
  - GET:  list current channel maintenance updates (non disruptive)
  - POST: trigger current channel maintenance updates
  - GET:  list next channel (new version) updates
      - A: disruptive update in the same caasp product/version (e.g. k8s minor version bump, container runtime)
      - B: migration target once overlap support is EOL (next caasp product/version)
  - POST: switch to the next channel
  - GET:  list update history
  - POST: trigger reboots

#### General considerations

##### Being outdated before a migration

We need to check if the installed packages are good enough to do the migration or not.
If it's good enough, we can either continue with the migration directly or apply normal
updates and prolong the migration.
The user should not be enforced to run all maintenance updates first, because the migration
would contain/overwrite the patches anyways.

### Dependencies

* new channels can be created on demand in SCC

### Concerns and Unresolved Questions

* should migration targets be manual only?
  - all updates manual for now
* categorize updates
  - when to put an update in a separate channel
    - adhoc (as needed by specific package)
  - how long is the overlapped support for a channel
    - couple of months
* use (constantly updated) release notes for normal updates as well
  - can be a short/readable link to the suse.com site e.g. (problem internet connection)
  - no release notes shipping mechanism

#### Manual/automatic update strategy

* customer can choose conservative or progressive update strategy by simply enabling/disabling timers?

## Alternatives

None known at the moment.

## Revision History:

| Date       | Comment       |
|:-----------|:--------------|
| 2018-07-31 | Initial Draft |
