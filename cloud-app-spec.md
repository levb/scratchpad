# Introduction

## Problem Statement

The current extensibility framework supports http-based **Integrations** and
Go/React **Plugins**.

1. **Integrations** are based on simple HTTP1.1 messages, and provide very basic
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
well as in-place configuration - need to be implemented.

**App Listing** will be a new single, installable/configurable listing, combining the
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
1. Launch with 10+5 New **Cloud Apps**. #TODO list
2. Launch as a full set of **Plugins**, **Integrations**, and **Cloud Apps**.
3. Interactive (cloud-apps only) in-place install and configuration, integrated
   into the marketplace UX.

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

# Architecture

## Ideas/Principles
1. All User Actions Are Commands
   1. Slash-commands, submissions from Post Actions and Interactive Dialogs, and
      from Mattermost UX elements are commands.
   1. Largely deprecate "responses" functionality, encourage using the REST APIs
   1. Standardize **Autocomplete query API**.
   1. Add `--json` way of providing command input/output encoded.
1. Outgoing webhooks use a JWT, provide the Mattermost API token for responding,
   with limited scope
1. Encourage the use of props on User, Channel, Post (any other entities?)
1. Ensure TCP connection reuse between Mattermost and **Cloud Apps**. #STRETCH
   experiment with HTTP2. May be non-trivial, but subject to "off-the shelf"
   tooling?

## **Cloud Apps**

### Overview

### Authentication 

1. When a **Cloud App** is installed, it receives "bot credentials" that it can
   use to invoke Mattermost REST APIs as a bot.
   - Q: can this all be handled via bot accounts?
1. When a **Cloud App** is installed, it gets a shared secret it can use to
   decode a Mattermost JWT.
1. Commands sent to the **Cloud App** include a JWT. The JWT contains a
   user-scoped Mattermost REST API token. It should then use the token to act on
   behalf of the user when posting back to Mattermost.
   - Q: What if the Cloud App wants to post back as the bot? Seems no problem,
     just use the correct API token.
1. Post-event notifications sent to the **Cloud App** include a JWT. The JWT
   contains a bot-scoped Mattermost REST API token.


### Commands and interactivity
1. A **Command** is a fundamental unit of executing specific user instructions
   by a **Cloud App**. They are "embeddable" as Post Actions, and "interactive"
   as Interactive Dialogs.
1. The protocol is to be simplified, so **Commands** return a simple response
   message to instruct immediate client action: e.g. open an Interactive Dialog
   or go to a URL. Creating and updating Posts is to be done via the REST API.
- 

#### Slash commands
**Commands** are traditionally submitted from the command line, by typing a
`/<trigger>`. Autocomplete functionality will be extended to improve dynamic
definition.

### Embedded commands (Post Action handlers)
- Post Actions represent a way of encoding 1-click commands into posts.
- The functionality will be adjusted to match slash-command autocomplete
- Can we implement "embedded interactive dialogs" to submit several fields at
  once?

### Interactive commands (Interactive dialogs and Bot Conversations)

#### Interactive dialogs
1. Can be directly launched from all relevant UX locations.
1. Can be launched as a result of another command.
1. Can be pre-configured with initial set of fields/values
2. Can dynamically fetch relevant field data and reconfigure, including the set of fields displayed, based on the user inputs (**Autocomplete Query API**)
3. In the end, submits a command (or Cancel)

#### Bot Conversations
1. Can be directly launched from all relevant UX locations.
1. Can be launched as a result of another command.
2. Each step is a call to the **Cloud App**
3. In the end, submits a command (or Cancel)


### Command Context and Encoding
1. V1 HTTP commands are sent as HTTP POSTs with form encoding, see [Slash
   Commands - Basic
   Usage](https://developers.mattermost.com/integrate/slash-commands/#basic-usage).
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
1. **PostActions** (buttons and selects) and **Interactive dialogs** submit
   commands. 



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
- Posting into channel: Incoming Webhooks vs API vs Response, are incoming
  webhooks useful?
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



# Glossary

## Legacy Terms
1. **Integrations** - "legacy" slack-inspired integrations, mostly consisting of slash-commands, webhooks, and slack attachments. Slack attachments support **Post Actions**.
2. **Interactive Dialogs** 
3. **Plugin Marketplace**
4. **Integrations directory**

## New Terms

