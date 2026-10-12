# Accident Alert System

A full-stack IoT safety dashboard and API for collision telemetry, emergency contacts, GPS location sharing, and alert delivery. The dashboard is currently a demonstration-ready prototype: records are held in server memory and reset when the API restarts. Add persistent storage and real authentication before production use.

## Requirements
- Node.js 20+ and npm

## Run locally
1. Copy `.env.example` to `backend/.env` and adjust values if needed. The API starts in safe mock mode when Twilio credentials are blank.
2. Install dependencies at the root, then install the frontend and backend dependencies: `npm install`, `npm --prefix backend install`, and `npm --prefix frontend install`.
3. Run `npm run dev` at the project root. The API is at `http://localhost:4000` and the dashboard is at `http://localhost:5173`.
4. Build both applications with `npm run build`.

## Included capabilities
- Accident Alert System dashboard with monitoring status, recent alerts, device state, and latest GPS coordinates.
- Profile and emergency contact management API; demo contacts are seeded for convenient local preview.
- Manual test-alert workflow and simulated ESP32/Arduino telemetry endpoint.
- Google Maps links are generated from validated latitude/longitude values.
- Twilio SMS and SMTP email integrations are optional; without credentials, alerts are logged as mock deliveries. Configure either channel (or both) in `backend/.env` to send notifications to the matching emergency-contact details.

## API
All API routes use the `/api` prefix.

- `GET /api/health` — service status.
- `GET /api/dashboard` — profile, device, contacts, and alert summary.
- `PUT /api/profile` — update the demo user profile.
- `GET /api/contacts`, `POST /api/contacts`, `DELETE /api/contacts/:id` — emergency contacts.
- `POST /api/alerts/test` — safely create a test alert (does not claim a real crash occurred).
- `POST /api/device/telemetry` — accept device readings. Send `x-device-api-key` matching `DEVICE_API_KEY`.

Example telemetry JSON:

```json
{
  "deviceId": "esp32-vehicle-01",
  "acceleration": { "x": 0.2, "y": 0.1, "z": 8.9, "magnitude": 9.0 },
  "gps": { "latitude": 37.7749, "longitude": -122.4194, "accuracy": 8 },
  "occurredAt": "2026-10-12T12:00:00.000Z"
}
```

A collision alert is generated only when `magnitude` meets the configurable `ACCIDENT_ALERT_SYSTEM_COLLISION_THRESHOLD` (default 7.5 g). Thresholds must be calibrated and validated against the actual vehicle, sensor mounting, and false-positive profile. Never rely on this prototype as the sole emergency response system.

## Hardware notes
Connect an accelerometer (for example MPU-6050) and GPS module to an ESP32/Arduino-compatible board. Have the device send authenticated JSON telemetry to `POST /api/device/telemetry` over TLS in production. Do not embed a production API key in publicly distributed firmware. GPS coordinates and personal/emergency-contact details are sensitive data; obtain consent, restrict retention/access, and use HTTPS and a secure database before deployment.

## Production checklist
- Replace in-memory demo data with a persistent database and migrations.
- Implement account registration, password hashing, sessions/JWT, authorization, rate limits, audit logs, and input validation appropriate to deployment.
- Configure and test Twilio/email providers, delivery retries, opt-in and regional compliance.
- Secure device provisioning and rotate `DEVICE_API_KEY`; use per-device credentials and TLS.
- Calibrate collision thresholds and test failure modes with qualified safety engineers.
