# Claude Loops

Vijf gerichte loop-skills plus een orchestratie-chain voor Claude Code. Elke loop pakt een ander soort werk, en alle zes delen dezelfde discipline: de orchestrator implementeert nooit zelf, geheugen leeft op disk in plaats van in de context, elke check wordt herdraaid met een eigen exit code, en elke loop heeft een harde stopconditie zodat hij nooit eindeloos tokens verbrandt.

Het token-zuinige zusje van [claude-supergoal](https://github.com/robinbril/claude-supergoal): dezelfde kerngedachte (niks is klaar tot iets onafhankelijks het bewijst), maar met een fractie van de overhead. Supergoal bewaar je voor werk dat de volle evaluator-plus-adviesraad-machinerie waard is; voor de rest pak je de smalste loop die past.

## De collectie

| Skill | Doel | Stopt bij |
|---|---|---|
| `/bugfix-loop` | een falende test, build of bug naar groen | groen, of hard na 5 iteraties |
| `/feature-loop` | een feature of tasklist story-voor-story bouwen, optioneel in parallelle waves | alles af of BLOCKED |
| `/quality-loop` | een UI, tekst of deliverable naar niveau via rubric-scores | score-plateau of 4 rondes |
| `/audit-loop` | onbekende hoeveelheden vondsten: bugs, security, dead code, claims | 2 droge rondes |
| `/monitor-loop` | externe state bewaken: CI, deploy, DNS, een URL | trigger afgehandeld of max wakes |
| `/chain-loop` | end-to-end shippen door de loops aan elkaar te rijgen | ACCEPT van de eind-evaluator |

## Waarom loops in plaats van een prompt

Een agent die je handmatig prompt stopt zodra hij zichzelf klaar verklaart, en dat is precies het moment waarop je hem niet kunt vertrouwen. Een goed ontworpen loop legt vooraf vast wat "klaar" bewijst, laat verse contexten het werk doen zodat niks zich ophoopt, en stopt op een meetbare conditie in plaats van op een gevoel.

De gedeelde principes, in elke skill ingebakken:

- **Plan zwaar, voer licht uit.** De denk-stap draait een keer op het hoofdmodel; de uitvoering gaat naar goedkope workers met een verse, minimale context per iteratie.
- **Geheugen op disk.** Plan, voortgang en learnings staan in state-files. Verse workers lezen die in plaats van de conversatie, dus een lange run loopt de context niet vol.
- **Verificatie door de orchestrator zelf.** Een worker-rapport is geen bewijs. Het doelcommand wordt herdraaid, de deliverable zelf gelezen, de pagina zelf geladen.
- **Harde stopcondities.** Iteratie-caps, plateau-detectie, droogte-tellers. Geen enkele loop kan blijven hangen.
- **Geen autonome commits.** Werk verzamelt in de working tree; de loop eindigt met een voorstel van thematische commit-groepen en wacht op jouw signaal.

## Model-routing

Alles draait op Claude-modellen via de model-parameter van de Agent/Workflow-tools, geen externe CLI's of tweede provider nodig:

- het sessiemodel orchestreert en doet de eenmalige plan-stap
- `sonnet` doet het bouw- en zoekwerk (workers, finders, judges op mechanische assen)
- `opus` alleen waar het oordeel de moat is: de eind-evaluator van de chain en second opinions op high-severity vondsten

Draai je alles op een model, dan werkt elke loop ook; routing is een besparing, geen vereiste.

## Installatie

```bash
git clone https://github.com/robinbril/claude-loops
cp -r claude-loops/skills/* ~/.claude/skills/
```

Daarna zijn ze beschikbaar als slash-commands (`/bugfix-loop`, `/feature-loop`, enzovoort) in elke Claude Code-sessie.

## Welke loop pak je

De vuistregel: de smalste loop die past.

- Iets is stuk en je hebt (of maakt) een herhaalbaar failing command: `/bugfix-loop`.
- Het werkt maar voelt niet af: `/quality-loop`.
- Er moet iets gebouwd worden dat in stories op te knippen is: `/feature-loop`.
- Je weet niet wat er allemaal mis is en wilt het complete beeld: `/audit-loop` (vindt en bewijst, fixt niet; de vondsten gaan door naar bugfix- of feature-loop).
- Je wacht op iets buiten de sessie (CI, deploy, DNS): `/monitor-loop`.
- Een niet-triviale taak end-to-end zonder babysitten: `/chain-loop`, die recon, plan, bouw, polish en fix aan elkaar rijgt en afsluit met een onafhankelijke evaluator die tegen de oorspronkelijke opdracht toetst.

## Chain-loop vs supergoal

Beide eindigen pas als iets onafhankelijks het werk bewijst tegen de oorspronkelijke opdracht. Het verschil is de overhead per fase: supergoal draait per fase een volledige evaluator plus een adviesraad-gate plus retry-met-rollback, en interviewt je uitgebreid in de planfase. Chain-loop doet een evaluator-pass per gate, een herstelronde bij REJECT, en stelt hooguit een scherpe vraag als de scope ambigu is. Grofweg: supergoal voor het werk van een dag, chain-loop voor het werk van een middag.

## Licentie

MIT
