---
name: fix-loop
description: Autonome fix-loop die een falende test, build of bug repareert tot het doelcommand groen is, met verse diagnose-workers per iteratie en een harde stop na 5 pogingen. Gebruik bij "fix deze bug", "maak de build groen", "deze test faalt", "/fix-loop", of elke failure met een herhaalbaar verificatiecommand. Token-zuinig alternatief voor een handmatige debug-sessie: de orchestrator implementeert nooit zelf en herdraait elke check met eigen exit code.
argument-hint: <failing command, foutmelding of bug-omschrijving>
---

# Fixloop

Jij bent de orchestrator. Je implementeert nooit zelf, je spawnt workers en herdraait checks. De taak: $ARGUMENTS

## Principe

Klaar = het doelcommand geeft exit 0 in JOUW terminal, niet in het rapport van een worker. Elke iteratie krijgt een verse worker-context; geheugen leeft op disk.

## Stap 0: baseline (eenmalig)

1. Bepaal het **doelcommand**: het exacte command dat nu faalt (test, build, typecheck, repro-script). Gaf de gebruiker alleen een symptoom, detecteer de stack en schrijf desnoods eerst een minimale repro-test (die is dan het doelcommand).
2. Draai het zelf. Leg vast: exact command, exit code, de relevante failure-output (max 50 regels, gefilterd).
3. Maak `fix-loop-state.md` in de scratchpad met: doelcommand, baseline-failure, lege pogingenlijst.

Geen baseline die faalt = niks te fixen, meld dat en stop.

## De loop (max 5 iteraties)

Per iteratie:

1. **Diagnose-worker** (Agent, `model: "sonnet"`, verse context). Prompt bevat uitsluitend: de failure-output, het doelcommand, de pogingen-sectie uit `fix-loop-state.md`, en de instructie zelf de relevante code te lezen. Verwacht terug: hypothese, bewijs (welke regel, welke waarde), minimaal fix-plan, te wijzigen files. Geen fix zonder bewijs uit de code.
2. **Fix-worker** (Agent, `model: "sonnet"`). Voert alleen het fix-plan uit, chirurgisch, minimale diff. Committet niet. Rapporteert de diff.
3. **Verificatie door jou**: draai het doelcommand opnieuw. Eigen exit code, negeer wat de worker beweert.
   - Exit 0: door naar afronding.
   - Faalt anders dan eerst: vooruitgang, log de poging in de state-file en volgende iteratie.
   - Faalt identiek: log het, en tel identieke failures.

## Escalatie en stop

- 2x identieke failure: escaleer de diagnose-worker naar `model: "opus"` (of laat het model-parameter weg zodat hij Fable erft) en geef expliciet mee welke hypotheses al sneuvelden.
- 5 iteraties zonder groen: STOP. Rapporteer de pogingen, de beste hypothese en wat een mens moet aanleveren. Eindeloos branden is verboden.

## Guardrails

- Nooit een test aanpassen of skippen om groen te krijgen. Is de test zelf aantoonbaar fout, meld dat expliciet met bewijs en vraag om een beslissing.
- Regressie-check aan het eind: draai eenmaal de bredere suite (of typecheck + lint). Nieuw rood door de fix = terug de loop in met dat als doelcommand.
- Geen commits; werk blijft in de working tree (commit-discipline).

## Eindrapport

Antwoord-eerst: wat de fout was, wat de fix is (1-2 zinnen), welke files geraakt, bewijs (command + exit 0). Daarna eventueel een commit-groep-voorstel.
