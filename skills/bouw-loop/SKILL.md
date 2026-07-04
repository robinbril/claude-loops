---
name: bouw-loop
description: Story-loop die een feature of tasklist story-voor-story bouwt met een verse sonnet-worker per story en verificatie door de orchestrator. Ondersteunt parallelle waves via de Workflow-tool voor onafhankelijke stories. Gebruik bij "bouw dit", "werk de tasklist af", "/bouw-loop", "implementeer deze stories", of een feature die opgeknipt kan worden. Het token-zuinige bouw-hart: plan zwaar (eenmalig), voer licht uit, verifieer per story precies een keer.
argument-hint: <feature-omschrijving of pad naar tasklist>
---

# Shiploop

Jij bent de orchestrator. Je bouwt nooit zelf, je plant eenmalig, spawnt per story een verse worker en verifieert met eigen tool-output. De taak: $ARGUMENTS

## Stap 0: plan (de enige dure stap)

Is er al een tasklist (pad meegegeven of `bouw-loop/plan.md` bestaat), lees die. Anders maak je hem zelf in de hoofdsessie (Fable, dit mag kosten):

- Recon: lees de relevante code, snap de conventies.
- Knip de taak in stories van elk max ~1 worker-context: per story een titel, 2-5 zinnen spec, **een toetsbaar acceptatiecriterium** (command + verwacht resultaat, of deliverable-pad, of observeerbaar gedrag), en optioneel `dependsOn`.
- Schrijf naar `bouw-loop/plan.md` (checkboxes) plus `bouw-loop/progress.md` (leeg, voor learnings).

Zwak criterium ("werkt goed") is verboden; herschrijf tot ja/nee-toetsbaar.

## Per story (sequentieel, default)

1. **Worker** (Agent, `model: "sonnet"`, verse context). Prompt bevat: de story + acceptatiecriterium, de learnings uit `bouw-loop/progress.md`, relevante file-paths uit je recon, en de huisregels: chirurgische diff, bestaande stijl matchen, niet committen, geen scope buiten de story.
2. **Verificatie door jou**: voer het acceptatiecriterium zelf uit. Command herdraaien met eigen exit code, deliverable zelf lezen, of bij UI de preview-tools gebruiken (snapshot/inspect, niet alleen screenshot). Het worker-rapport is geen bewijs.
3. Pass: vink af in `plan.md`, schrijf herbruikbare learnings (patronen, valkuilen, commands) naar `progress.md`, volgende story.
4. Fail: een retry met de failure-output erbij. Tweede fail: markeer `BLOCKED` met reden, ga door met stories die er niet van afhangen.

## Waves (parallel, wanneer stories onafhankelijk zijn)

Zijn er 3+ stories zonder onderlinge `dependsOn` en zonder file-overlap, draai ze als wave via de Workflow-tool: `pipeline(stories, story => agent(workerPrompt, {model: 'sonnet', schema: RESULT}))`, met `isolation: 'worktree'` alleen als file-overlap niet uit te sluiten is. Na de wave verifieer je elke story alsnog zelf, een voor een, en merge je worktrees pas na groen. Afhankelijke stories daarna sequentieel.

## Stop en afronding

- Alle stories af of BLOCKED: stop. Nooit doorgaan met verzinnen van extra werk.
- Eindcheck: draai eenmaal de volledige suite + typecheck.
- Geen commits zonder signaal. Eindig met: wat af is, wat BLOCKED (en waarom), bewijs per story (command + resultaat), en een voorstel van 3-7 thematische commit-groepen (commit-discipline).
