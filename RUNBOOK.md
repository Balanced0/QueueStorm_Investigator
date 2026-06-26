# QueueStorm Investigator Runbook

## Start Service

```bash
npm install
cp .env.example .env
npm start
```

Set `XAI_API_KEY` before calling `POST /analyze-ticket`.

## Health Check

```bash
curl http://localhost:8000/health
```

Expected response:

```json
{"status":"ok"}
```

## Analyze A Ticket

```bash
curl -X POST http://localhost:8000/analyze-ticket \
  -H "Content-Type: application/json" \
  -d '{"ticket_id":"TKT-001","complaint":"I sent 5000 taka to a wrong number around 2pm today.","language":"en","channel":"in_app_chat","user_type":"customer","transaction_history":[{"transaction_id":"TXN-9101","timestamp":"2026-04-14T14:08:22Z","type":"transfer","amount":5000,"counterparty":"+8801719876543","status":"completed"}]}'
```

## Error Handling Checks

Malformed JSON should return HTTP 400:

```bash
curl -i -X POST http://localhost:8000/analyze-ticket \
  -H "Content-Type: application/json" \
  -d 'not valid json'
```

Missing `ticket_id` or `complaint` should return HTTP 400. Empty complaint text should return HTTP 422.

## Fallback Behavior

If xAI is unavailable, the API key is missing, the request times out, or Grok returns invalid JSON, the endpoint returns HTTP 200 with a complete manual-review response using:

- `evidence_verdict: "insufficient_data"`
- `case_type: "other"`
- `severity: "low"`
- `department: "customer_support"`
- `human_review_required: true`

## Operational Notes

- Keep `XAI_API_KEY` only in `.env` or the deployment secret manager.
- Do not log customer PINs, OTPs, passwords, full card numbers, or secrets.
- Treat `fraud_risk`, `dispute_resolution`, high severity, critical severity, inconsistent evidence, and ambiguous cases as human-review cases.
