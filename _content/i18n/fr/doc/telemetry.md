---
title: "Télémétrie Go"
layout: article
breadcrumb: true
date: 2024-02-07:00:00Z
template: true
---

<style>
.DocInfo {
  background-color: var(--color-background-info);
  padding: 1.5rem 2rem 1.5rem 4rem;
  border-left: 0.875rem solid var(--color-border);
  position: relative;
}
.DocInfo:before {
  content: "ⓘ";
  position: absolute;
  top: 1rem;
  left: 1rem;
  font-size: 2rem;
}
</style>

Table des matières :

 [Contexte](#background)\
 [Vue d'ensemble](#overview)\
 [Configuration](#config)\
 [Compteurs](#counters)\
 [Rapports et envoi](#reports)\
 [Graphiques](#charts) \
 [Propositions de télémétrie](#proposals)\
 [Invite IDE](#ide) \
 [Foire Aux Questions](#faq)

## Contexte {#background}

La télémétrie Go est un moyen pour les programmes de la toolchain Go de collecter des données sur leurs
performances et leur utilisation. Ici, « toolchain Go » désigne les outils de développement maintenus
par l'équipe Go, y compris la commande `go` et les outils complémentaires tels que le
serveur de langage Go [`gopls`] ou l'outil de sécurité Go [`govulncheck`]. La télémétrie Go est
uniquement destinée à être utilisée dans les programmes maintenus par l'équipe Go et leurs dépendances sélectionnées comme [Delve].

Par défaut, les données de télémétrie sont conservées uniquement sur l'ordinateur local, mais les utilisateurs peuvent
opt in à l'envoi d'un sous-ensemble approuvé de données de télémétrie vers [telemetry.go.dev].
Les données envoyées aident l'équipe Go à améliorer le langage Go et ses outils,
en nous aidant à comprendre l'utilisation et les défaillances.

Le mot « telemetry » a acquis des connotations négatives dans le monde du logiciel libre,
souvent à juste titre. Pourtant, mesurer l'expérience utilisateur est un élément important
de l'ingénierie logicielle moderne, et les sources de données comme les issues GitHub ou les enquêtes annuelles sont des indicateurs grossiers et décalés,
insuffisants pour les types de questions auxquels l'équipe Go doit pouvoir répondre.
La télémétrie Go est conçue pour aider les programmes de la toolchain à collecter des données utiles
sur leur fiabilité, leurs performances et leur utilisation, tout en maintenant la
transparence et la confidentialité que les utilisateurs attendent du projet Go. Pour en savoir plus
sur le processus de conception et la motivation de la télémétrie, voir les
[articles de blog sur la télémétrie](https://research.swtch.com/telemetry).
Pour en savoir plus sur la télémétrie et la confidentialité, voir la
[politique de confidentialité de la télémétrie](https://telemetry.go.dev/privacy).

Cette page explique en détail le fonctionnement de la télémétrie Go. Pour des réponses rapides aux
questions fréquentes, voir la [FAQ](#faq).

<div class="DocInfo">
Avec Go 1.23 ou ultérieur, pour <strong>opt in</strong> à l'envoi des données de télémétrie
à l'équipe Go, exécute :
<pre>
go telemetry on
</pre>
Pour désactiver complètement la télémétrie, y compris la collecte locale, exécute :
<pre>
go telemetry off
</pre>
Pour revenir au mode par défaut de télémétrie locale uniquement, exécute :
<pre>
go telemetry local
</pre>
Avant Go 1.23, cela peut également être fait avec la
commande <code>golang.org/x/telemetry/cmd/gotelemetry</code>. Voir <a
href="#config">Configuration</a> pour plus de détails.
</div>

## Vue d'ensemble {#overview}

La télémétrie Go utilise trois types de données principaux :

- Les [_compteurs_](#counters) sont des comptes légers d'événements nommés, instrumentés
  dans le programme de la toolchain. Si la collecte est activée (le [mode](#config)
  est **local** ou **on**), les compteurs sont écrits dans un fichier mappé en mémoire dans le
  système de fichiers local.
- Les [_rapports_](#reports) sont des résumés agrégés des compteurs pour une semaine donnée.
  Si l'envoi est activé (le [mode](#config) est **on**), les rapports pour
  les [compteurs approuvés](#proposals) sont envoyés vers [telemetry.go.dev], où
  ils sont accessibles publiquement.
- Les [_graphiques_](#charts) résument les rapports envoyés par tous les utilisateurs.
  Les graphiques peuvent être consultés sur [telemetry.go.dev].

Toutes les données et la configuration de télémétrie Go locales sont stockées dans le répertoire
<code>[os.UserConfigDir()](/pkg/os#UserConfigDir)/go/telemetry</code>.
Dans la suite, nous ferons référence à ce répertoire comme `<gotelemetry>`.

Le diagramme ci-dessous illustre ce flux de données.

<div class="image">
  <center>
    <img max-width="800px" src="/doc/telemetry/dataflow.png" />
  </center>
</div>

Dans le reste de ce document, nous explorerons les composants de ce diagramme. Mais
d'abord, approfondissons la configuration qui le contrôle.

## Configuration {#config}

Le comportement de la télémétrie Go est contrôlé par une seule valeur : le
_mode_ de télémétrie. Les valeurs possibles pour `mode` sont `local` (la valeur par défaut), `on` ou
`off` :

- Quand `mode` est `local`, les données de télémétrie sont collectées et stockées sur l'ordinateur
  local, mais jamais envoyées vers des serveurs distants.
- Quand `mode` est `on`, les données sont collectées, et peuvent être envoyées selon
  l'[échantillonnage](#uploads).
- Quand `mode` est `off`, les données ne sont ni collectées ni envoyées.

Avec Go 1.23 ou ultérieur, les commandes suivantes interagissent avec le mode de télémétrie :

- `go telemetry` : voir le mode actuel.
- `go telemetry on` : mettre le mode à `on`.
- `go telemetry off` : mettre le mode à `off`.
- `go telemetry local` : mettre le mode à `local`.

Les informations sur la configuration de la télémétrie sont également disponibles via des variables
d'environnement Go en lecture seule :

- `go env GOTELEMETRY` rapporte le mode de télémétrie.
- `go env GOTELEMETRYDIR` rapporte le répertoire contenant la configuration et les données de télémétrie.

La commande [`gotelemetry`](/pkg/golang.org/x/telemetry/cmd/gotelemetry) peut
également être utilisée pour configurer le mode de télémétrie, ainsi que pour inspecter les données de télémétrie
locales. Utilise cette commande pour l'installer :

```
go install golang.org/x/telemetry/cmd/gotelemetry@latest
```

Pour les informations d'utilisation complètes de l'outil en ligne de commande `gotelemetry`,
voir sa [documentation de package](/pkg/golang.org/x/telemetry/cmd/gotelemetry).

## Compteurs {#counters}

Comme mentionné ci-dessus, la télémétrie Go est instrumentée via des _compteurs_. Les compteurs se présentent
en deux variantes : les compteurs de base et les stack counters.

### Compteurs de base

Un _compteur de base_ est une valeur incrémentable avec un nom qui décrit
l'événement qu'il compte. Par exemple, le compteur `gopls/client:vscode` enregistre
le nombre de fois qu'une session `gopls` est initiée par VS Code. À côté de ce
compteur, nous pouvons avoir `gopls/client:neovim`, `gopls/client:eglot`, et ainsi de suite, pour
enregistrer les sessions avec différents éditeurs ou clients de langage. Si tu as utilisé
plusieurs éditeurs au cours de la semaine, tu pourrais enregistrer les données de compteurs suivantes :

    gopls/client:vscode 8
    gopls/client:neovim 5
    gopls/client:eglot  2

Quand les compteurs sont liés de cette façon, nous faisons parfois référence à la partie avant
le `:` comme le _nom du graphique_ (`gopls/client` dans ce cas), et la partie après `:`
comme le _nom du bucket_ (`vscode`). Nous verrons pourquoi cela est important quand nous discuterons
des [graphiques](#charts).

Les compteurs de base peuvent également représenter un _histogramme_. Par exemple, le {{raw
`<code>gopls/completion/latency:&lt;50ms</code>`}} compteur enregistre le nombre
de fois qu'une autocomplétion prend moins de 50 ms.

{{raw `
<pre>
gopls/completion/latency:&lt;10ms
gopls/completion/latency:&lt;50ms
gopls/completion/latency:&lt;100ms
...
</pre>
`}}

Ce pattern pour l'enregistrement des données d'histogramme est une convention : il n'y a rien
de spécial dans le {{raw `<code>&lt;50ms</code>`}} nom de bucket. Ces types de
compteurs sont couramment utilisés pour mesurer les performances.

### Stack counters

Un _stack counter_ est un compteur qui enregistre également la call stack courante du
programme de la toolchain Go quand le compteur est incrémenté. Par exemple, le
stack counter `crash/crash` enregistre la call stack quand un programme de la toolchain
plante :

    crash/crash
    golang.org/x/tools/gopls/internal/golang.hoverBuiltin:+22
    golang.org/x/tools/gopls/internal/golang.Hover:+94
    golang.org/x/tools/gopls/internal/server.Hover:+42
    ...

Les stack counters mesurent généralement des événements où les invariants du programme sont violés.
L'exemple le plus courant est un crash, mais un autre exemple est le
stack counter `gopls/bug`, qui compte les situations inhabituelles identifiées à l'avance
par le programmeur, telles qu'une panic récupérée ou une erreur qui « ne peut pas
se produire ». Les stack counters n'incluent que les noms et numéros de ligne des fonctions
au sein des programmes de la toolchain Go. Ils ne contiennent aucune information sur les
entrées de l'utilisateur, telles que les noms ou le contenu du code source d'un utilisateur.

Les stack counters peuvent aider à retrouver des bugs rares ou délicats qui ne sont pas signalés
par d'autres moyens. Depuis l'introduction du compteur `gopls/bug`, nous avons trouvé
[des dizaines d'instances](https://github.com/golang/go/issues?q=label%3Agopls%2Ftelemetry-wins)
de code « inatteignable » qui a été atteint en pratique, et retrouver ces
exceptions a conduit à la découverte (et à la correction) de nombreux bugs visibles par l'utilisateur qui
n'étaient soit pas évidents pour l'utilisateur, soit trop difficiles à signaler. Surtout avec
les tests de prépublication, les stack counters peuvent nous aider à améliorer le produit plus
efficacement que sans automatisation.

### Fichiers de compteurs

Toutes les données de compteurs sont écrites dans le répertoire `<gotelemetry>/local`, dans
des fichiers nommés selon le schéma suivant :

```
[program name]@[program version]-[go version]-[GOOS]-[GOARCH]-[date].v1.count
```

- Le **nom du programme** est le basename du chemin de package du programme, tel que rapporté
  par [debug.BuildInfo].
- La **version du programme** et la **version go** sont également rapportées par [debug.BuildInfo].
- Les valeurs **GOOS** et **GOARCH** sont rapportées par
  [`runtime.GOOS`](/pkg/runtime#GOOS) et
  [`runtime.GOARCH`](/pkg/runtime#GOARCH).
- La **date** est la date de création du fichier de compteurs, au format `YYYY-MM-DD`.

Ces fichiers sont mappés en mémoire dans chaque instance en cours d'exécution des programmes instrumentés.
L'utilisation d'un fichier mappé en mémoire signifie que même si le programme
plante immédiatement, ou si plusieurs copies d'outils instrumentés s'exécutent
simultanément, les compteurs sont enregistrés de façon sûre.

## Rapports et envoi {#reports}

Approximativement une fois par semaine, les données de compteurs sont agrégées en rapports nommés
`<date>.json` dans le répertoire `<gotelemetry>/local`. Ces rapports additionnent tous les
compteurs de la semaine précédente, regroupés par les mêmes identifiants de programme utilisés pour
le fichier de compteurs (nom du programme, version du programme, version go, GOOS et GOARCH).

Les rapports locaux peuvent être consultés sous forme de graphiques avec la commande
[`gotelemetry view`](/pkg/golang.org/x/telemetry/cmd/gotelemetry).
Voici un exemple de résumé du compteur `gopls/completion/latency` :

<div class="image">
  <center>
    <img max-width="800px" src="/doc/telemetry/gopls-latency.png" />
  </center>
</div>

### Envoi {#uploads}

Si l'envoi de télémétrie est activé, le processus de rapport hebdomadaire générera également
des rapports contenant le sous-ensemble de compteurs présents dans la
[configuration d'envoi](https://telemetry.go.dev/config). Ces compteurs doivent être
approuvés par le processus de révision publique décrit dans la section suivante. Après avoir
été envoyé avec succès, une copie des rapports envoyés est stockée dans
le répertoire `<gotelemetry>/upload`.

Une fois qu'un nombre suffisant d'utilisateurs opt in à l'envoi de données de télémétrie, le processus d'envoi
ignorera aléatoirement l'envoi d'une fraction de rapports, pour réduire les volumes de collecte
et augmenter la confidentialité tout en maintenant la significance statistique.

## Graphiques {#charts}

En plus d'accepter les envois, le site web [telemetry.go.dev] rend les données envoyées
accessibles publiquement. Chaque jour, les rapports envoyés sont traités en deux
sorties, disponibles sur la page d'accueil de [telemetry.go.dev].

- Les rapports _fusionnés_ agrègent les compteurs de tous les envois reçus le jour donné.
- Les _graphiques_ représentent les données envoyées telles que spécifiées dans la [chart config], qui a été
  produite dans le cadre du processus de proposition. En rappelant la discussion sur
  les [compteurs](#counters), les noms de compteurs comme `foo:bar` sont décomposés
  en nom de graphique `foo` et nom de bucket `bar`. Chaque graphique agrège
  les compteurs du même nom de graphique dans les buckets correspondants.

Les graphiques sont spécifiés dans le format du package [chartconfig]. Par exemple,
voici la configuration du graphique `gopls/client`.

    title: Editor Distribution
    counter: gopls/client:{vscode,vscodium,vscode-insiders,code-server,eglot,govim,neovim,coc.nvim,sublimetext,other}
    description: measure editor distribution for gopls users.
    type: partition
    issue: https://go.dev/issue/61038
    issue: https://go.dev/issue/62214 # add vscode-insiders
    program: golang.org/x/tools/gopls
    version: v0.13.0 # temporarily back-version to demonstrate config generation.

Cette configuration décrit le graphique à produire, énumère l'ensemble des
compteurs à agréger, et spécifie les versions du programme auxquelles le
graphique s'applique. De plus, le [processus de proposition](#proposals) exige qu'une proposition
acceptée soit associée au graphique. Voici le graphique résultant de cette configuration :

<div class="image">
  <center>
    <img src="/doc/telemetry/gopls-clients.png" />
  </center>
</div>

## Le processus de proposition de télémétrie {#proposals}

Les modifications apportées à la configuration d'envoi ou à l'ensemble des graphiques sur [telemetry.go.dev] doivent
passer par le _processus de proposition de télémétrie_, qui vise à assurer la
transparence autour des modifications apportées à la configuration de la télémétrie.

Il faut noter qu'il n'y a en fait aucune distinction entre la configuration d'envoi et la
configuration des graphiques dans ce processus. La configuration d'envoi est elle-même exprimée
en termes des agrégations que nous voulons afficher sur telemetry.go.dev, basée
sur le principe que nous ne devons collecter que les données que nous voulons _voir_.

Le processus de proposition est le suivant :

1. Le proposant crée une CL modifiant [config.txt] du package [chartconfig]
   pour contenir les nouvelles agrégations de compteurs souhaitées.
2. Le proposant dépose une [proposition] pour merger cette CL.
3. Une fois que la discussion sur l'issue est résolue, la proposition est approuvée ou refusée
   par un membre de l'équipe Go.
4. Un processus automatique régénère la configuration d'envoi pour permettre l'envoi des
   compteurs requis pour le nouveau graphique. Ce processus ajoutera également régulièrement de
   nouvelles versions des programmes concernés à la configuration d'envoi à mesure qu'ils sont
   publiés.

Pour être approuvés, les nouveaux graphiques ne peuvent pas contenir d'informations sensibles sur l'utilisateur,
et doivent également être à la fois utiles et réalisables. Pour être utiles,
les graphiques doivent servir un objectif spécifique, avec des résultats exploitables. Pour être
réalisables, il doit être possible de collecter de façon fiable les données requises, et les
mesures résultantes doivent être statistiquement significatives. Pour démontrer
la faisabilité, le proposant peut être invité à instrumenter le programme cible avec
des compteurs et à les collecter localement en premier.

L'ensemble complet de ces propositions est disponible dans le
[projet de propositions](https://github.com/orgs/golang/projects/29) sur GitHub.

## Invite IDE {#ide}

Pour que la télémétrie puisse répondre aux types de questions que nous voulons lui poser, l'ensemble des
utilisateurs qui opt in à l'envoi n'a pas besoin d'être grand : environ 16 000
participants permettraient des mesures statistiquement significatives au
niveau de granularité souhaité. Cependant, il y a tout de même un coût pour assembler cet
échantillon sain : nous devons demander à un grand nombre de développeurs Go s'ils veulent
opt in.

De plus, même si un grand nombre d'utilisateurs choisissent d'opt in _maintenant_ (peut-être
après avoir lu un article du blog Go), ces utilisateurs pourraient être biaisés vers des développeurs Go expérimentés,
et avec le temps, cet échantillon initial deviendra encore plus biaisé.
De plus, quand les personnes remplacent leurs ordinateurs, elles doivent activement choisir d'opt in
à nouveau. Dans la série d'articles de blog sur la télémétrie, ceci est appelé le
[« coût de campagne »](https://research.swtch.com/telemetry-opt-in#campaign) du
modèle opt-in.

Pour aider à maintenir l'échantillon des utilisateurs participants à jour, le serveur de langage Go
[`gopls`] prend en charge une invite qui demande aux utilisateurs d'opt in à la télémétrie Go.
Voici à quoi cela ressemble depuis VS Code :

<div class="image">
  <center>
    <img width="600px" src="/doc/telemetry/prompt.png" />
  </center>
</div>

Si les utilisateurs choisissent « Yes », leur [mode](#config) de télémétrie sera mis à `on`,
comme s'ils avaient exécuté
[`gotelemetry on`](/pkg/golang.org/x/telemetry/cmd/gotelemetry). De cette façon,
l'opt-in est aussi simple que possible, et nous pouvons continuellement atteindre un échantillon large et
stratifié de développeurs Go.

## Foire Aux Questions {#faq}

**Q : Comment activer ou désactiver la télémétrie Go ?**

R : Utilise la commande `gotelemetry`, qui peut être installée avec `go install
golang.org/x/telemetry/cmd/gotelemetry@latest`. Exécute `gotelemetry off` pour
tout désactiver, y compris la collecte locale. Exécute `gotelemetry on` pour tout activer,
y compris l'envoi des compteurs approuvés vers [telemetry.go.dev]. Voir
la section [Configuration](#config) pour plus d'informations.

**Q : Où les données locales sont-elles stockées ?**

R : Dans le répertoire <code>[os.UserConfigDir()](/pkg/os#UserConfigDir)/go/telemetry</code>.

**Q : À quelle fréquence les données sont-elles envoyées, si je opt in ?**

R : Approximativement une fois par semaine.

**Q : Quelles données sont envoyées, si je opt in ?**

R : Seuls les compteurs listés dans la
[configuration d'envoi](https://telemetry.go.dev/config) peuvent être envoyés.
Celle-ci est générée depuis la [chart config], qui peut être plus lisible.

**Q : Comment les compteurs sont-ils ajoutés à la configuration d'envoi ?**

R : Via le [processus de proposition public](#proposals).

**Q : Où puis-je voir les données de télémétrie qui ont été envoyées ?**

R : Les données envoyées sont disponibles sous forme de graphiques ou de résumés fusionnés sur [telemetry.go.dev].

**Q : Où se trouve le code source de la télémétrie Go ?**

R : Sur [golang.org/x/telemetry](/pkg/golang.org/x/telemetry).

[`gopls`]: /pkg/golang.org/x/tools/gopls
[`govulncheck`]: /pkg/golang.org/x/vuln/cmd/govulncheck
[Delve]: /pkg/github.com/go-delve/delve#section-readme
[debug.BuildInfo]: /pkg/runtime/debug#BuildInfo
[proposal]: /issue/new?assignees=&labels=Telemetry-Proposal&projects=golang%2F29&template=12-telemetry.yml&title=x%2Ftelemetry%2Fconfig%3A+proposal+title
[telemetry.go.dev]: https://telemetry.go.dev
[chartconfig]: /pkg/golang.org/x/telemetry/internal/chartconfig
[config.txt]: https://go.googlesource.com/telemetry/+/refs/heads/master/internal/chartconfig/config.txt
