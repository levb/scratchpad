# Overview

## Problem Statement

The current extensibility framework supports http-based **Integrations** and
**Plugins**.

1. **Integrations** are based on simple http1 messages, and provide very basic
   interactive functionality, and are not easily hostable on-prem. "As is" this
   framework is not sufficient to develop most interactive in-place use-cases.
   They are mostly supported on mobile.
2. **Plugins** do not offer significant capabilities on mobile. Historically,
   they have been implemented with WebApp first in mind.
3. **Plugins** require development in Go, React; expensive and slow to develop
   and review.
4. **Plugin** installation and configuration management UX is neither
   consolidated, nor consistent.
5. **Plugins** are not sufficiently isolated neither in terms of data access,
   nor in terms of performance for cloud (and some enterprise) deployments.
6. **Slack** compatibility has not been actively maintained, is behind.

## Project Summary 

**App Marketplace** will be the place to discover, install/upgrade/downgrade,
and configure **Apps** from.  **App Marketplace** will be largely based on the
existing **Plugin Marketplace**, but needs to be significantly extended in
functionality. Specifically, supporting **Integrations** and **Cloud Apps**, as
well as configuration - need to be implemented.

**App** will be a new single, installable/configurable listing, combining the
elements of **Integrations** (webhooks, commands), **Plugin**, and **Cloud App**
into a single deployable/configurable package.

**Cloud Apps** will be http-based, interactive services capable of implementing
content sharing, in-place viewing and editing integration with other 3rd party
services. **Cloud Apps** will be designed as "mobile first". In spite of the
name, **Cloud Apps** the long-term objective is to support **Cloud Apps** in
on-prem deployments.

**Cloud Apps** will likely benefit from new services/capabilities
added/refactored in the core Mattermost product. #TODO. 

Launch and migration: #TODO.

## Objectives - 
### Minimal supported (mobile and WebApp) use-cases 
1. **UX Hooks** - Invite channel members to a call, from a UX element in channel
2. **Dynamic interactive data inputs** - Share a post/thread upstream from the
   Post menu, specify additional (dynamic) metadata interactively
3. Interactive **subscription notifications** (a.k.a. incoming webhooks) -
   Subscribe to Google Calendar events, with *Accept/Decline with Note* and
   *Propose a Different time* from the notification message
4. **Dashboard** - a widget (LHS/RHS equiv) #TODO

### App Marketplace
1. Contains a full set of **Plugins**, **Integrations**, and **Cloud Apps**.
2. 10+5 New **Cloud Apps** at launch.
3. Interactive (cloud-apps only) install and configuration within the
   marketplace UX.

### **Cloud Apps** Hosting, Performance, Reliability
1. Hostable as multi-tenant by the upstream application provider, or as
   single-tenant in Mattermost Cloud.
2. In Mattermost Cloud, packaged for fast deployment (k8s service?,
   serverless?).
3. In Mattermost Cloud, 95% 100ms target response time (first-byte sent to last
   byte received) for user actions (in addition to the upstream R/T time).
4. Mattermost Cloud-first design for monitoring, logging, introspection
5. #STRETCH on-prem hostable (dependencies???)
6. Storage, other service dependencies TBD, encourage the use of props in MM
   data model?

### Scoped **Cloud App** access
1. Introduce capability scope declaration and enforcement for **Cloud Apps**
   #TODO

# **Cloud App** Architecture

## Ideas
1. All actions are commands. Do not use Post Actions, direct REST API
   invocations ("create-issue").
1. Replace "responses" with the use of REST APIs
1. Standardize "auto-complete"/search request/response formats.
1. Add `--json` way of providing command input/output encoded.
1. Ensure TCP connection reuse between Mattermost and **Cloud Apps**. #STRETCH
   experiment with HTTP2.
1. Outgoing webhooks use a JWT, provide the API token for responding, with
   limited scope
1. Encourage the use of props on User, Channel, Post (any other entities?)

## Commands
- **Command** is a fundamental unit of executing specific user instructions by a
  **Cloud App**.
- The protocol is to be simplified, so **Commands** return a simple response
  message to instruct immediate client action: e.g. open an Interactive Dialog
  or go to a URL. Creating and updating Posts is to be done via the REST API.
- 

### /-commands
**Commands** are traditionally submitted from the command line, by typing a `/<trigger>`. 

### Embedded **commands** (**Post Action** handlers)
1. Post Actions represent a way of encoding commands into the UX
1. **PostActions** (buttons and selects) and **Interactive dialogs** submit commands. 

### Command Context and Encoding
1. V1 HTTP commands are sent as HTTP POSTs with form encoding, see [Slash Commands - Basic Usage](https://developers.mattermost.com/integrate/slash-commands/#basic-usage).
2. V2 HTTP commands will be sent as JSON, with an expandable Context. Example:
```http
POST / HTTP/1.1
Accept-Encoding: gzip
Accept: application/json
Authorization: Token nezum4kpu3faiec7r7c5zt6tfy     #TODO use JWT
Content-Length: xxx
Content-Type: application/json
Host: 127.0.0.1:8080
User-Agent: mattermost-5.xxx
```
```json
{
    "command":{
        "namespace":"jira",
        "function":"create_issue",
        "raw":"/jira create issue --project=MM ...",
        "encoded":{
            "project":"MM",
            "...":"..."
        }
    },
    "security":{
        "token":"xxx #TODO use JWT",
        "trigger_id":"xxx #TODO use JWT",
    },
    "context":{
        "channel":{},
        "post":{},
        "parent_post":{},
        "root_post":{},
        "source":{
            "id":"post_menu_xxx",
            "props":{},
        },
        "team":{},
        "user":{}
    },
}
```

## Webhooks (Post-event notifications)
- MessagePosted (existing webhook)
- ChannelCreated
- UserCreated
- UserJoinedChannel
- UserJoinedTeam
- UserLeftChannel
- UserLeftTeam
- UserUpdated



### Interactive attachments
1. Post Actions represent a way of encoding commands into the UX
1. **PostActions** (buttons and selects) and **Interactive dialogs** submit commands. 



- action requests
- configuration metadata

### Application structure


## Cloud App Marketplace



## Deliverables

# Resources

## Dependency services
## On-prem ho
## Storage
## High Availabil

# Questions
- Posting into channel: Incoming Webhooks vs API vs Response, are incoming webhooks useful?
- "Outgoing webhooks
- "expand" model for outgoing webhooks
- "HA", storage, cache invalidation/update for self-hosted 
- Cloud - gradual rollout version management


# Tasks

## Command
### Dynamic argument definition
- Know what options are relevant in the context
### Autocomplete types
- Date picker
- Plain text (many versions?)
- ...

