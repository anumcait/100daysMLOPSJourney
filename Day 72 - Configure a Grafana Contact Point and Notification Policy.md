# Day 72: Configure a Grafana Contact Point and Notification Policy
The xFusionCorp Industries ML platform team requires that high-severity model alerts trigger notifications to the on-call channel via webhook. Currently, the existing alert rules are effective only if an individual is paged when alerts are activated. The monitoring stack is operational, and an in-stack webhook-sink service (container webhook-sink) is available at http://webhook-sink:5000/hook. Additionally, Grafana has the Prometheus datasource already configured. Your objective is to set up Grafana alerting so that any alert with the label severity=high is directed to the specified webhook endpoint.


The Grafana UI is running on port 3000. The Grafana button opens the login page. Admin credentials: admin / grafana2026. The webhook sink is reachable from Grafana at http://webhook-sink:5000/hook.

The end state must include:

GET /api/v1/provisioning/contact-points returns at least one contact point of type webhook whose settings.url references webhook-sink.
GET /api/v1/provisioning/policies returns a notification-policy tree containing a route whose receiver matches that contact point and whose matchers include severity = high.
Contact points answer the question 'where does a notification go?'—an endpoint (webhook, email, Slack). Notification policies answer 'which alerts go to which contact point?'—by label-matching the alert. Both pieces must be in place before any alert rule actually pages a human.

## Objective

Configure Grafana alerting so that any alert with the label `severity=high` sends notifications to the in-stack webhook endpoint.

**Webhook URL**

```text
http://webhook-sink:5000/hook
```

**Grafana**

- URL: `http://localhost:3000`
- Username: `admin`
- Password: `grafana2026`

---

## Steps

### 1. Log in to Grafana

Open Grafana and log in using:

- Username: `admin`
- Password: `grafana2026`

---

### 2. Create a Contact Point

Navigate to:

```
Alerting → Notification configuration → Contact points
```

Create a new contact point with the following configuration:

| Field | Value |
|-------|-------|
| Name | `high-severity-webhook` |
| Type | `Webhook` |
| URL | `http://webhook-sink:5000/hook` |

Save the contact point.

---

### 3. Configure Notification Policy

Navigate to:

```
Alerting → Notification configuration → Notification policies
```

Add a new route under the **Default policy**.

Configure:

| Field | Value |
|-------|-------|
| Label | `severity` |
| Operator | `=` |
| Value | `high` |
| Contact Point | `high-severity-webhook` |

Leave **Continue matching subsequent sibling nodes** unchecked.

Click **Add route**, then **Save policy**.

---

## Verification

### Verify Contact Point

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/v1/provisioning/contact-points
```

Expected output:

```json
[
  {
    "uid": "dfubvtqeq9n9ca",
    "name": "high-severity-webhook",
    "type": "webhook",
    "settings": {
      "url": "http://webhook-sink:5000/hook"
    },
    "disableResolveMessage": false
  }
]
```

---

### Verify Notification Policy

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/v1/provisioning/policies
```

Expected output:

```json
{
  "receiver": "empty",
  "group_by": [
    "grafana_folder",
    "alertname"
  ],
  "routes": [
    {
      "receiver": "high-severity-webhook",
      "object_matchers": [
        [
          "severity",
          "=",
          "high"
        ]
      ]
    }
  ]
}
```

---

## Validation Checklist

- ✅ Webhook contact point created
- ✅ Webhook URL set to `http://webhook-sink:5000/hook`
- ✅ Notification policy route matches `severity=high`
- ✅ Route delivers to `high-severity-webhook`
- ✅ Configuration verified using Grafana Provisioning APIs

---

## Result

Successfully configured Grafana Alerting to route all alerts labeled `severity=high` to the webhook endpoint via a dedicated webhook contact point and notification policy.

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1026027b-d2c0-4e36-ba29-fac81bbba81c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/ab64806d-a045-43d9-833b-f964534fa930" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6c3f9137-93c2-4a74-a90a-28eacdc74760" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/76fd5322-5beb-410e-a407-51c2097ba6fe" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/24d050f7-7591-4101-8ec0-eb336aeb7586" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/def9cae3-792c-43ca-9998-55522c983ff0" />






