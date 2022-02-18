---
title: Objects
---

An [Object](/reference#objects) represents a resource in your system that you want to map into Knock.

In this guide we'll walk through how to use objects for a few different use cases in Knock.

We'll start with an overview of objects and how to use them, then we'll walk through two common use cases for objects: Slack channel notifications and handling mutable data on long-running notifications (such as digests).

**Note:** Objects are an advanced feature within Knock. You can send multi-channel notifications across all channel types (except Slack) without touching the Objects API. If you're just getting started, we'd recommend coming back to objects when you've already started to leverage a few channels using Knock.

## An overview of objects

Objects are a powerful and flexible way to ensure Knock always has the most up-to-date information required to send your notifications. They also enable you to send notifications to non-user recipients.

You can use objects to:

- send out-of-app notifications to non-user recipients (such as a Slack channels)
- send in-app notifications to non-user resources in your product (the activity feed you see on a Notion page is a good example)
- reference mutable data in your notification templates (such as when a user edits a comment before a notification is sent)

**Note:** As of today, our objects API support for in-app notifications and mutable data referencing is in beta. If you're interested in using this functionality, please reach out and let us know via the feedback button at the top of this page.

## The Knock objects API

All objects belong to `collections`, which group objects of the same type together. An object should be unique within a collection, identified by the `id` given. We use the `{collection, id}` pair to know when to create or update an object.

Objects follow the same rules as all other items in Knock in that they are unique and logically separated per Knock environment.

```javascript Set an object in Knock
import { Knock } from "@knocklabs/node";
const knockClient = new Knock(process.env.KNOCK_API_KEY);

knockClient.objects.set("projects", "project-1", {
  name: "My project",
  total_assets: 10,
  tags: ["cool", "fun", "project"],
});
```

## Guidelines for use

### Collection naming

You should use plural collection names wherever possible. The collection name should describe the group of one or many objects within the collection. Good examples of collection names are `projects`, `teams`, `accounts`.

### The object identifier

The object `id` should be unique within the collection. It should also be a stable identifier, likely the primary key of the object in your system so it can be easily referenced later. Please note: object ids **cannot be changed once set**.

### Properties

Objects can contain any number of key-value property pairs that you can then reference in templates and trigger conditions. Properties will always be shallowly merged between set calls, meaning that existing properties will be overwritten.

## Examples

### Slack channel notifications

A common notification use case we see in SaaS applications is the ability for users to connect a object in the product they're using to a channel in their own Slack workspace. That way when something happens in that object (e.g. a comment is left) they receive a notification about it in their connected Slack channel.

Let's take a fictional example here where we have an audio collaboration service that allows its customers to connect a Project object to a Slack channel. Once the Project and Slack channel are connected, all Comments left within the Project will result in notifications sent to the customer's Slack channel.

Here's how we'd use Knock objects to solve this.

1. **Register our Project object to Knock**

Typically whenever the project is created or updated we'll want to send it through to Knock.

```javascript Send project object to Knock
await knock.objects.set("projects", project.id, {
  name: project.name,
});
```

2. **Store the Slack connection information for the Project**

Once our customer chooses to connect their Slack channel to the Project, we have a callback that then adds the Slack information as Channel Data.

```javascript Store Slack connection on object
// Connect Slack data to our Project object in Knock so that we can
// send notifications about this project to our customer's Slack channel
await knock.objects.setChannelData(
  "projects",
  project.id,
  process.env.KNOCK_SLACK_CHANNEL_ID,
  {
    connections: [
      {
        incoming_webhook: { url: "url-from-slack" }
      },
    ],
  },
);
```

3. **Add Slack as a step to our workflow**

Inside of the Knock dashboard, we're going to add a new Slack step to our `new-comment` workflow that will send a notification displaying the comment that was left in our product.

4. **Send the Project as a recipient in your workflow trigger**

Now when we trigger our `new-comment` workflow, we also want to add our Project object as a recipient so that the newly added Slack step will be triggered.

```javascript Workflow trigger with an object
await knock.workflows.trigger("new-comment", {
  actor: comment.authorId,
  recipients: [...projectUserIds, { id: project.id, collection: "projects" }],
  data: { comment },
});
```

Knock then executes the workflow for this Project object as it would for any user recipients sent in the workflow trigger, skipping over any steps that aren't relevant. (In this case, the Project object only has one piece of channel data mapped to it—the Slack channel—so it won't trigger notifications for any other channel steps in our `new-comment` workflow.) When the Slack step is reached, the connection information we stored earlier will be used as a means to know which channel to send a message to and how to authenticate to that channel.