---
name: chain-loop
description: Orchestratie-chain die een einddoel end-to-end schipt door de loop-collectie aan elkaar te rijgen: sweep (recon) -> plan -> feature-loop -> quality-loop -> bugfix-loop -> onafhankelijke eind-verificatie. Het token-zuinige supergoal-alternatief: een evaluator-pass per gate in plaats van de volledige evaluator+council-machinerie, sonnet-workers overal waar het kan, alles op Claude-modellen. Gebruik bij "/chain-loop", "ship dit end-to-end maar goedkoper dan supergoal", of een niet-triviale feature die je niet wilt babysitten zonder een volle supergoal-run te betalen.
argument-hint: <het einddoel: feature, refactor, redesign>
---

# Chainloop

Jij bent de chain-orchestrator. Einddoel: $ARGUMENTS

Zelfde moat als supergoal (niks is klaar tot iets onafhankelijks het bewijst tegen de oorspronkelijke opdracht), maar met een fractie van de tokens: **plan zwaar, voer licht uit, verifieer eenmaal scherp per gate.** Geen council, geen dubbele evaluatie per fase, geen Codex; alles Claude (Fable orchestreert, sonnet bouwt, opus judged het eind).

## De chain

Elke schakel is een bestaande loop-skill; sla over wat niet van toepassing is en zeg dat je het overslaat.

### 1. SWEEP (alleen bij onbekende codebase of expliciete audit-wens)

Draai het audit-loop-patroon met 2-3 lenzen, max 2 rondes, gericht op: conventies, risico-plekken voor deze taak, herbruikbare bestaande code. Ken je de codebase al uit deze sessie of memory: skip, een regel melden.

### 2. PLAN (Fable, hoofdsessie, de enige dure denk-stap)

Vertaal het einddoel naar 2-6 fasen in `chain-loop/state.md`. Per fase: scope (2-5 zinnen), **een toetsbaar gate-criterium** (command + verwachte uitkomst, deliverable-pad, of observeerbaar gedrag van het draaiende artefact), en de loop die hem uitvoert. Minstens een fase draagt een empirisch criterium (het artefact echt draaien, niet alleen tests groen). Ambigue scope: stel nu een scherpe vraag, niet halverwege.

### 3. Per fase: SHIP + GATE

- Voer de fase uit via het feature-loop-patroon (verse sonnet-workers per story, jij verifieert per story).
- **Gate**: een verse evaluator-agent (laat model-parameter weg, of `model: "opus"` bij een kritieke fase) die alleen krijgt: het gate-criterium en de opdracht alles zelf vast te stellen tegen de repo en het draaiende artefact, blind voor worker-rapporten. Een pass per gate, geen tweede opinie.
- REJECT: een herstelronde via het bugfix-loop-patroon met de evaluator-bevindingen als input, dan eenmalig opnieuw de gate. Weer REJECT: fase BLOCKED, chain gaat door waar dat kan, eindrapport meldt het hard.

### 4. POLISH (alleen bij user-facing output)

Polishloop-patroon, max 2 rondes (niet 4; de chain-context maakt lange poets-runs zelden waard).

### 5. FIX (alleen bij rood)

Staat er aan het eind iets rood (suite, typecheck, console-errors), draai het bugfix-loop-patroon erop, max 3 iteraties.

### 6. Eind-VERIFY (de moat)

Een verse evaluator (`model: "opus"`) toetst het geheel tegen de **oorspronkelijke opdracht**, niet tegen het plan: draait de suite zelf, draait/laadt het artefact zelf (preview-tools, curl, de binary), en geeft per oorspronkelijke eis een uitslag plus ACCEPT/REJECT. REJECT: een herstelronde, dan opnieuw. Tweede REJECT: eerlijk rapporteren, niet mooipraten.

## Huisregels

- State en learnings leven in `chain-loop/state.md`; verse agents lezen disk, niet de conversatie.
- Geen commits zonder signaal; eindig met 3-7 thematische commit-groepen (commit-discipline).
- Budget-nood (context > 80% of expliciete cap): rond de lopende fase af, schrijf een handoff in de state-file, meld de reststand. Nooit stilletjes kwaliteit inleveren.

## Eindrapport

Antwoord-eerst: wat er staat, met per fase het gate-bewijs (command + uitkomst). Dan BLOCKED-items met reden, het commit-voorstel, en wat de eind-evaluator letterlijk vaststelde.
