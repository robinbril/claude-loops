---
name: audit-loop
description: Loop-until-dry vind-loop voor onbekende hoeveelheden: bugs, security-issues, dead code, inconsistenties of research-claims. Rondes van parallelle finders met verschillende lenzen, dedup, adversarial verificatie, en stoppen na 2 droge rondes. Gebruik bij "audit dit", "vind alle bugs", "check de hele codebase op X", "/audit-loop", of research waar een keer zoeken de staart mist. Vindt en bewijst; fixt zelf niks (handoff naar /bugfix-loop of /feature-loop).
argument-hint: <wat te vinden: bugs, security, dead code, claims, ...>
---

# Sweeploop

Jij bent de orchestrator van een vind-loop. Doel: $ARGUMENTS

## Principe

Onbekende-hoeveelheid-discovery stopt niet bij een teller maar bij droogte: pas als 2 opeenvolgende rondes niks nieuws opleveren is de put leeg. Elke finding overleeft alleen adversarial verificatie.

## Setup (eenmalig)

- Bepaal 3-4 **lenzen** passend bij het doel. Code-audit: correctness, security, performance, consistency. Grote repo: per-directory of per-subsystem. Research: per bron-type (docs, code, issues, web).
- Maak in de scratchpad `audit-loop-state.md`: doel, lenzen, lege `seen`-lijst (key = file:line of claim-hash), lege confirmed-lijst.

## De ronde (via de Workflow-tool)

Deze skill-invocatie is je opt-in voor Workflow. Per ronde:

1. **Finders parallel**: een agent per lens (`model: "sonnet"`, schema-output met per finding: key, omschrijving, bewijs-locatie, severity). Elke finder krijgt het doel, zijn lens, en de keys uit `seen` met de opdracht die over te slaan.
2. **Dedup in plain code**: filter alles wat al in `seen` zit. Fresh findings gaan `seen` in (ook als ze straks sneuvelen, anders convergeert de loop nooit).
3. **Adversarial verify**: per fresh finding een skeptic-agent (`model: "sonnet"`, verse context) met de opdracht de finding te WEERLEGGEN tegen de echte code/bron. Twijfel = refuted. High-severity findings krijgen een tweede stem (`model: "opus"`); alleen unaniem overleeft. CONFIRMED gaat de confirmed-lijst in met bewijs.
4. Log de ronde: gevonden, fresh, confirmed, droogte-teller.

## Stopcondities (hard)

- 2 opeenvolgende rondes met 0 fresh findings: droog, klaar.
- Zonder budget-directief: max 3 rondes. Met een "+Nk"-budget: `while (budget.remaining() > 50_000)`.
- Meld altijd wat NIET gedekt is (lens niet gedraaid, directory geskipt); stille truncatie is verboden.

## Eindrapport

Gerankte confirmed-lijst (severity eerst) met per item: wat, waar (`file:line`), bewijs, en de aanbevolen vervolg-loop (/bugfix-loop voor bugs, /feature-loop voor structureel werk). Gesneuvelde plausibele-maar-weerlegde findings in een aparte korte sectie, zodat niemand ze opnieuw aandraagt.
