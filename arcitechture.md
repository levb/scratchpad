# Architecture.

* [**Cloud Apps**.](#--cloud-apps--)
   + [Overview.](#overview)
   + [Authentication with Mattermost.](#authentication-with-mattermost)
   + [Authentication with Upstream Applications.](#authentication-with-upstream-applications)
   + [Commands and interactivity.](#commands-and-interactivity)
   + [Event notifications.](#event-notifications)
   + [Installation.](#installation)
   + [Configuration.](#configuration)
   + [mattermost-cloud-app.json](#mattermost-cloud-appjson)
* [Cloud Apps in Mattermost Server](#cloud-apps-in-mattermost-server)
* [Cloud App Marketplace](#cloud-app-marketplace)

## **Cloud Apps**.

### Overview.
1. Mattermost interacts with **Cloud Apps** in 2 ways: **Commands** that have request/response semantics, and 1-way **Event notifications**.
2. Cloud Apps interact with Mattermost primarily by using the Mattermost REST APIs. The "Response" functionality of command handlers will be greatly simplified for Cloud Apps.

### Authentication with Mattermost.

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

### Authentication with Upstream Applications.
1. #TODO.

### Commands and interactivity.
1. A **Command** is a fundamental unit of executing specific user instructions
   by a **Cloud App**. They are "embeddable" as Post Actions, and "interactive"
   as Interactive Dialogs.
1. The protocol is to be simplified, so **Commands** return a simple response
   message to instruct immediate client action: e.g. open an Interactive Dialog
   or go to a URL. Creating and updating Posts is to be done via the REST API.

#### Slash commands.
**Commands** are traditionally submitted from the message box, by typing a
`/<trigger>`. Autocomplete functionality will be extended to improve dynamic
definitions and new data types (date/time, plain text, json, file, etc).

#### Embedded commands.
- Post Actions represent a way of encoding 1-click, fully pre-configured
  commands into Posts (Slack Attachments).
- The functionality will be adjusted to match slash-command autocomplete
- Can we implement "embedded interactive dialogs" to submit several fields at
  once?

#### Interactive commands.
1. Interactive dialogs.
   1. Can be directly launched from all relevant UX locations.
   1. Can be launched as a result of another command.
   1. Can be pre-configured with initial set of fields/values.
   2. Can dynamically fetch relevant field data and reconfigure, including the set of fields displayed, based on the user inputs (**Autocomplete Query API**).
   3. In the end, submits a command (or Cancel).
1. Bot Conversations.
   1. Can be directly launched from all relevant UX locations.
   1. Can be launched as a result of another command.
   2. Each step is a call to the **Cloud App**.
   3. In the end, submits a command (or Cancel).

#### Command Request.
Legacy HTTP slash commands are sent as HTTP POSTs with form encoding, see [Slash
Commands - Basic
Usage](https://developers.mattermost.com/integrate/slash-commands/#basic-usage).

V2 HTTP commands will be sent as JSON, with an expandable Context. Example:

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
        "trigger_id":"xxx #TODO use JWT"
    },
    "context":{
        "source":{
            "id":"post_menu_xxx",
            "client_or_session_id_to_send_websockets_to":"xxx",
            "props":{}
        },
        "config":{
           "settingX":"value"
        },
        "channel":{},
        "post":{},
        "parent_post":{},
        "root_post":{},
        "team":{},
        "acting_user":{},
        "mentioned":{}
    },
}
```
#### Command Request Expansion: 
- #TODO.

#### Command Response.

### Event notifications.
- MessagePosted (matches the existing webhook)
  Must be scoped to: User, Channel.
- ChannelCreated
- UserCreated
- UserJoinedChannel
- UserJoinedTeam
- UserLeftChannel
- UserLeftTeam
- UserUpdated

### Installation.
- Performs all necessary registrations and generates/stores
- Run the "install" command

### Configuration. 
- Expose pre-configured commands

### mattermost-cloud-app.json

## Cloud Apps in Mattermost Server

### Minimal Changes to the Core Code.
1. Implement as a "Cloud Apps" plugin (mandatory, pre-loaded).
2. Preserve the current plugin and integrations support "as is"

### mattermost-plugin-cloud-apps

## Cloud App Marketplace

