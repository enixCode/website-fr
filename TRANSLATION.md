# Translation guide

Ce repo est un fork de `golang.org/x/website` enrichi de traductions localisées du contenu.

## Structure

Les traductions vivent sous `_content/i18n/<locale>/` et miroir la structure de `_content/` :

```
_content/blog/context.md              # source (anglais, upstream)
_content/i18n/fr/blog/context.md      # traduction française
_content/i18n/es/blog/context.md      # traduction espagnole (future)
```

Locales supportées : `fr` (en cours). Codes ISO 639-1 minuscules.

Aucune source upstream n'est modifiée. Les sync upstream (`git fetch upstream && git merge`) ne produisent donc jamais de conflit sur les fichiers de traduction.

## Règles communes à toutes les locales

**Ne jamais traduire :**
- Code, identifiants, noms de packages, signatures de fonctions
- Blocs ` ```...``` ` et `<pre>...</pre>`
- Templates Go : `{{ ... }}`, `{{- ... -}}`
- Frontmatter clé (`title:`, `summary:`, `date:` restent comme clés ; les valeurs textuelles, oui, voir par locale)
- URLs, chemins, ancres
- Noms propres, noms de produits, marques

**Préserver strictement :**
- Structure Markdown (titres, listes, tableaux, liens)
- Indentation et sauts de ligne
- Attributs HTML
- Ordre des paragraphes

**Ton :**
- Direct, technique, sans surcharge. Le style upstream est sobre, on garde sobre.
- Pas d'em-dash (—). Utiliser virgule, parenthèses, deux-points, point.

## Règles par locale

### fr (français)

**Anglicismes à conserver (ne pas traduire) :**
- `goroutine`, `package`, `slice`, `channel`, `runtime`, `garbage collector`
- `interface`, `struct`, `pointer`, `receiver`, `method`
- `build`, `compiler`, `linker`, `runtime`
- `commit`, `pull request`, `merge`, `branch`, `repository` / `repo`
- `framework`, `pipeline`, `endpoint`, `middleware`, `handler`
- `string`, `byte`, `rune`, `int`, `bool` et tous les types Go

**Termes à traduire :**
- function → fonction
- variable → variable
- error → erreur
- value → valeur
- type → type (identique)
- test → test (identique)

**Style :**
- Tutoiement (`tu`) cohérent avec la doc Go originale qui est directe
- Pas de majuscules de politesse
- Garder l'orthographe française complète (accents obligatoires)

**Frontmatter :**
- `title:` : traduire la valeur
- `summary:` : traduire la valeur
- `date:`, `by:`, `tags:` : ne pas toucher

## Workflow

### En local

```bash
./scripts/translate-file.sh fr _content/blog/context.md
# génère _content/i18n/fr/blog/context.md
git add _content/i18n/fr/blog/context.md
git commit -m "i18n(fr): translate _content/blog/context.md"
```

### Sync upstream

```bash
git fetch upstream
git merge upstream/master
# aucun conflit attendu sur _content/i18n/**
```

### Plus tard (runner)

Un workflow GitHub Actions appellera le même script `scripts/translate-file.sh` sur les fichiers modifiés par le merge upstream.

## Convention de commit

- `i18n(<locale>): translate <path>` pour une nouvelle traduction
- `i18n(<locale>): update <path>` pour resynchroniser après un upstream
- `i18n(<locale>): fix <path>` pour corriger une traduction existante

## Feuille de route : quoi traduire, dans quel ordre

Le repo upstream contient ~450 fichiers de contenu. On ne traduit pas tout d'un coup. Priorité par valeur perçue pour un nouvel arrivant francophone.

### Tier 1 — Pages d'atterrissage et navigation (priorité absolue)

L'utilisateur arrive ici en premier. Doit être traduit avant tout le reste.

- [ ] `_content/index.md` (13.8K) — page d'accueil go.dev
- [ ] `_content/help.md` (4.2K) — page Get Help
- [ ] `_content/brand.md` (9.5K) — page de marque
- [ ] `_content/menus.yaml` — navigation principale (labels du header/footer)
- [ ] `_content/resources.yaml` — liens ressources
- [ ] `_content/testimonials.yaml` — citations affichées sur la home
- [ ] `_content/learn/index.md` — page Learn

Pages courtes utilitaires :
- [ ] `_content/tos.md` (405B) — conditions d'utilisation
- [ ] `_content/copyright.md` (385B) — copyright
- [ ] `_content/conduct.html` (8.7K) — code de conduite
- [ ] `_content/project.html` (4.5K) — page projet

### Tier 2 — Documentation canonique Go

Le cœur de la doc, celle que tout le monde lit.

- [ ] `_content/doc/effective_go.html` — Effective Go, doc majeure
- [ ] `_content/doc/faq.md` — FAQ
- [ ] `_content/doc/code.html` — How to Write Go Code
- [ ] `_content/doc/docs.md` — index de la doc
- [ ] `_content/doc/contribute.html` — guide de contribution
- [ ] `_content/doc/editors.html` — éditeurs supportés
- [ ] `_content/doc/install.md` et téléchargement

### Tier 3 — Tour interactif

Énorme valeur pour débutants, mais gros volume.

- [ ] `_content/tour/` — toutes les pages du Tour of Go

### Tier 4 — Articles de blog (à la demande)

432 articles. **Ne pas traduire en masse.** Traduire à la demande, soit pour un article spécifique demandé, soit pour les "classiques" (ex: `context.md`, `error-handling-and-go.md`, `pipelines.md`).

### Tier 5 — Reference et reste

- `_content/ref/` : référence avancée (spec, mémoire). À évaluer, peut rester en EN dans un premier temps.
- `_content/gopls/`, `_content/solutions/`, `_content/wiki/` : à évaluer.

### Ce qu'on ne traduit JAMAIS

- `_content/AUTHORS.md`, `_content/CONTRIBUTORS.md` — listes de noms
- `_content/robots.txt`
- `_content/*.tmpl` : templates Go. Les chaînes UI en dur seront traitées séparément quand on branchera le routeur i18n côté serveur. **Pas dans cette phase.**
- Tout `_content/src/`, `_content/css/`, `_content/js/`, `_content/fonts/`, `_content/images/`, `_content/assets/`

### Page test recommandée

Premier essai : **`_content/help.md`** (4.2K).

Raisons : assez court pour une boucle rapide, contient un mix représentatif (frontmatter avec `title:`, HTML inline, liens externes, anglicismes attendus — mailing list, IRC, Slack — termes à traduire — "help", "community", "get help"). Si la sortie est propre sur `help.md`, le process est validé pour tout le Tier 1.

Procédure de validation après traduction :
1. Lire le fichier de sortie, vérifier que la structure HTML/frontmatter est intacte
2. Vérifier qu'aucun terme de la liste "à conserver" n'a été traduit
3. Vérifier que le ton est cohérent (tutoiement, pas de majuscules de politesse)
4. Lancer `go run ./cmd/golangorg` et confirmer que la page ne casse pas le serveur
