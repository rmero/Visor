# Visor — Official VRP Viewer for macOS

**De eerste officiële viewer voor het Visual Result Protocol (VRP)**

Visor is een native macOS-applicatie die AI-agents hun resultaten **direct visueel** laat tonen, in plaats van alleen tekst, logs of codeblokken.

---

## Wat is VRP?

**VRP (Visual Result Protocol)** is een open protocol, speciaal ontworpen als **complementair** aan **MCP** (Model Context Protocol).

- **MCP** → Geeft agents toegang tot tools en data  
- **VRP** → Zorgt voor **rijke, visuele resultaten** (live previews, afbeeldingen, bestandsveranderingen, apps, etc.)

Samen maken ze agentic workflows veel krachtiger en gebruiksvriendelijker.

## Waarom Visor?

De meeste AI-tools geven alleen tekstoutput. Visor lost dit op door:
- Directe, mooie visuele previews te tonen
- Snelle iteratie mogelijk te maken ("maak dit mooier" → zie direct het resultaat)
- Native macOS-integraties te gebruiken (Simulator, WebView, file views, etc.)
- Een productieve en rustige interface voor developers en power users

## Belangrijkste Features

- Volledige ondersteuning voor VRP 0.2.1
- Dynamische result viewers die automatisch het juiste formaat tonen
- Ondersteuning voor meerdere actieve agent-sessies tegelijk
- Actie-knoppen (Approve, Regenerate, Open in Simulator, Download, etc.)
- Geschiedenis van alle resultaten
- Menu Bar Extra voor snelle toegang
- Command Palette (⌘K)
- Donker modus en Stage Manager geoptimaliseerd

### Ondersteunde Result Types
- `app_preview` — Live web en Simulator previews
- `image_gallery` — Afbeeldingen met varianten en vergelijking
- `file_system_delta` — Interactieve bestands- en mapweergave
- `interactive_artifact` — Interactieve web en component previews
- `document_preview` — Schone documentweergave
- `diff_visual` — Code diffs met syntax highlighting
- `composite` — Meerdere resultaten naast elkaar

## Snelle Start

1. Download Visor (binnenkort via GitHub Releases)
2. Start je agent of MCP-server met VRP-ondersteuning
3. Open Visor → de app detecteert automatisch VRP-sessies
4. Kijk direct naar de visuele resultaten

## Technologie

- Swift 6 + SwiftUI
- WebSocket (URLSessionWebSocketTask)
- WebKit voor live previews
- SwiftData voor lokale opslag
- Volledig async/await

## Roadmap

- **v0.1** — MVP met basis viewers
- **v0.2** — Volledige MCP integratie + acties
- **v0.3** — Native Simulator en SwiftUI previews
- **v0.4** — Multi-agent canvas en sessie replay
- **v1.0** — Public VRP standaard + iOS companion app

## VRP Protocol

De volledige specificatie vind je in [`VRP-SPEC.md`](VRP-SPEC.md).

## Bijdragen

Dit project is open source. Bijdragen zijn zeer welkom:

- Nieuwe result type viewers
- Protocol verbeteringen
- Voorbeelden in Python, TypeScript of andere talen
- Bug reports en feature requests

## Licentie

MIT License

---

**Gemaakt voor het agentic AI tijdperk**

Wil je meewerken? Open gerust een Issue of Pull Request.
