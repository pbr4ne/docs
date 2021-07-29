---
title: API reference
showNav: false
layout: api
---

import { Attributes, Attribute } from "../../components/Attributes";
import { Endpoints, Endpoint } from "../../components/Endpoints";
import {
  Section,
  ContentColumn,
  ExampleColumn,
} from "../../components/ApiSections";

<Section title="API Reference" slug="overview">
<ContentColumn>

The Knock API enables you to add a complete notification engine to your product. This API provides
programmatic access to integrating Knock via a REST-ful API.

</ContentColumn>
<ExampleColumn>

```bash Base URL
https://api.knock.app/v1
```

</ExampleColumn>

</Section>

<Section title="Client libraries" slug="client-libraries">
<ContentColumn>

Knock offers native SDKs in several popular programming languages:

- [Node JS](https://github.com/knocklabs/knock-node)
- [Python](https://github.com/knocklabs/knock-python)
- [Elixir](https://github.com/knocklabs/knock-elixir)

</ContentColumn>
</Section>

<Section title="API keys" slug="api-keys">
<ContentColumn>

Knock authenticates your API requests using your account's API keys. API requests made without authentication or using an incorrect key will return a 401 error. Requests using a valid key but with insufficient permissions will return a 403 error.

You can view and manage your API keys in the Developer Dashboard. There are two types of API keys:

- Publishable keys are only meant to identify your account with Knock. They aren't secret, and can safely be made public in any of your client-side code. Publishable keys are prefixed with `pk_*`.

- Secret keys can perform any API request to Knock, they should be kept secure and private! Be sure to prevent secret keys from being made publicly accessible, such as in client-side code, GitHub, unsecured S3 buckets, and so forth. Secret keys are prefixed with `sk_*`.

Each Environment in your account has both a publishable and secret key pair. API requests will be scoped to the provided key's Environment.

</ContentColumn>
</Section>

<Section title="Authentication" slug="authentication">
<ContentColumn>

You must pass your API key to Knock as a Bearer token using the following header:

```bash Authentication header
Authentication: Bearer sk_test_12345
```

</ContentColumn>
</Section>

<Section title="Errors" slug="errors">
<ContentColumn>

Knock uses standard [HTTP response codes](https://developer.mozilla.org/en-US/Web/HTTP/Status) to indicate the success or failure of your API requests.

- `2xx` - Indicates success.
- `4xx` - Indicates an error, normally due to error caused by incorrect or missing request information (e.g. providing an incorrect API key).
- `5xx` - Indicates a Knock server error.

</ContentColumn>
</Section>

<Section title="Users" slug="users">
<ContentColumn>

A user represents an individual who may need to receive a notification from Knock. They are always referenced by your internal identifier.

### Attributes

<Attributes>
  <Attribute
    name="id"
    type="string"
    description="Unique identifier for the user"
  />
  <Attribute name="name" type="string" description="The full name of the user" />
  <Attribute name="email" type="string" description="The email of the user" />
  <Attribute
    name="*"
    type="key-value pairs"
    description="Any custom properties"
  />
</Attributes>
</ContentColumn>
<ExampleColumn>

<Endpoints>
  <Endpoint name="identify-user" method="PUT" path="/users/:user_id" />
  <Endpoint name="get-user" method="GET" path="/users/:user_id" />
  <Endpoint name="delete-user" method="DELETE" path="/users/:user_id" />
</Endpoints>

```json User object
{
  "__typename": "User",
  "id": "user_1",
  "name": "User name",
  "email": "user@example.com",
  "foo": "bar",
  "baz": true,
  "created_at": null,
  "updated_at": "2021-03-05T12:00:00Z"
}
```

</ExampleColumn>
</Section>

<Section title="Identify a user" slug="identify-user">
<ContentColumn>

Identifying a user will create or update a user in Knock, merging the properties given with what we currently have set on the user (if any).

### Endpoint

<Endpoint method="PUT" path="/users/:user_id" />

### Path parameters

<Attributes>
  <Attribute
    name="id"
    type="string"
    description="Unique identifier of the user"
  />
</Attributes>

### Body parameters

<Attributes>
  <Attribute name="name" type="string" description="The name of the user" />
  <Attribute name="email" type="string" description="The email of the user" />
  <Attribute
    name="*"
    type="key-value pairs"
    description="Any custom properties"
  />
</Attributes>

### Returns

A User.

</ContentColumn>
<ExampleColumn>

```javascript Identify user
import Knock from "@knocklabs/node";

const knock = new Knock("sk_example_123456789");

const user = await knock.users.identify("jhammond", {
  name: "John Hammond",
  email: "jhammond@ingen.net",
});
```

```json Response
{
  "__typename": "User",
  "id": "user_1",
  "name": "User name",
  "email": "user@example.com",
  "foo": "bar",
  "baz": true,
  "created_at": null,
  "updated_at": "2021-03-05T12:00:00Z"
}
```

</ExampleColumn>
</Section>

<Section title="Get a user" slug="get-user">
<ContentColumn>

Retrieve a user by their ID, including all properties previously set.

### Endpoint

<Endpoint method="GET" path="/users/:user_id" />

### Path parameters

<Attributes>
  <Attribute
    name="id"
    type="string"
    description="Unique identifier of the user"
  />
</Attributes>

### Returns

A User.

</ContentColumn>
<ExampleColumn>

```javascript Get user
import Knock from "@knocklabs/node";

const knock = new Knock("sk_example_123456789");

const user = await knock.users.get("jhammond");
```

```json Response
{
  "__typename": "User",
  "id": "user_1",
  "name": "User name",
  "email": "user@example.com",
  "foo": "bar",
  "baz": true,
  "created_at": null,
  "updated_at": "2021-03-05T12:00:00Z"
}
```

</ExampleColumn>
</Section>

<Section title="Workflows" slug="workflows">
<ContentColumn>

A Workflow orchestrates the delivery of messages to your end users. When you configure a workflow you'll determine which channels its messages should route to, what those messages should look like on each channel, as well as any functions—batch, throttle, digest—you want applied to the messages prior to delivery. A workflow is triggered by a `notify` call, usually when something occurs in your product that you want your users to know about (e.g. a new comment.)

</ContentColumn>
<ExampleColumn>
<Endpoints>
  <Endpoint
    name="trigger-workflow"
    method="POST"
    path="/workflows/:key/trigger"
  />
  <Endpoint
    name="cancel-workflow"
    method="POST"
    path="/workflows/:key/cancel"
  />
</Endpoints>

</ExampleColumn>
</Section>

<Section title="Triggering a workflow (notify)" slug="trigger-workflow">
<ContentColumn>

A notify calls a workflow created via the Knock dashboard.

### Endpoint

<Endpoint method="POST" path="/workflows/:key/trigger" />

### Path parameters

<Attributes>
  <Attribute
    name="key"
    type="string"
    description="The key of the workflow to call"
  />
</Attributes>

### Body parameters

<Attributes>
  <Attribute
    name="recipients"
    type="string[]"
    description="A list of user ids of who should be notified"
  />
  <Attribute
    name="actor"
    type="string"
    description="The id of the actor who performed the action associated"
  />
  <Attribute
    name="cancellation_key"
    type="string (optional)"
    description="The key to use for canceling the workflow"
  />
  <Attribute
    name="tenant"
    type="string (optional)"
    description="The id of the tenant that this workflow run should be associated with"
  />
  <Attribute
    name="data"
    type="dictionary"
    description="A set of key value pairs to pass to the notify"
  />
</Attributes>

### Returns

<Attributes>
  <Attribute
    name="workflow_run_id"
    type="string"
    description="A unique UUID of this workflow run"
  />
</Attributes>

</ContentColumn>
<ExampleColumn>

```javascript Trigger workflow
import Knock from "@knocklabs/node";

const knock = new Knock("sk_example_123456789");

await knock.notify("dinosaurs-loose", {
  actor: "dnedry",
  recipients: ["jhammond", "agrant", "imalcolm", "esattler"],
  data: {
    type: "trex",
    priority: 1,
  },
  tenant: "jurassic-park",
  cancellationKey: triggerAlert.id,
});
```

```json Response
{
  "workflow_run_id": "some-result-id"
}
```

</ExampleColumn>
</Section>

<Section title="Canceling workflows" slug="cancel-workflow">
<ContentColumn>

Cancel a delayed workflow for one or more recipients.

### Endpoint

<Endpoint method="POST" path="/workflows/:key/cancel" />

### Path parameters

<Attributes>
  <Attribute name="key" type="string" description="The key of the workflow" />
</Attributes>

### Body parameters

<Attributes>
  <Attribute
    name="cancellation_key"
    type="string"
    description="The cancellation key unique to the notify"
  />
  <Attribute
    name="recipients"
    type="string[]"
    description="An optional list of user ids to cancel the workflow for"
  />
</Attributes>

### Returns

`204`, with empty content.

</ContentColumn>
<ExampleColumn>

```javascript Cancel workflow
import Knock from "@knocklabs/node";

const knock = new Knock("sk_example_123456789");

await knock.workflows.cancel("dinosaurs-loose", triggerAlert.id);
```

</ExampleColumn>
</Section>

<Section title="Preferences" slug="preferences">
<ContentColumn>

A preference determines whether a user should receive a particular type of notification. By default
all preferences are opted in unless a preference explicitly opts the user out of the notification.

Preferences are always executed as channel types and then workflows. That means that if a channel type
is set to opt out the user then no notifications will be sent on that channel.

### Attributes

<Attributes>
  <Attribute
    name="id"
    type="string"
    description="Unique identifier for the preference set"
  />
  <Attribute
    name="workflows"
    type="object"
    description="A set of preferences for workflows, each can resolve to a boolean or to a set of channel types"
  />
  <Attribute
    name="channel_types"
    type="object"
    description="A set of preferences for channel types"
  />
</Attributes>

</ContentColumn>
<ExampleColumn>

<Endpoints>
  <Endpoint name="get-all" method="GET" path="/users/:user_id/preferences" />
  <Endpoint
    name="get-preferences"
    method="GET"
    path="/users/:user_id/preferences/:id"
  />
  <Endpoint
    name="set-preferences"
    method="PUT"
    path="/users/:user_id/preferences/:id"
  />
  <Endpoint
    name="set-channel-type-preferences"
    method="PUT"
    path="/users/:user_id/preferences/:id/channel_types"
  />
  <Endpoint
    name="set-channel-type-preference"
    method="PUT"
    path="/users/:user_id/preferences/:id/channel_types/:type"
  />
  <Endpoint
    name="set-workflow-preferences"
    method="PUT"
    path="/users/:user_id/preferences/:id/workflows"
  />
  <Endpoint
    name="set-workflow-preference"
    method="PUT"
    path="/users/:user_id/preferences/:id/workflows/:key"
  />
</Endpoints>

```json Preference object
{
  "__typename": "PreferenceSet",
  "id": "default",
  "workflows": {
    "dinosaurs-loose": {
      "channel_types": {
        "email": true
      }
    }
  },
  "channel_types": {
    "in_app_feed": true
  }
}
```

</ExampleColumn>
</Section>

<Section title="Get a preference set" slug="get-preferences">
<ContentColumn>

Retrieve a user's preference object. Will always return an empty preference object,
even if it does not currently exist for the user.

### Endpoint

<Endpoint
  name="get-preferences"
  method="GET"
  path="/users/:user_id/preferences/:id"
/>

### Path parameters

<Attributes>
  <Attribute
    name="user_id"
    type="string"
    description="Unique identifier for the user"
  />
  <Attribute
    name="id"
    type="string"
    description="Unique identifier for the preference set"
  />
</Attributes>

### Response

Returns a `PreferenceSet`.

</ContentColumn>
<ExampleColumn>

```javascript Get preferences
import Knock from "@knocklabs/node";
const knock = new Knock("sk_example_123456789");

// Retrieves the `default` set
const preferences = await knock.preferences.get("jhammond");
```

```json Response
{
  "__typename": "PreferenceSet",
  "id": "default",
  "workflows": {
    "dinosaurs-loose": {
      "channel_types": {
        "email": true
      }
    }
  },
  "channel_types": {
    "in_app_feed": true
  }
}
```

</ExampleColumn>
</Section>

<Section title="Set preferences" slug="set-preferences">
<ContentColumn>

Sets preferences within the given preference set. This is a destructive operation and
will replace any existing preferences with the preferences given.

### Endpoint

<Endpoint
  name="set-preferences"
  method="PUT"
  path="/users/:user_id/preferences/:id"
/>

### Path parameters

<Attributes>
  <Attribute
    name="user_id"
    type="string"
    description="Unique identifier for the user"
  />
  <Attribute
    name="id"
    type="string"
    description="Unique identifier for the preference set"
  />
</Attributes>

### Body parameters

<Attributes>
  <Attribute
    name="channel_types"
    type="object"
    description="A set of channel type preferences to set"
  />
  <Attribute
    name="workflows"
    type="object"
    description="A set of workflow preferences, can be a boolean or an object containing channel type preferences"
  />
</Attributes>

### Response

Returns a `PreferenceSet`.

</ContentColumn>
<ExampleColumn>

```javascript Set preferences
import Knock from "@knocklabs/node";
const knock = new Knock("sk_example_123456789");

const preferences = await knock.preferences.set("jhammond", {
  channel_types: {
    in_app_feed: false,
  },
  workflows: {
    "dinosaurs-loose": {
      channel_types: {
        email: true,
      },
    },
  },
});
```

```json Response
{
  "__typename": "PreferenceSet",
  "id": "default",
  "workflows": {
    "dinosaurs-loose": {
      "channel_types": {
        "email": true
      }
    }
  },
  "channel_types": {
    "in_app_feed": true
  }
}
```

</ExampleColumn>
</Section>

<Section title="In-app notification feeds" slug="feeds">
<ContentColumn>

A feed exposes the messages delivered to an in-app feed channel, formatted specially to be consumed
in a notification feed.

A feed will always return a list of `FeedItems`, which are pointers to a message delivered and contain
all of the information needed in order to render an item within a notification feed.

**Note: feeds are a specialized form of messages that are designed purely for in-app rendering, and
as such return information that is required on the client to do so**

### Attributes

<Attributes>
  <Attribute
    name="entries"
    type="FeedItem[]"
    description="An ordered list of feed items (most recent first)"
  />
  <Attribute
    name="page_info"
    type="PageInfo"
    description="Pagination information for the items returned"
  />
  <Attribute
    name="vars"
    type="object"
    description="Environment specific account variables"
  />
  <Attribute
    name="meta"
    type="FeedMetadata"
    description="Information about the total unread and unseen items"
  />
</Attributes>

</ContentColumn>
<ExampleColumn>

<Endpoints>
  <Endpoint method="GET" path="/users/:user_id/feeds/:feed_id" />
</Endpoints>

```json Response
{
  "entries": [
    {
      "__typename": "FeedItem",
      "__cursor": "g3QAAAABZAACaWRtAAAAGzFzTXRJc1J2WnRZZjg2YU9ma00yUENwQzZYYw==",
      "activities": [
        {
          "__typename": "Activity",
          "actor": {
            "__typename": "User",
            "id": "c121a5ea-8f2c-4c60-ab40-9966047d5bea",
            "created_at": null,
            "updated_at": "2021-05-08T20:40:01.340Z",
            "email": "some-user@knock.app",
            "name": "Some User"
          },
          "data": {
            "dest_environment_name": "Production",
            "src_environment_name": "Development",
            "total_merged": 1
          },
          "id": "1sMtIwNnDIV52a8G8kmymzDVExQ",
          "inserted_at": "2021-05-11T00:50:09.895759Z",
          "recipient": {
            "__typename": "User",
            "id": "c121a5ea-8f2c-4c60-ab40-9966047d5bea",
            "created_at": null,
            "updated_at": "2021-05-08T20:40:01.340Z",
            "email": "some-user@knock.app",
            "name": "Some User"
          },
          "updated_at": "2021-05-11T00:50:09.895759Z"
        }
      ],
      "actors": [
        {
          "__typename": "User",
          "id": "c121a5ea-8f2c-4c60-ab40-9966047d5bea",
          "created_at": null,
          "updated_at": "2021-05-08T20:40:01.340Z",
          "email": "some-user@knock.app",
          "name": "Some User"
        }
      ],
      "archived_at": null,
      "blocks": [
        {
          "content": "**{{ actor.name }}** merged {{ total_merged }} {% if total_merged == 1 %} change {% else %} changes {% endif %}\nfrom **{{ src_environment_name }}** into **{{ dest_environment_name }}**.",
          "name": "body",
          "rendered": "<p><strong>The person</strong> merged 1  change \nfrom <strong>Development</strong> into <strong>Production</strong>.</p>",
          "type": "markdown"
        },
        {
          "content": "{{ vars.app_url }}/{{ account_slug }}/commits",
          "name": "action_url",
          "rendered": "https://example.com/thing/commits",
          "type": "text"
        }
      ],
      "data": {
        "dest_environment_name": "Production",
        "src_environment_name": "Development",
        "total_merged": 1
      },
      "id": "1sMtIsRvZtYf86aOfkM2PCpC6Xc",
      "inserted_at": "2021-05-11T00:50:09.904531Z",
      "read_at": "2021-05-13T02:45:28.559124Z",
      "seen_at": "2021-05-11T00:51:43.617550Z",
      "source": {
        "__typename": "WorkflowSource",
        "key": "merged-changes",
        "version_id": "7251cd3f-0028-4d1a-9466-ee79522ba3de"
      },
      "tenant": null,
      "total_activities": 1,
      "total_actors": 1,
      "updated_at": "2021-05-13T02:45:28.559863Z"
    }
  ],
  "vars": {
    "app_name": "The app name"
  },
  "meta": {
    "__typename": "FeedMetadata",
    "unread_count": 0,
    "unseen_count": 0
  },
  "page_info": {
    "__typename": "PageInfo",
    "after": null,
    "before": null,
    "page_size": 50
  }
}
```

</ExampleColumn>
</Section>

<Section title="Get feed for user" slug="get-feed">
<ContentColumn>

Retrieves a feed of items for a given `user_id` on the given `feed_id`.

**Note: if you're making this call from a client-side environment you should be using your publishable key
along with a user token to make this request**

### Endpoint

<Endpoint method="GET" path="/users/:user_id/feeds/:feed_id" />

### Path parameters

<Attributes>
  <Attribute name="user_id" type="string" description="The ID of the user" />
  <Attribute
    name="feed_id"
    type="string"
    description="The ID of the feed (the channel ID)"
  />
</Attributes>

### Query parameters

<Attributes>
  <Attribute
    name="page_size"
    type="number"
    description="The total number to retrieve per page (defaults to 50)"
  />
  <Attribute
    name="after"
    type="string"
    description="The cursor to retrieve items after (hint: use the `__cursor` field)"
  />
  <Attribute
    name="before"
    type="string"
    description="The cursor to retrieve items before (hint: use the `__cursor` field)"
  />
  <Attribute
    name="source"
    type="string (optional)"
    description="Limits the feed to only items of the source workflow"
  />
  <Attribute
    name="tenant"
    type="string (optional)"
    description="Limits the feed to only display items with the corresponding tenant, or where the tenant is empty"
  />
  <Attribute
    name="status"
    type="string (optional)"
    description="One of `unread`, `unseen`, `all`"
  />
</Attributes>

### Returns

A feed response.

</ContentColumn>
</Section>

<Section title="Messages" slug="messages">
<ContentColumn>

A message is a notification delivered on a particular channel to a user.

### Attributes

<Attributes>
  <Attribute
    name="id"
    type="string"
    description="The unique ID of this message"
  />
  <Attribute
    name="channel_id"
    type="string"
    description="The ID of the channel for where this message was delivered"
  />
  <Attribute
    name="recipient"
    type="string"
    description="The ID of the user who received the message"
  />
  <Attribute
    name="workflow"
    type="string"
    description="The key of the workflow that this message was generated from"
  />
  <Attribute
    name="tenant"
    type="string"
    description="The ID of the tenant that this message belongs to"
  />
  <Attribute
    name="status"
    type="string"
    description="One of `queued`, `sent`, `delivered`, `undelivered`, `seen`, `unseen`"
  />
  <Attribute
    name="read_at"
    type="string (optional)"
    description="When the message was last read"
  />
  <Attribute
    name="seen_at"
    type="string (optional)"
    description="When the message was last seen"
  />
  <Attribute
    name="archived_at"
    type="string (optional)"
    description="When the message was archived"
  />
</Attributes>

</ContentColumn>
<ExampleColumn>

<Endpoints>
  <Endpoint method="PUT" path="/messages/:id/:status" />
  <Endpoint method="DELETE" path="/messages/:id/:status" />
  <Endpoint method="POST" path="/messages/batch/:status" />
</Endpoints>

```json Message object
{
  "__typename": "Message",
  "id": "1rjI9XBgWQ6EUA3D3Ul3VjUimOD",
  "channel_id": "0bfd9f86-56b0-41f0-ade3-dc5cc6a69bb8",
  "recipient": "user_12345",
  "workflow": "new-comment",
  "tenant": null,
  "status": "delivered",
  "read_at": null,
  "seen_at": null,
  "archived_at": null,
  "inserted_at": "2021-03-05T12:00:00Z",
  "updated_at": "2021-03-05T12:00:00Z"
}
```

</ExampleColumn>
</Section>

<Section title="Updating a message status" slug="update-message-status">
<ContentColumn>

Marks the given message as the `status`, recording an event in the process.

### Endpoint

<Endpoint method="PUT" path="/messages/:id/:status" />

### Path parameters

<Attributes>
  <Attribute name="id" type="string" description="The ID of the message" />
  <Attribute
    name="status"
    type="enum"
    description="One of `seen`, `read`, `archive`"
  />
</Attributes>

### Returns

A Message.

</ContentColumn>
</Section>

<Section title="Undoing a message status change" slug="undo-message-status">
<ContentColumn>

Un-marks the given message as the `status`, recording an event in the process.

### Endpoint

<Endpoint method="DELETE" path="/messages/:id/:status" />

### Path parameters

<Attributes>
  <Attribute name="id" type="string" description="The ID of the message" />
  <Attribute
    name="status"
    type="enum"
    description="One of `seen`, `read`, `archived`"
  />
</Attributes>

### Returns

A Message.

</ContentColumn>
</Section>

<Section title="Batch changing message statuses" slug="batch-update-message-status">
<ContentColumn>

Messages changes can also be made in bulk by using the batch API, where all message ids given in
the list will have their statuses changed.

### Endpoint

<Endpoint method="POST" path="/messages/batch/:status" />

### Path parameters

<Attributes>
  <Attribute
    name="status"
    type="enum"
    description="One of `seen`, `read`, `archived` or `unseen`, `unread`, `unarchived`"
  />
</Attributes>

### Body parameters

<Attributes>
  <Attribute
    name="message_ids"
    type="string[]"
    description="A list of one or more message IDs"
  />
</Attributes>

### Returns

A list of Messages that were mutated during the call.

</ContentColumn>
</Section>