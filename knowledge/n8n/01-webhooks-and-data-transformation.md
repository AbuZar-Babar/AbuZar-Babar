# n8n — Webhooks & Data Transformation

## Learning Progress

### Triggers learned
- **Manual Trigger:** starts a workflow when manually executed; useful for testing.
- **Schedule Trigger:** starts a workflow according to a schedule (for example, daily at a specific time).
- **Webhook Trigger:** starts a workflow when another system sends an HTTP request to an n8n webhook URL.

## Webhook Fundamentals

A webhook is an HTTP endpoint exposed by n8n. An external application sends an HTTP request to the webhook, and n8n starts the workflow.

Typical flow:

```text
External Application
       |
       | HTTP request (often POST + JSON)
       v
    Webhook
       |
       v
  n8n workflow
```

### HTTP methods
- **GET:** commonly used to retrieve/request data.
- **POST:** commonly used to send data to a workflow.

### Test vs production webhook
- **Test URL:** used while developing/testing a workflow; n8n must be listening for the test event.
- **Production URL:** used when the workflow is active and intended to receive real requests.

### Docker note
When n8n is running locally in Docker, `localhost:5678` is normally accessible from the local machine, but an external internet service cannot normally reach a localhost webhook directly. Public exposure/tunneling is a later deployment topic.

## Respond to Webhook

The **Respond to Webhook** node controls the HTTP response returned to the system that called the webhook.

Basic flow:

```text
Client
  | POST JSON
  v
Webhook
  v
Process data
  v
Respond to Webhook
  | JSON response
  v
Client
```

For a webhook using the Respond to Webhook node, configure the Webhook response behavior to use the **Respond to Webhook** node.

Example response:

```json
{
  "success": true,
  "message": "Webhook received successfully"
}
```

## n8n Expressions and `$json`

`$json` represents the JSON data currently available to the node.

The important rule learned:

> Always inspect the previous node's output before writing an expression, because the data structure changes as it moves through the workflow.

For example, a Webhook may initially provide request data under `body`:

```text
$json.body.name
```

After an Edit Fields node transforms the data, the next node may receive:

```text
$json.name
```

## Edit Fields

The **Edit Fields** node is used to add, modify, rename, select, and shape data before passing it to another node.

### Include Other Input Fields

In the current n8n UI, the setting is **Include Other Input Fields**.

- **ON:** pass existing input fields through along with fields set in Edit Fields.
- **OFF:** output only the fields explicitly defined in Edit Fields.

### Adding a static field

Example:

```text
source = website
```

Input:

```json
{
  "name": "AbuZar",
  "email": "abuzar@example.com",
  "message": "I want to learn n8n"
}
```

With Include Other Input Fields enabled, output becomes:

```json
{
  "name": "AbuZar",
  "email": "abuzar@example.com",
  "message": "I want to learn n8n",
  "source": "website"
}
```

### Adding a dynamic field

A field can use an expression instead of a fixed value:

```text
customer_type = {{$json.name}}
```

The value changes according to the input data.

### Data normalization / renaming

A useful pattern is to turn:

```json
{
  "name": "AbuZar",
  "email": "abuzar@example.com",
  "message": "I want to learn n8n"
}
```

into a clean downstream schema:

```json
{
  "full_name": "AbuZar",
  "email_address": "abuzar@example.com",
  "message": "I want to learn n8n",
  "source": "website",
  "status": "new"
}
```

This is useful when preparing data for APIs, CRMs, databases, or other systems that expect a different schema.

## Completed Practice Workflow

```text
Postman
   |
   | POST JSON
   v
Webhook
   |
   v
Edit Fields
   |
   v
Respond to Webhook
   |
   v
Postman
```

Example Postman input:

```json
{
  "name": "AbuZar",
  "email": "abuzar@example.com",
  "message": "I want to learn n8n"
}
```

The workflow practiced:
1. Receiving JSON through a Webhook.
2. Adding fields with Edit Fields.
3. Using expressions to access values.
4. Transforming/renaming fields.
5. Returning a custom JSON response with Respond to Webhook.

Example transformed response:

```json
{
  "success": true,
  "customer": {
    "full_name": "AbuZar",
    "email_address": "abuzar@example.com",
    "message": "I want to learn n8n"
  },
  "metadata": {
    "source": "website",
    "status": "new"
  }
}
```

## Core Mental Model

```text
RECEIVE → TRANSFORM → SEND

Webhook → Edit Fields → Respond to Webhook
```

This pattern is a foundation for later n8n work with APIs, CRMs, databases, forms, AI workflows, and business automation.

## Next Topic

**IF node / conditional logic** — route data down different workflow branches based on conditions such as `priority == "urgent"`.
