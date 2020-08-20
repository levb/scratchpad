# Tasks

* [Slash Commands](#slash-commands)
* [Embedded commands](#embedded-commands)
* [Interactive commands](#interactive-commands)
* [Storage](#storage)
* [Mattermost Cloud](#mattermost-cloud)
   + [Packaging and Deployment](#packaging-and-deployment)
   + [Monitoring](#monitoring)
* [Mattermost Server](#mattermost-server)
   + [Plugin API](#plugin-api)

## Slash Commands

### Support JSON in/out

### Dynamic argument definition
- Know what options are relevant in the context

### Autocomplete types
- Date picker
- Plain text (many versions?)
- ...

### Command text input 
- With (filtered) autocomplete
- 

## Embedded commands

## Interactive commands

## Storage
- Research tradeoffs of relying on props on User, Channel, Team
- Still need persistence, cache with invalidation, background jobs, stateful timers (reminders)
- Should Cloud Apps using 3rd paty services explicitly declare them? Enforcement?

## Mattermost Cloud

### Packaging and Deployment 
### Monitoring

## Mattermost Server

### Plugin API
- Is StorePluginConfig robust enough? Support individual keys? #TODO research, how else would Cloud Apps store config?
