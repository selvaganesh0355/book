---
title: Webhooks
---

# Webhooks

Webhooks let you receive real-time notifications from other services when specific events happen. Instead of polling for changes, you provide a URL endpoint, and the service sends an HTTP request to your server when the event occurs. This is a common pattern in modern APIs, including GitHub, Stripe, Slack, and many others.

## How Webhooks Work

You register a webhook by giving the service a URL (for example, `https://api.graphql.guide/github-hook`). When a specified event happens—like a repository being starred—GitHub sends an HTTP POST request to your endpoint. The request contains a JSON payload describing the event.

For example, if you subscribe to the [`watch` event](https://developer.github.com/v3/activity/events/types/#watchevent) on a GitHub repository, you might receive a payload like this:

```json
{
  "action": "started",
  "repository": {
    "name": "guide",
    "full_name": "GraphQLGuide/guide",
    "watchers_count": 9
  },
  "sender": {
    "login": "lorensr",
    "type": "User",
    "html_url": "https://github.com/lorensr"
  }
}
```

Your server then parses this JSON to determine what happened. In this example, the `sender` is the user who starred the repository, and the `repository` object shows the updated watcher count.

## Typical Webhook Workflow

| Step | Description |
|------|-------------|
| 1    | You register your webhook URL with the service (e.g., GitHub, Stripe). |
| 2    | The service sends an HTTP POST request to your URL when the event occurs. |
| 3    | Your server receives the request and processes the payload. |
| 4    | Optionally, your server responds with a status code (usually 200 OK). |

## Example: Handling a GitHub Webhook in Node.js

Here's a simple example of how you might handle a webhook in Node.js using Express:

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/github-hook', (req, res) => {
  const event = req.body;
  console.log('Received event:', event);

  if (event.action === 'started' && event.repository) {
    console.log(`${event.sender.login} starred ${event.repository.full_name}`);
  }

  res.status(200).send('OK');
});

app.listen(3000, () => {
  console.log('Webhook listener running on port 3000');
});
```

## Testing Your Webhooks

Before deploying your webhook endpoint, it's a good idea to test it with mock requests. This helps you verify that your server handles incoming payloads correctly.

[Beeceptor](https://beeceptor.com/) is a handy tool that lets you quickly create custom endpoints to inspect incoming webhook requests. You simply set up a free endpoint (for example, `https://your-endpoint.beeceptor.com`), point your webhook to this URL, and then view and debug the payloads directly in Beeceptor’s dashboard. This makes it easy to prototype and troubleshoot your webhooks without deploying your own server. Another useful option is [HTTPBin](https://httpbin.org/), which allows you to inspect the structure of HTTP requests by sending them to endpoints like `https://httpbin.org/post`. Both tools help you verify and debug your webhook integrations before going live.

## Best Practices

- **Validate Payloads:** Always verify the authenticity of incoming requests (e.g., using a secret or signature).
- **Respond Quickly:** Return a 2xx status code as soon as possible. Process heavy tasks asynchronously.
- **Log Events:** Keep logs for debugging and auditing webhook activity.
- **Handle Retries:** Some services retry failed webhook deliveries. Make your endpoint idempotent.

