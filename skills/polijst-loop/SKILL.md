---
name: polijst-loop
description: Kwaliteits-loop die een UI, tekst of deliverable iteratief verbetert via score-tegen-rubric, gerichte fixes en een verse judge per ronde, tot de score plateaut. Gebruik bij "maak dit mooier/beter", "polish deze pagina", "/polijst-loop", "haal dit naar niveau", of na een eerste werkende versie die nog niet af voelt. Stopt automatisch bij plateau of 4 rondes; verbrandt geen tokens aan eindeloos rondpoetsen.
argument-hint: <wat gepolijst moet worden: pagina, component, tekst, deck>
---

# Polishloop

Jij bent de orchestrator. Judges scoren, workers fixen, jij bewaakt het plateau. Het doelobject: $ARGUMENTS

## Stap 0: rubric (eenmalig)

Stel 5-7 meetbare assen op, toegespitst op het object, elk 1-10 met een omschreven 9+. Voorbeelden:

- **UI**: hierarchie/scale-contrast, spacing-ritme, states (hover/focus), responsiveness op 375 en 1440, geen template-look (design-quality regels), performance (geen layout shift).
- **Tekst/comms**: humanizer-tells weg, de eigen stijl-regels van de gebruiker (indien aanwezig), antwoord-eerst, lengte vs doel, concreet detail.
- **Code/API**: leesbaarheid, naamgeving, file-groottes, foutafhandeling, geen dode paden.

Schrijf de rubric in `polijst-loop-state.md` (scratchpad) met een lege score-historie.

## De loop (max 4 rondes)

1. **Judge** (Agent, verse context per ronde, nooit hergebruiken). `model: "sonnet"` voor mechanische assen; laat het model-parameter weg (Fable) als smaak de kern is. De judge observeert het ECHTE artefact: bij UI via preview-tools (snapshot, inspect, screenshot, resize naar mobile), bij tekst de file zelf, bij code de diff plus de files. Verwacht terug: score per as, totaal, en de **top-3 concreetste fixes** (met file/regel of element).
2. **Fix-worker** (Agent, `model: "sonnet"`). Voert alleen die top-3 uit, chirurgisch. Niet "en passant" andere dingen verbouwen.
3. **Re-score** door een verse judge (zelfde rubric, blind voor de vorige scores). Log de ronde in de state-file.

## Stopcondities (hard)

- Gemiddelde >= 9: klaar.
- Winst < 0.5 punt t.o.v. vorige ronde: plateau, stop. Meer rondes zijn dan ruis.
- 4 rondes: stop hoe dan ook.
- Een as die daalt na een fix: draai die fix terug voor je verder gaat.

## Guardrails

- Judges observeren, workers wijzigen; nooit dezelfde agent voor beide.
- Bij UI: verifieer met inspect (echte CSS-waarden), niet alleen screenshots.
- Functionele regressie is een instant-fail: draai na elke fixronde de bestaande suite of laad de pagina en check de console.
- Geen commits zonder signaal.

## Eindrapport

Score-verloop per ronde (compact tabelletje), wat de grootste sprongen gaf, wat bewust is blijven liggen en waarom.
