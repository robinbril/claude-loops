---
name: waak-loop
description: Babysit-loop voor externe state die de harness niet zelf trackt: een CI-run, deploy, DNS-propagatie, URL-health, mailbox of kanban-kolom. Pollt cache-bewust via ScheduleWakeup, voert bij verandering een vooraf afgesproken actie uit en stopt bij het eindcriterium. Gebruik bij "hou de deploy in de gaten", "wacht tot de CI groen is en ga dan door", "check elk kwartier of X", "/waak-loop", of elke wacht-op-extern situatie. Verbrandt per wake precies een check, geen context.
argument-hint: <wat bewaken + wat te doen bij verandering>
---

# Watchloop

Jij bewaakt externe state en handelt bij verandering. Opdracht: $ARGUMENTS

## Stap 0: contract (eenmalig, voor de eerste slaap)

Leg vier dingen vast in `waak-loop-state.md` (scratchpad):

1. **De check**: een zo goedkoop mogelijke enkele call (gh run view, curl -s, een MCP-tool, een grep op een logfile). Geen browser, geen screenshots als een API-call bestaat.
2. **De trigger**: welke waarde-verandering telt (status != in_progress, HTTP 200, record zichtbaar).
3. **De actie**: wat er bij de trigger gebeurt. Omkeerbaar en vooraf door de gebruiker benoemd: direct uitvoeren. Onomkeerbaar (deploy, send, delete) en niet expliciet vooraf geautoriseerd: rapporteren en wachten.
4. **Het einde**: wanneer de loop klaar is (trigger afgehandeld, deadline, of max aantal wakes, default 20).

Ontbreekt een van de vier in de opdracht, vul het minst riskante in en meld je aanname in een regel.

## De wake-cyclus

Per wake:

1. Voer de check uit (een call).
2. Geen verandering: log een regel in de state-file (timestamp, waarde) en plan de volgende wake.
3. Trigger: voer de actie uit, verifieer het resultaat met een echte tool-call, rapporteer antwoord-eerst, en stop (of loop door als de opdracht doorlopend is).
4. Check faalt zelf (tool-error): een retry direct; faalt die ook, meld het en plan een langere wake in plaats van blind doorpollen.

## Pacing (cache-bewust, denk in cache-windows)

Kies de ScheduleWakeup-delay op wat je bewaakt, nooit op een rond getal:

- Snel-veranderend (CI-run van ~10 min, deploy): **270s**, blijft binnen de prompt-cache-TTL.
- Traag (DNS, mailbox, dagelijkse job): **1200-1800s**.
- Nooit 300s (cache-miss zonder de wachttijd te benutten). Geef in de `prompt`-parameter dezelfde /waak-loop-opdracht verbatim terug en zet in `reason` concreet wat je bewaakt.

## Guardrails

- Een check per wake; geen "nu ik toch wakker ben"-extra werk.
- De state-file is het geheugen; de conversatie niet.
- Max-wakes bereikt zonder trigger: stop met een eerlijke laatste stand, niet stilletjes doorslapen.
