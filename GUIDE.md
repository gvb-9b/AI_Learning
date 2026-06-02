# GUIDE — Comprendre et utiliser ce setup Claude Code

Un cours en français pour comprendre ce qu'on vient de mettre en place dans ce projet, pourquoi chaque pièce existe, comment l'utiliser au quotidien, et comment dupliquer la structure sur n'importe quel autre projet.

---

## Sommaire

| Section | Résumé |
|---------|--------|
| [1. Ce qu'on a construit](#1-ce-quon-a-construit) | Vue d'ensemble en 30 secondes |
| [2. Les 3 briques de Claude Code](#2-les-3-briques-de-claude-code) | CLAUDE.md, sous-agents, slash commands |
| [3. Les deux mémoires (important)](#3-les-deux-mémoires-important) | Différence auto-mémoire vs mémoire projet |
| [4. La règle de lecture de la mémoire](#4-la-règle-de-lecture-de-la-mémoire) | Le principe du sommaire (issu du PDF) |
| [5. L'architecture de ce projet](#5-larchitecture-de-ce-projet) | Chaque fichier, son rôle |
| [6. La boucle d'auto-apprentissage](#6-la-boucle-dauto-apprentissage) | Comment le système s'améliore |
| [7. Au quotidien : workflows](#7-au-quotidien--workflows) | Exemples concrets |
| [8. Dupliquer ce setup ailleurs](#8-dupliquer-ce-setup-ailleurs) | Recette en 5 min |
| [9. Pour aller plus loin](#9-pour-aller-plus-loin) | Hooks, rules, skills |

---

## 1. Ce qu'on a construit

Tu as un projet sandbox avec :

- **Une règle de lecture** qui force Claude à consulter un sommaire avant de lire des fichiers mémoire (le principe du PDF que tu as fourni).
- **Trois sous-agents spécialisés** qui isolent leurs tâches (recherche, revue, écriture de mémoire) dans leur propre fenêtre de contexte.
- **Deux slash commands** qui transforment une intention ("retiens ce truc", "relis et corrige") en un workflow reproductible.
- **Une mémoire projet** structurée en 3 fichiers typés (leçons, décisions, conventions), chacun avec un sommaire à jour.

Le résultat : tu corriges Claude une fois → la leçon est enregistrée → les sessions suivantes ne refont pas la même erreur sans que tu aies à le rappeler.

---

## 2. Les 3 briques de Claude Code

### Brique A — `CLAUDE.md`

Un fichier markdown à la racine du projet, **chargé automatiquement dans chaque session**. C'est l'endroit où tu écris ce que Claude doit savoir sur le projet en permanence : conventions, layout, règles "fais toujours X". La doc Anthropic recommande de rester sous 200 lignes.

`CLAUDE.md` supporte la syntaxe d'import `@chemin/fichier.md` : le fichier importé est chargé en entier au lancement de la session. C'est comme ça qu'on injecte `memory/INDEX.md` dans chaque session.

### Brique B — Les sous-agents (`.claude/agents/<nom>.md`)

Des agents spécialisés que l'agent principal **délègue** quand il rencontre une tâche qui matche leur description. Chaque sous-agent :

- a sa **propre fenêtre de contexte** (la pollution de recherche ou de revue n'envahit pas la conversation principale) ;
- a son **propre system prompt** (le corps markdown du fichier) ;
- peut avoir des **outils restreints** (ex: code-reviewer en read-only) ;
- peut tourner sur un **modèle différent** (`model: inherit`, `sonnet`, `haiku`, `opus`).

Tu n'as pas besoin de les appeler explicitement. Si la description du sous-agent matche la tâche en cours, Claude principal délègue tout seul.

### Brique C — Les slash commands (`.claude/commands/<nom>.md`)

Des raccourcis pour des workflows que tu réutilises. Tu tapes `/learn ça` → Claude exécute le contenu du fichier `learn.md` comme une instruction, en remplaçant `$ARGUMENTS` par "ça".

Note doc : Anthropic a fusionné les "commands" dans les "skills" (format `.claude/skills/<nom>/SKILL.md`). Les vieux `.claude/commands/*.md` fonctionnent toujours et sont plus simples (un seul fichier). On reste sur ce format pour le moment.

---

## 3. Les deux mémoires (important)

**C'est le point qui crée le plus de confusion. Lis bien.**

Claude Code a **DEUX systèmes de mémoire** distincts qui coexistent :

| | Auto-mémoire | Mémoire projet (ce qu'on a fait) |
|---|---|---|
| **Qui écrit** | Claude tout seul | Toi (via `/learn`) |
| **Emplacement** | `~/.claude/projects/<projet>/memory/` | `<projet>/memory/` (dans le repo) |
| **Versionné Git** | Non (machine-locale) | Oui (committé) |
| **Partagé équipe** | Non | Oui |
| **Quand ça charge** | Les 200 premières lignes de `MEMORY.md` dans chaque session | `@memory/INDEX.md` via CLAUDE.md |
| **Usage typique** | Préférences perso que Claude apprend (style, build commands) | Décisions projet, leçons d'erreurs, conventions équipe |

**Les deux sont utiles, pour des choses différentes.** L'auto-mémoire fait son boulot sans que tu t'en occupes. La mémoire projet est ce que tu construis intentionnellement.

Dans ce setup, c'est la **mémoire projet** qu'on outille (INDEX, leçons, décisions, conventions), parce que c'est ce qui sert l'objectif "self-learning équipe-compatible".

---

## 4. La règle de lecture de la mémoire

C'est le principe central du PDF que tu as fourni, appliqué ici.

### Le problème

Quand un fichier mémoire grossit (500, 1000 lignes), Claude le relit en entier à chaque consultation. Coûteux, lent, et selon l'étude *Lost in the Middle* (Stanford) la qualité des réponses chute quand l'info est noyée dans un long contexte.

### La solution

Chaque fichier mémoire commence par un **TABLE OF CONTENTS** :

```markdown
## TABLE OF CONTENTS

| Section title | One-line summary | Line |
|---------------|------------------|------|
| Choix de la base | Pourquoi Postgres et pas Mongo | 24 |
| Tarification | Grille de prix actuelle | 58 |
| Piège déploiement | Erreur de migration qui a coûté un week-end | 96 |
```

Et `CLAUDE.md` contient la règle :

> 1. Lis `@memory/INDEX.md` pour savoir quel fichier ouvrir.
> 2. Ouvre le fichier et lis SEULEMENT son TABLE OF CONTENTS.
> 3. Saute au numéro de ligne. Lis seulement cette section.
> 4. NE LIS JAMAIS le fichier en entier.

Le `memory-keeper` sous-agent maintient les numéros de ligne à jour à chaque ajout.

### Pourquoi ça marche

Anthropic recommande exactement cette approche dans son article ["Effective context engineering for AI agents"](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) : garder des **repères légers** (chemins, numéros de ligne) et charger l'info à la demande. Chaque token chargé inutilement épuise le "budget d'attention" du modèle.

---

## 5. L'architecture de ce projet

```
AI_learning/
├── CLAUDE.md                    # Instructions projet + règle de lecture + @import INDEX
├── GUIDE.md                     # Ce fichier (cours)
├── index.html, index.css, script.js  # Code jouet (peu importe)
│
├── .claude/
│   ├── agents/
│   │   ├── code-reviewer.md     # Revue de code (read-only, cyan)
│   │   ├── explorer.md          # Recherche dans le projet (read-only, blue)
│   │   └── memory-keeper.md     # Écrit/maintient memory/ (yellow)
│   ├── commands/
│   │   ├── learn.md             # /learn → memory-keeper
│   │   └── review-and-fix.md    # /review-and-fix → code-reviewer + edits
│   └── settings.json            # Permissions (bash git/grep/ls/wc/find sans prompt)
│
└── memory/
    ├── INDEX.md                 # Sommaire des fichiers mémoire (importé par CLAUDE.md)
    ├── lessons.md               # Erreurs passées + règle pour les éviter
    ├── decisions.md             # Choix d'architecture + raison
    └── conventions.md           # Style de code + organisation
```

### Ce qui est chargé à chaque session

- `CLAUDE.md` (en entier — fait ~50 lignes, OK)
- `memory/INDEX.md` (via `@import` dans CLAUDE.md)

C'est tout. Les fichiers mémoire (`lessons.md`, etc.) ne sont PAS chargés au lancement — ils sont lus à la demande, et seulement la section utile, grâce à la règle de lecture.

### Ce qui n'est PAS chargé au lancement

- Les sous-agents → chargés seulement quand l'agent principal délègue.
- Les slash commands → chargés seulement quand tu tapes `/learn` ou `/review-and-fix`.
- Les sections des fichiers mémoire → chargées par `Read` ciblé sur un offset.

---

## 6. La boucle d'auto-apprentissage

Voilà comment le système "apprend" :

```
┌─────────────────────────────────────────────────────────────┐
│  Session N                                                  │
│                                                             │
│  1. Tu demandes une tâche                                   │
│  2. Claude code                                             │
│  3. Tu corriges : "non, pas comme ça, fais X au lieu de Y"  │
│  4. Tu tapes :  /learn "toujours X, jamais Y, parce que Z"  │
│  5. memory-keeper écrit l'entrée dans memory/lessons.md     │
│     + met à jour la TABLE OF CONTENTS                       │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ (commit, push, partage équipe)
┌─────────────────────────────────────────────────────────────┐
│  Session N+1 (peut-être un autre dev)                       │
│                                                             │
│  1. CLAUDE.md chargé → règle de lecture connue              │
│  2. memory/INDEX.md chargé → Claude sait que lessons.md     │
│     contient des leçons                                     │
│  3. Tâche similaire arrive                                  │
│  4. Claude consulte la TOC de lessons.md → voit "toujours X"│
│  5. Claude saute à la section, lit, applique X dès le 1er   │
│     coup                                                    │
│  ✅ La même erreur n'est pas refaite                        │
└─────────────────────────────────────────────────────────────┘
```

**Important** : Claude ne "s'améliore" pas tout seul. C'est TOI qui captures les leçons via `/learn`. Le système organise et présente ces leçons de manière à ce que Claude les consulte au bon moment. C'est de l'**apprentissage assisté**, pas de l'apprentissage autonome.

---

## 7. Au quotidien : workflows

### Workflow A — Tâche normale

```
Toi: "Ajoute une fonction validateEmail dans script.js"
Claude: [code la fonction]
Toi: /review-and-fix
Claude: [délègue à code-reviewer → liste les findings]
       1. [BUG] script.js:42 — regex n'accepte pas les + dans l'email
       2. [STYLE] script.js:50 — préférer const à let
       Lesquels appliquer ?
Toi: 1
Claude: [applique fix 1, ignore fix 2]
```

### Workflow B — Capturer une leçon après correction

```
Toi: [Claude vient de mocker la DB dans un test]
Toi: "Non, ne mocke jamais la DB dans les tests d'intégration,
      on s'est fait avoir au dernier sprint avec une migration
      qui passait mockée et cassait en prod"
Toi: /learn "tests d'intégration : jamais de mock DB. Raison:
            mock cache divergence schema → migration prod cassée"
Claude: [délègue à memory-keeper → écrit dans lessons.md]
       Saved to memory/lessons.md → ## tests-no-mock-db (line 24)
```

### Workflow C — Recherche avant changement

Tu n'as rien à faire de spécial. Quand Claude principal a besoin de trouver un truc, il délègue à `explorer` automatiquement (parce que la description du sous-agent matche). Tu peux forcer :

```
Toi: "Trouve toutes les occurrences de fetch() dans le projet"
Claude: [délègue à explorer → résultat ciblé en contexte propre]
```

### Workflow D — Ajouter une décision d'archi

```
Toi: /learn "on garde du vanilla JS, pas de framework. Raison:
            projet pédagogique, framework masquerait les concepts"
Claude: [memory-keeper détecte que c'est une décision, pas une
        leçon → écrit dans decisions.md]
       Saved to memory/decisions.md → ## no-framework-vanilla-js
```

---

## 8. Dupliquer ce setup ailleurs

Tu veux le même squelette dans un autre projet ? Recette en 5 minutes :

### Option 1 — Copie manuelle

```bash
cd /chemin/vers/nouveau-projet
cp -r /Users/gvb/projex/AI_learning/.claude .
cp -r /Users/gvb/projex/AI_learning/memory .
cp /Users/gvb/projex/AI_learning/CLAUDE.md .
cp /Users/gvb/projex/AI_learning/GUIDE.md .
```

Puis édite `CLAUDE.md` pour remplacer la première section ("AI_learning — Claude Code learning sandbox") par la description du nouveau projet. Le reste (règle de lecture, layout, conventions) reste tel quel.

Vide les entrées des fichiers mémoire (`lessons.md`, `decisions.md`, `conventions.md`) — leur structure reste, les entrées sont spécifiques au projet.

### Option 2 — Demander à Claude

Dans le nouveau projet, lance Claude Code et tape :

> Réplique le setup du projet ~/projex/AI_learning ici : copie CLAUDE.md, le dossier .claude/, le dossier memory/ avec sa structure mais sans les entrées. Adapte la première section de CLAUDE.md au contexte de ce projet.

### Option 3 — En faire un template Git

À terme : `git init` ce projet, push sur GitHub comme template, puis `gh repo create --template`. Hors scope ici.

---

## 9. Pour aller plus loin

Une fois à l'aise avec ce setup, tu peux ajouter :

### Hooks (`.claude/settings.json`)

Des commandes shell exécutées automatiquement à certains événements (avant un edit, après un commit, etc.). Utile pour forcer un lint ou refuser une action. Doc : https://code.claude.com/docs/en/hooks-guide

Exemple d'usage ici : un hook `PreToolUse` qui empêche Claude d'éditer un fichier mémoire sans passer par `memory-keeper`.

### Path-scoped rules (`.claude/rules/<topic>.md`)

Des règles markdown qui ne se chargent QUE quand Claude touche à certains fichiers (via une glob `paths:` en frontmatter). Plus fin que `CLAUDE.md` qui charge tout, tout le temps.

Exemple : `.claude/rules/css.md` avec `paths: ["**/*.css"]` ne se charge que quand Claude lit/édite un fichier CSS.

### Skills (`.claude/skills/<nom>/SKILL.md`)

L'évolution des slash commands. Permet d'avoir un dossier avec scripts, exemples, et templates en plus du fichier principal. À considérer quand un workflow devient trop complexe pour un seul fichier markdown.

### Le script de mise à jour automatique du sommaire

Pour l'instant, `memory-keeper` met à jour les TABLE OF CONTENTS quand il écrit. Si tu édites les fichiers mémoire à la main, tu peux lui demander de re-indexer :

> Re-index memory/lessons.md

Si tu veux automatiser ça à chaque modif de fichier mémoire, c'est un hook `PostToolUse` à brancher sur Edit/Write avec un matcher sur `memory/*.md`.

### Multi-agents au-delà de Claude Code

Si un jour tu veux des agents qui tournent vraiment en parallèle/autonomie (au-delà de la délégation Claude Code) : regarde le [Claude Agent SDK](https://docs.claude.com/en/api/agent-sdk/overview). C'est un autre monde — agents qui tournent comme des processus, communiquent, persistent. Hors scope ici, mais bon à connaître.

---

## Vérifier que tout fonctionne

Pour tester la boucle de bout en bout, tape dans Claude Code :

```
/learn "test : le guide GUIDE.md doit toujours être maintenu à jour quand on change la structure du projet"
```

Tu devrais voir :
1. Claude réfléchir à quel fichier ça appartient (probablement `conventions.md` ou `decisions.md`).
2. Déléguer à `memory-keeper`.
3. `memory-keeper` écrire l'entrée + mettre à jour le TABLE OF CONTENTS.
4. Retour : "Saved to memory/<file>.md → ## <title> (line N)".

Si ça marche, le système est en place. Tu peux supprimer l'entrée de test après.
