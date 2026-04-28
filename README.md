# Dullas Secure App

A minimal, security-hardened Flask web service containerised with a distroless Docker image and served by Gunicorn.

---

## Features

- **Health check endpoint** – `GET /healthz` returns `{"status": "ok"}`.
- **Echo endpoint** – `POST /v1/echo` accepts a JSON body and echoes back the trimmed message.
- **Security headers** – every response includes `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, and `Content-Security-Policy`.
- **Request size cap** – incoming requests are limited to 32 KB.
- **Distroless container** – the production image is based on `gcr.io/distroless/python3-debian12:nonroot`, minimising the attack surface.

---

## Project Structure

```
.
├── app/
│   ├── __init__.py
│   └── main.py          # Flask application factory & route definitions
├── tests/
│   └── test_main.py     # pytest unit tests
├── Dockerfile            # Multi-stage build (builder → distroless)
└── requirements.txt      # Python dependencies (Flask, Gunicorn)
```

---

## API Reference

### `GET /healthz`

Returns a liveness check response.

**Response `200 OK`**
```json
{"status": "ok"}
```

---

### `POST /v1/echo`

Echoes back the trimmed `message` field from the request body.

**Request**

| Header         | Value              |
|----------------|--------------------|
| `Content-Type` | `application/json` |

```json
{"message": "hello"}
```

**Successful Response `200 OK`**
```json
{"message": "hello"}
```

**Error Responses**

| Status | `error` value                           | Cause                                        |
|--------|-----------------------------------------|----------------------------------------------|
| 415    | `content_type_must_be_application_json` | `Content-Type` is not `application/json`     |
| 400    | `invalid_json_object`                   | Body is not a JSON object                    |
| 400    | `message_must_be_string`                | `message` field is missing or not a string   |
| 400    | `message_length_out_of_range`           | Trimmed message is empty or longer than 200 chars |

---

## Security Headers

All responses include the following headers:

| Header                    | Value            |
|---------------------------|------------------|
| `X-Content-Type-Options`  | `nosniff`        |
| `X-Frame-Options`         | `DENY`           |
| `Referrer-Policy`         | `no-referrer`    |
| `Content-Security-Policy` | `default-src 'none'` |

---

## Getting Started

### Prerequisites

- Python 3.12+
- Docker (for container builds)

### Run Locally (Development)

```bash
# Install dependencies
pip install -r requirements.txt

# Start Gunicorn
python -m gunicorn -w 2 -b 127.0.0.1:8080 app.main:app
```

The service will be available at `http://127.0.0.1:8080`.

### Run with Docker

```bash
# Build the image
docker build -t secure-app:local .

# Run the container
docker run --rm -p 8080:8080 secure-app:local
```

---

## Running Tests

```bash
pip install pytest
pytest tests/
```

All 4 unit tests should pass in under a second:

```
....                                                                     [100%]
4 passed in 0.19s
```

---

## Dependencies

| Package   | Version |
|-----------|---------|
| Flask     | 3.1.3   |
| Gunicorn  | 22.0.0  |
