# Visual Result Protocol (VRP)

**Versie:** 0.2.1  
**Datum:** 7 mei 2026  
**Status:** Draft  
**Doel:** Complementair protocol naast MCP (Model Context Protocol)

---

## 1. Inleiding

**VRP (Visual Result Protocol)** is een open protocol dat zich richt op het **snelle, rijke en visuele presenteren van resultaten** die AI-agents genereren.

Terwijl **MCP** zich bezighoudt met tools, resources en acties (wat de agent *mag doen*), richt **VRP** zich puur op **wat de agent heeft opgeleverd** en hoe dit zo duidelijk en visueel mogelijk aan de gebruiker getoond kan worden.

### Kernprobleem dat VRP oplost

Huidige AI-agents geven voornamelijk tekst, logs en codeblokken. VRP zorgt ervoor dat gebruikers **direct het echte resultaat** zien (live app, gegenereerde afbeeldingen, opgeruimde bestanden, etc.).

---

## 2. Doelstellingen

- Snelle visuele feedback in plaats van alleen tekst
- Directe, rijke previews van concrete resultaten
- Sterke native ondersteuning (vooral macOS, later iOS, tvOS, Raspberry Pi)
- Laag bandbreedtegebruik
- Volledig complementair aan MCP
- Extensible en eenvoudig te implementeren

---

## 3. Kernprincipes

1. **Result-First** — Focus op het eindresultaat, niet op interne stappen.
2. **Streaming-first** — Resultaten kunnen incrementeel gestuurd worden.
3. **Rich & Native** — Ondersteunt hoogwaardige platform-specifieke weergaves.
4. **User-Centric** — Zo min mogelijk ruis, maximaal bruikbaar resultaat.
5. **Extensible** — Makkelijk nieuwe result types toevoegen.
6. **Secure by default** — Minimale permissies, geen ongewenste code-executie.

---

## 4. Relatie met MCP

| Aspect | MCP | VRP |
|--------|-----|-----|
| Doel | Control & Tooling | Visualization & Results |
| Focus | Wat de agent mag doen | Wat de agent heeft opgeleverd |
| Richting | Client → Agent | Agent → Client |
| Inhoud | Tool calls, resources | Visuele payloads, previews |

Beide protocollen kunnen parallel over dezelfde WebSocket-verbinding werken. VRP berichten zijn onderscheidbaar via het `vrp_version` veld.

---

## 5. Transport

- **Primair**: WebSocket (`ws://` / `wss://`)
- **Alternatieven**: Server-Sent Events (SSE), HTTP streaming
- **Poort (standaard)**: `7830`
- **Discovery**: Via MCP handshake of mDNS (`_vrp._tcp.local`) op het lokale netwerk
- **Encoding**: UTF-8 JSON

---

## 6. Berichtstructuur

Elk VRP-bericht is een JSON-object met de volgende toplevelvelden:

```json
{
  "vrp_version": "0.2.1",
  "message_id": "msg_abc123456",
  "session_id": "sess_xyz789012",
  "agent_id": "agent-coding-42",
  "timestamp": "2026-05-07T12:34:56.789Z",

  "type": "result",
  "result_type": "app_preview",
  "status": "success",
  "summary": "Korte duidelijke beschrijving voor de gebruiker",

  "visual_payload": { },
  "actions": [],
  "metadata": { }
}
```

### 6.1 Toplevelvelden

| Veld | Type | Verplicht | Beschrijving |
|------|------|-----------|--------------|
| `vrp_version` | string | ja | Protocol versie, bijv. `"0.2.1"` |
| `message_id` | string | ja | Unieke berichtidentifier (`msg_` prefix) |
| `session_id` | string | ja | Sessie-identifier (`sess_` prefix) |
| `agent_id` | string | nee | Identifier van de genererende agent |
| `timestamp` | string | ja | ISO 8601 tijdstempel |
| `type` | string | ja | Berichttype: `result`, `partial`, `error`, `ping`, `ack` |
| `result_type` | string | conditioneel | Verplicht bij `type: result` of `type: partial` |
| `status` | string | ja | `success`, `partial`, `error` |
| `summary` | string | ja | Korte leesbare samenvatting voor de gebruiker |
| `visual_payload` | object | conditioneel | Verplicht bij `type: result` of `type: partial` |
| `actions` | array | nee | Beschikbare acties voor de gebruiker |
| `metadata` | object | nee | Extra vrije metadata |

### 6.2 Berichttypen

| Type | Beschrijving |
|------|-------------|
| `result` | Volledig resultaat |
| `partial` | Tussentijds streaming resultaat |
| `error` | Foutmelding |
| `ping` | Keepalive |
| `ack` | Bevestiging van ontvangst |

---

## 7. Result Types

### 7.1 `app_preview`

Live preview van een webapplicatie of iOS/macOS Simulator.

```json
{
  "result_type": "app_preview",
  "visual_payload": {
    "preview_type": "web | simulator | swiftui",
    "url": "http://localhost:3000",
    "bundle_id": "com.example.MyApp",
    "platform": "ios | macos | web",
    "screenshot_base64": "<base64>",
    "interactive": true
  }
}
```

| Veld | Type | Beschrijving |
|------|------|-------------|
| `preview_type` | string | `web`, `simulator`, of `swiftui` |
| `url` | string | URL voor web previews |
| `bundle_id` | string | Bundle ID voor Simulator previews |
| `platform` | string | Doelplatform |
| `screenshot_base64` | string | Optionele statische fallback |
| `interactive` | bool | Of de preview interactief is |

---

### 7.2 `image_gallery`

Een of meerdere gegenereerde afbeeldingen, eventueel met varianten.

```json
{
  "result_type": "image_gallery",
  "visual_payload": {
    "images": [
      {
        "id": "img_001",
        "label": "Variant A",
        "url": "https://...",
        "base64": "<base64>",
        "mime_type": "image/png",
        "width": 1024,
        "height": 768,
        "prompt_used": "Een rustig berglandschap"
      }
    ],
    "layout": "grid | carousel | comparison"
  }
}
```

---

### 7.3 `file_system_delta`

Interactieve weergave van bestandswijzigingen (toegevoegd, verwijderd, aangepast).

```json
{
  "result_type": "file_system_delta",
  "visual_payload": {
    "base_path": "/Users/robin/project",
    "changes": [
      {
        "path": "src/App.swift",
        "change_type": "modified | added | deleted | renamed",
        "old_path": null,
        "size_bytes": 4096,
        "preview": "import SwiftUI\n..."
      }
    ],
    "summary": {
      "added": 3,
      "modified": 5,
      "deleted": 1
    }
  }
}
```

---

### 7.4 `interactive_artifact`

Een interactief HTML/JS/CSS component of webpagina die direct in de viewer getoond wordt.

```json
{
  "result_type": "interactive_artifact",
  "visual_payload": {
    "artifact_type": "html | react | vue | svelte",
    "html": "<html>...</html>",
    "css": "body { ... }",
    "js": "console.log('hello')",
    "sandbox": true,
    "allow_scripts": false
  }
}
```

> **Security**: `sandbox: true` is verplicht tenzij expliciet uitgeschakeld door de gebruiker. `allow_scripts` staat standaard op `false`.

---

### 7.5 `document_preview`

Schone weergave van een document (Markdown, PDF, tekst).

```json
{
  "result_type": "document_preview",
  "visual_payload": {
    "format": "markdown | pdf | plaintext | html",
    "content": "# Titel\n\nInhoud...",
    "url": "https://...",
    "title": "Documenttitel",
    "word_count": 512
  }
}
```

---

### 7.6 `diff_visual`

Code diff met syntax highlighting.

```json
{
  "result_type": "diff_visual",
  "visual_payload": {
    "diffs": [
      {
        "file_path": "src/main.swift",
        "language": "swift",
        "old_content": "let x = 1",
        "new_content": "let x = 42",
        "hunks": [
          {
            "old_start": 10,
            "new_start": 10,
            "lines": [
              { "type": "context", "content": "func foo() {" },
              { "type": "removed", "content": "  let x = 1" },
              { "type": "added", "content": "  let x = 42" },
              { "type": "context", "content": "}" }
            ]
          }
        ]
      }
    ]
  }
}
```

---

### 7.7 `composite`

Meerdere result types gecombineerd in één bericht, naast of onder elkaar getoond.

```json
{
  "result_type": "composite",
  "visual_payload": {
    "layout": "horizontal | vertical | tabs",
    "items": [
      {
        "result_type": "diff_visual",
        "label": "Wijzigingen",
        "visual_payload": { }
      },
      {
        "result_type": "app_preview",
        "label": "Live Preview",
        "visual_payload": { }
      }
    ]
  }
}
```

---

## 8. Acties

Acties zijn optionele knoppen die de viewer toont bij een resultaat.

```json
{
  "actions": [
    {
      "id": "action_approve",
      "label": "Approve",
      "style": "primary | secondary | destructive",
      "icon": "checkmark.circle",
      "mcp_tool": "apply_changes",
      "mcp_params": { "confirm": true }
    },
    {
      "id": "action_regenerate",
      "label": "Regenerate",
      "style": "secondary",
      "icon": "arrow.clockwise"
    },
    {
      "id": "action_open_simulator",
      "label": "Open in Simulator",
      "style": "secondary",
      "icon": "iphone"
    },
    {
      "id": "action_download",
      "label": "Download",
      "style": "secondary",
      "icon": "arrow.down.circle"
    }
  ]
}
```

### Actievelden

| Veld | Type | Verplicht | Beschrijving |
|------|------|-----------|-------------|
| `id` | string | ja | Unieke actie-ID |
| `label` | label | ja | Weergavenaam |
| `style` | string | nee | `primary`, `secondary`, `destructive` |
| `icon` | string | nee | SF Symbol naam |
| `mcp_tool` | string | nee | MCP tool om aan te roepen bij klik |
| `mcp_params` | object | nee | Parameters voor de MCP tool |

---

## 9. Streaming (Partial Results)

Voor grote of langlopende resultaten kan een agent tussentijdse updates sturen.

```json
{
  "type": "partial",
  "result_type": "image_gallery",
  "status": "partial",
  "stream_index": 2,
  "stream_total": 5,
  "is_final": false,
  "visual_payload": {
    "images": [ { "id": "img_001", "base64": "..." } ]
  }
}
```

| Veld | Type | Beschrijving |
|------|------|-------------|
| `stream_index` | int | Volgnummer van dit deel (0-based) |
| `stream_total` | int | Totaal verwacht aantal delen (`-1` = onbekend) |
| `is_final` | bool | `true` bij het laatste deel |

---

## 10. Foutafhandeling

```json
{
  "type": "error",
  "status": "error",
  "error": {
    "code": "PAYLOAD_TOO_LARGE",
    "message": "De payload overschrijdt de maximale grootte van 10MB.",
    "result_type": "image_gallery",
    "recoverable": true
  }
}
```

### Foutcodes

| Code | Beschrijving |
|------|-------------|
| `UNSUPPORTED_RESULT_TYPE` | Onbekend of niet-ondersteund result type |
| `PAYLOAD_TOO_LARGE` | Payload groter dan 10MB |
| `INVALID_FORMAT` | JSON is niet geldig of velden ontbreken |
| `SESSION_NOT_FOUND` | Sessie-ID bestaat niet |
| `PERMISSION_DENIED` | Actie niet toegestaan |
| `INTERNAL_ERROR` | Interne serverfout |

---

## 11. Sessiebeheer

### Handshake

Bij verbinding stuurt de client een `hello` bericht:

```json
{
  "vrp_version": "0.2.1",
  "type": "hello",
  "client_id": "visor-macos-1.0",
  "capabilities": ["app_preview", "image_gallery", "diff_visual", "composite"],
  "platform": "macos"
}
```

De server bevestigt:

```json
{
  "vrp_version": "0.2.1",
  "type": "hello_ack",
  "session_id": "sess_xyz789012",
  "server_capabilities": ["app_preview", "image_gallery", "file_system_delta", "diff_visual"]
}
```

### Sessie afsluiten

```json
{
  "type": "goodbye",
  "session_id": "sess_xyz789012",
  "reason": "user_closed | agent_done | timeout"
}
```

---

## 12. Beveiliging

- Alle verbindingen over publiek netwerk: verplicht **TLS** (`wss://`)
- Lokale verbindingen (`localhost`): `ws://` toegestaan
- Sandbox voor `interactive_artifact` is **verplicht** tenzij expliciet uitgeschakeld
- `allow_scripts: false` is de standaard
- Agents mogen geen arbitraire commando's injecteren via `mcp_tool`
- Maximale payloadgrootte: **10MB** per bericht
- Rate limiting aanbevolen: max 100 berichten/seconde per sessie

---

## 13. Metadata

Het `metadata` veld is vrij en kan aanvullende informatie bevatten:

```json
{
  "metadata": {
    "agent_model": "claude-sonnet-4-6",
    "generation_time_ms": 1420,
    "tokens_used": 3812,
    "tool_calls": ["read_file", "write_file"],
    "tags": ["refactor", "swift"],
    "context_id": "ctx_abc123"
  }
}
```

---

## 14. Voorbeeldimplementaties

### Python (agent-side)

```python
import asyncio
import json
import websockets
from datetime import datetime, timezone
import uuid

async def send_vrp_result(ws, result_type: str, payload: dict, summary: str):
    message = {
        "vrp_version": "0.2.1",
        "message_id": f"msg_{uuid.uuid4().hex[:12]}",
        "session_id": "sess_demo001",
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "type": "result",
        "result_type": result_type,
        "status": "success",
        "summary": summary,
        "visual_payload": payload,
        "actions": [],
        "metadata": {}
    }
    await ws.send(json.dumps(message))

async def main():
    async with websockets.connect("ws://localhost:7830") as ws:
        await send_vrp_result(
            ws,
            result_type="document_preview",
            payload={"format": "markdown", "content": "# Hello VRP!", "title": "Test"},
            summary="Document gegenereerd"
        )

asyncio.run(main())
```

### TypeScript (agent-side)

```typescript
import WebSocket from "ws";

interface VRPMessage {
  vrp_version: string;
  message_id: string;
  session_id: string;
  timestamp: string;
  type: "result" | "partial" | "error" | "ping" | "ack";
  result_type?: string;
  status: "success" | "partial" | "error";
  summary: string;
  visual_payload?: Record<string, unknown>;
  actions?: VRPAction[];
  metadata?: Record<string, unknown>;
}

interface VRPAction {
  id: string;
  label: string;
  style?: "primary" | "secondary" | "destructive";
  icon?: string;
  mcp_tool?: string;
  mcp_params?: Record<string, unknown>;
}

function sendResult(ws: WebSocket, message: Omit<VRPMessage, "vrp_version" | "timestamp">) {
  ws.send(JSON.stringify({
    vrp_version: "0.2.1",
    timestamp: new Date().toISOString(),
    ...message,
  }));
}

const ws = new WebSocket("ws://localhost:7830");

ws.on("open", () => {
  sendResult(ws, {
    message_id: "msg_ts_001",
    session_id: "sess_demo001",
    type: "result",
    result_type: "diff_visual",
    status: "success",
    summary: "Refactor voltooid — 3 bestanden gewijzigd",
    visual_payload: {
      diffs: []
    }
  });
});
```

---

## 15. Versiehistorie

| Versie | Datum | Wijzigingen |
|--------|-------|-------------|
| 0.2.1 | 2026-05-07 | Huidige draft — alle 7 result types, streaming, sessiebeheer |
| 0.2.0 | 2026-04-15 | Toevoeging `composite` en `interactive_artifact` |
| 0.1.0 | 2026-03-01 | Initiële draft met basis result types |

---

## 16. Roadmap

- **v0.3** — Bidirectionele interactie (gebruikersacties terugsturen naar agent)
- **v0.3** — Native SwiftUI preview support
- **v0.4** — Multi-agent canvas: meerdere VRP-streams naast elkaar
- **v0.4** — Sessie replay en geschiedenis export
- **v1.0** — Publieke standaard, formele RFC, iOS companion support

---

## Licentie

MIT License — zie [LICENSE](LICENSE)

---

*VRP is ontworpen als open standaard. Bijdragen, verbeteringen en implementaties in andere talen zijn zeer welkom via Issues en Pull Requests.*
