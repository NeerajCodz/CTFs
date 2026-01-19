# Task 3 - Remote Server Challenge Flow

## Objective
Recover Flag 3 by interacting with the exposed backend API.

## Endpoint Model
From binary string inspection:
- `GET /api/status` (health/info path)
- `GET /api/challenge` (challenge payload)
- `POST /api/submit` (answer validation)

## Fetch Challenge
```bash
curl -s https://ccs26-appdev-server.vercel.app/api/challenge
```

Response:

```json
{
  "message": "Decode the clue to find the answer.",
  "clue": "aGl0bGVyX2xpa2VkX2hlbGxvX2tpdHR5",
  "hint": "Base64"
}
```

## Decode Clue
```bash
echo aGl0bGVyX2xpa2VkX2hlbGxvX2tpdHR5 | base64 -d
```

Decoded value:

```text
hitler_liked_hello_kitty
```

## Submit Answer
```bash
curl -s -X POST https://ccs26-appdev-server.vercel.app/api/submit \
  -H "Content-Type: application/json" \
  -d '{"answer":"hitler_liked_hello_kitty"}'
```

Server response:

```json
{
  "success": true,
  "message": "Correct!",
  "flag": "CTF{touch_grass}"
}
```

## Result
```text
CTF{touch_grass}
```

## Key Insight
Clean API mapping plus deterministic decode/submit flow is often the intended solve path for web-backed CTF binaries.
