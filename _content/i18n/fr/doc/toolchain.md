---
title: "Toolchains Go"
layout: article
---

## Introduction {#intro}

Depuis Go 1.21, la distribution Go comprend une commande `go` et un Go toolchain intégré,
c'est-à-dire la bibliothèque standard ainsi que le compiler, l'assembleur et les autres outils.
La commande `go` peut utiliser son Go toolchain intégré ainsi que d'autres versions
qu'elle trouve dans le `PATH` local ou télécharge selon les besoins.

Le choix du Go toolchain utilisé dépend du paramètre d'environnement `GOTOOLCHAIN`
et des lignes `go` et `toolchain` du fichier `go.mod` du module principal ou du fichier `go.work` du workspace courant.
En passant d'un module principal à un autre ou d'un workspace à un autre,
la version du toolchain utilisée peut varier, tout comme le font les versions des dépendances de modules.

Dans la configuration standard, la commande `go` utilise son propre toolchain intégré
quand ce toolchain est au moins aussi récent que les lignes `go` ou `toolchain` du module principal ou du workspace.
Par exemple, en utilisant la commande `go` intégrée à Go 1.21.3 dans un module principal qui déclare `go 1.21.0`,
la commande `go` utilise Go 1.21.3.
Quand la ligne `go` ou `toolchain` est plus récente que le toolchain intégré,
la commande `go` exécute plutôt le toolchain plus récent.
Par exemple, en utilisant la commande `go` intégrée à Go 1.21.3 dans un module principal qui déclare `go 1.21.9`,
la commande `go` trouve et exécute Go 1.21.9.
Elle cherche d'abord dans le PATH un programme nommé `go1.21.9`, et sinon télécharge et met en cache
une copie du toolchain Go 1.21.9.
Ce basculement automatique de toolchain peut être désactivé, mais dans ce cas,
pour une meilleure compatibilité ascendante,
la commande `go` refusera de s'exécuter dans un module principal ou un workspace où la ligne `go`
exige une version plus récente de Go.
Autrement dit, la ligne `go` définit la version minimale requise de Go pour utiliser un module ou un workspace.

Les modules qui sont des dépendances d'autres modules peuvent avoir besoin de déclarer une version minimale de Go
inférieure au toolchain préféré à utiliser lorsqu'on travaille directement dans ce module.
Dans ce cas, la ligne `toolchain` dans `go.mod` ou `go.work` définit un toolchain préféré
qui a la priorité sur la ligne `go` quand la commande `go` décide
quel toolchain utiliser.

Les lignes `go` et `toolchain` peuvent être vues comme spécifiant les exigences de version
pour la dépendance du module vis-à-vis du Go toolchain lui-même, tout comme les lignes `require` dans `go.mod`
spécifient les exigences de version pour les dépendances envers d'autres modules.
La commande `go get` gère la dépendance envers le Go toolchain de la même façon qu'elle
gère les dépendances envers d'autres modules.
Par exemple, `go get go@latest` met à jour le module pour exiger le dernier Go toolchain publié.

Le paramètre d'environnement `GOTOOLCHAIN` peut imposer une version spécifique de Go, en ignorant
les lignes `go` et `toolchain`. Par exemple, pour tester un package avec Go 1.21rc3 :

	GOTOOLCHAIN=go1.21rc3 go test

Le paramètre `GOTOOLCHAIN` par défaut est `auto`, qui active le basculement de toolchain décrit précédemment.
La forme alternative `<name>+auto` définit le toolchain par défaut à utiliser avant de décider s'il faut
basculer davantage. Par exemple, `GOTOOLCHAIN=go1.21.3+auto` demande à la commande `go` de
commencer sa décision avec Go 1.21.3 par défaut, mais d'utiliser quand même un toolchain plus récent si
les lignes `go` et `toolchain` le demandent.
Comme le paramètre `GOTOOLCHAIN` par défaut peut être modifié avec `go env -w`,
si tu as Go 1.21.0 ou une version ultérieure installée, alors

	go env -w GOTOOLCHAIN=go1.21.3+auto

est équivalent à remplacer ton installation de Go 1.21.0 par Go 1.21.3.

Le reste de ce document explique plus en détail comment les toolchains Go sont versionnés, choisis et gérés.

## Versions de Go {#version}

Les versions publiées de Go utilisent la syntaxe de version `1.*N*.*P*`, qui désigne la *P*-ième publication de Go 1.*N*.
La publication initiale est 1.*N*.0, comme dans `1.21.0`. Les publications ultérieures comme 1.*N*.9 sont souvent appelées patch releases.

Les release candidates de Go 1.*N*, émises avant 1.*N*.0, utilisent la syntaxe de version `1.*N*rc*R*`.
La première release candidate de Go 1.*N* a la version 1.*N*rc1, comme dans `1.23rc1`.

La syntaxe `1.*N*` est appelée « version de langage ». Elle désigne l'ensemble de la famille des publications Go
qui implémentent cette version du langage Go et de la bibliothèque standard.

La version de langage pour une version de Go est le résultat de la troncature de tout ce qui suit le *N* :
1.21, 1.21rc2 et 1.21.3 implémentent tous la version de langage 1.21.

Les toolchains Go publiés comme Go 1.21.0 et Go 1.21rc1 rapportent cette version spécifique
(par exemple, `go1.21.0` ou `go1.21rc1`)
via `go version` et [`runtime.Version`](/pkg/runtime/#Version).
Les toolchains Go non publiés (encore en développement) construits depuis le dépôt de développement Go
rapportent à la place uniquement la version de langage (par exemple, `go1.21`).

Deux versions de Go quelconques peuvent être comparées pour décider si l'une est inférieure, supérieure,
ou égale à l'autre. Si les versions de langage diffèrent, c'est cela qui détermine la comparaison :
1.21.9 < 1.22. Au sein d'une version de langage, l'ordre du plus petit au plus grand est :
la version de langage elle-même, puis les release candidates ordonnées par *R*, puis les publications ordonnées par *P*.

Par exemple, 1.21 < 1.21rc1 < 1.21rc2 < 1.21.0 < 1.21.1 < 1.21.2.

Avant Go 1.21, la publication initiale d'un toolchain Go était la version 1.*N*, non 1.*N*.0,
donc pour *N* < 21, l'ordre est ajusté pour placer 1.*N* après les release candidates.

Par exemple, 1.20rc1 < 1.20rc2 < 1.20rc3 < 1.20 < 1.20.1.

Les versions antérieures de Go avaient des beta releases, avec des versions comme 1.18beta2.
Les beta releases sont placées immédiatement avant les release candidates dans l'ordre des versions.

Par exemple, 1.18beta1 < 1.18beta2 < 1.18rc1 < 1.18 < 1.18.1.

<!-- Unpublished note: the download page also lists Go 1.9.2rc2, which does not respect
this version syntax. That was created as a test of some potential release automation
before Go 1.9.2 but is not considered a "real" toolchain. -->

## Noms des toolchains Go {#name}

Les toolchains Go standard sont nommés <code>go<i>V</i></code> où *V* est une version de Go
désignant une beta release, une release candidate, ou une publication.
Par exemple, `go1.21rc1` et `go1.21.0` sont des noms de toolchain ;
`go1.21` et `go1.22` ne le sont pas (les publications initiales sont `go1.21.0` et `go1.22.0`),
mais `go1.20` et `go1.19` le sont.

Les toolchains non standard utilisent des noms de la forme <code>go<i>V</i>-<i>suffix</i></code>
pour tout suffix.

Les toolchains sont comparés en comparant la version <code><i>V</i></code> intégrée dans le nom
(en supprimant le `go` initial et en ignorant tout suffix commençant par `-`).
Par exemple, `go1.21.0` et `go1.21.0-custom` sont considérés égaux à des fins de classement.

## Configuration des modules et workspaces {#config}

Les modules et workspaces Go spécifient la configuration liée aux versions
dans leurs fichiers `go.mod` ou `go.work`.

La ligne `go` déclare la version minimale requise de Go pour utiliser
le module ou le workspace.
Pour des raisons de compatibilité, si la ligne `go` est absente d'un fichier `go.mod`,
le module est considéré comme ayant une ligne implicite `go 1.16`,
et si la ligne `go` est absente d'un fichier `go.work`,
le workspace est considéré comme ayant une ligne implicite `go 1.18`.

La ligne `toolchain` déclare un toolchain suggéré à utiliser avec
le module ou le workspace.
Comme décrit dans « [Sélection du toolchain Go](#select) » ci-dessous,
la commande `go` peut exécuter ce toolchain spécifique lorsqu'elle opère
dans ce module ou workspace
si la version du toolchain par défaut est inférieure à celle du toolchain suggéré.
Si la ligne `toolchain` est absente,
le module ou workspace est considéré comme ayant une ligne implicite
<code>toolchain go<i>V</i></code>,
où *V* est la version de Go de la ligne `go`.

Par exemple, un `go.mod` qui indique `go 1.21.0` sans ligne `toolchain`
est interprété comme s'il avait une ligne `toolchain go1.21.0`.

Le toolchain Go refuse de charger un module ou un workspace qui déclare
une version minimale requise de Go supérieure à la version propre du toolchain.

Par exemple, Go 1.21.2 refusera de charger un module ou un workspace
avec une ligne `go 1.21.3` ou `go 1.22`.

La ligne `go` d'un module doit déclarer une version supérieure ou égale à
la version `go` déclarée par chacun des modules listés dans les instructions `require`.
La ligne `go` d'un workspace doit déclarer une version supérieure ou égale à
la version `go` déclarée par chacun des modules listés dans les instructions `use`.

Par exemple, si le module *M* requiert une dépendance *D* avec un `go.mod`
qui déclare `go 1.22.0`, alors le `go.mod` de *M* ne peut pas dire `go 1.21.3`.

La ligne `go` de chaque module définit la version de langage que le compiler
applique lors de la compilation des packages de ce module.
La version de langage peut être modifiée par fichier en utilisant une
[contrainte de build](/cmd/go#hdr-Build_constraints) :
si une contrainte de build est présente et implique une version minimale d'au moins `go1.21`,
la version de langage utilisée lors de la compilation de ce fichier sera cette version minimale.

Par exemple, un module contenant du code qui utilise la version de langage Go 1.21
doit avoir un fichier `go.mod` avec une ligne `go` comme `go 1.21` ou `go 1.21.3`.
Si un fichier source spécifique ne doit être compilé qu'avec un toolchain Go plus récent,
ajouter `//go:build go1.22` à ce fichier source garantit à la fois que seuls Go 1.22 et
les toolchains plus récents compileront le fichier, et change également la version de langage dans ce
fichier à Go 1.22.

Les lignes `go` et `toolchain` sont le plus facilement et sûrement modifiées
en utilisant `go get` ; voir la [section dédiée à `go get` ci-dessous](#get).

Avant Go 1.21, les toolchains Go traitaient la ligne `go` comme une exigence consultative :
si les builds réussissaient, le toolchain supposait que tout fonctionnait,
et sinon il affichait une note sur l'incompatibilité potentielle de version.
Go 1.21 a changé la ligne `go` pour en faire une exigence obligatoire.
Ce comportement est partiellement rétro-porté vers les versions de langage antérieures :
les publications Go 1.19 à partir de Go 1.19.13 et les publications Go 1.20 à partir de Go 1.20.8
refusent de charger des workspaces ou des modules déclarant la version Go 1.22 ou ultérieure.

Avant Go 1.21, les toolchains n'exigeaient pas qu'un module
ou un workspace ait une ligne `go` supérieure ou égale à la
version `go` requise par chacun de ses modules de dépendance.

## Le paramètre `GOTOOLCHAIN` {#GOTOOLCHAIN}

La commande `go` sélectionne le toolchain Go à utiliser en se basant sur le paramètre `GOTOOLCHAIN`.
Pour trouver le paramètre `GOTOOLCHAIN`, la commande `go` utilise les règles standard pour tout
paramètre d'environnement Go :

 - Si `GOTOOLCHAIN` est défini avec une valeur non vide dans l'environnement du processus
   (tel que renvoyé par [`os.Getenv`](/pkg/os/#Getenv)), la commande `go` utilise cette valeur.

 - Sinon, si `GOTOOLCHAIN` est défini dans le fichier de valeurs par défaut de l'environnement de l'utilisateur
   (géré avec
   [`go env -w` et `go env -u`](/cmd/go/#hdr-Print_Go_environment_information)),
   la commande `go` utilise cette valeur.

 - Sinon, si `GOTOOLCHAIN` est défini dans le fichier de valeurs par défaut d'environnement du toolchain Go intégré
   (`$GOROOT/go.env`), la commande `go` utilise cette valeur.

Dans les toolchains Go standard, le fichier `$GOROOT/go.env` définit `GOTOOLCHAIN=auto` par défaut,
mais les toolchains Go réempaquetés peuvent changer cette valeur.

Si le fichier `$GOROOT/go.env` est absent ou ne définit pas de valeur par défaut, la commande `go`
suppose `GOTOOLCHAIN=local`.

Exécuter `go env GOTOOLCHAIN` affiche le paramètre `GOTOOLCHAIN`.

## Sélection du toolchain Go {#select}

Au démarrage, la commande `go` sélectionne le toolchain Go à utiliser.
Elle consulte le paramètre `GOTOOLCHAIN`,
qui prend la forme `<name>`, `<name>+auto` ou `<name>+path`.
`GOTOOLCHAIN=auto` est un raccourci pour `GOTOOLCHAIN=local+auto` ;
de même, `GOTOOLCHAIN=path` est un raccourci pour `GOTOOLCHAIN=local+path`.
Le `<name>` définit le toolchain Go par défaut :
`local` indique le toolchain Go intégré
(celui fourni avec la commande `go` en cours d'exécution), et sinon `<name>` doit
être un nom de toolchain Go spécifique, tel que `go1.21.0`.
La commande `go` préfère exécuter le toolchain Go par défaut.
Comme indiqué ci-dessus, à partir de Go 1.21, les toolchains Go refusent de s'exécuter dans
des workspaces ou des modules qui requièrent des versions de Go plus récentes.
Au lieu de cela, ils rapportent une erreur et terminent.

Quand `GOTOOLCHAIN` est défini à `local`, la commande `go` exécute toujours le toolchain Go intégré.

Quand `GOTOOLCHAIN` est défini à `<name>` (par exemple, `GOTOOLCHAIN=go1.21.0`),
la commande `go` exécute toujours ce toolchain Go spécifique.
Si un binaire portant ce nom est trouvé dans le PATH système, la commande `go` l'utilise.
Sinon, la commande `go` utilise un toolchain Go qu'elle télécharge et vérifie.

Quand `GOTOOLCHAIN` est défini à `<name>+auto` ou `<name>+path` (ou les raccourcis `auto` ou `path`),
la commande `go` sélectionne et exécute une version de Go plus récente selon les besoins.
Plus précisément, elle consulte les lignes `toolchain` et `go` dans le fichier `go.work`
du workspace courant ou, en l'absence de workspace,
dans le fichier `go.mod` du module principal.
Si le fichier `go.work` ou `go.mod` a une ligne `toolchain <tname>`
et que `<tname>` est plus récent que le toolchain Go par défaut,
alors la commande `go` exécute `<tname>` à la place.
Si le fichier a une ligne `toolchain default`,
alors la commande `go` exécute le toolchain Go par défaut,
désactivant toute tentative de mise à jour au-delà de `<name>`.
Sinon, si le fichier a une ligne `go <version>`
et que `<version>` est plus récente que le toolchain Go par défaut,
alors la commande `go` exécute `go<version>` à la place.

Pour exécuter un toolchain autre que le toolchain Go intégré,
la commande `go` recherche dans le chemin d'exécution du processus
(`$PATH` sur Unix et Plan 9, `%PATH%` sur Windows)
un programme portant le nom donné (par exemple, `go1.21.3`) et l'exécute.
Si aucun tel programme n'est trouvé, la commande `go`
[télécharge et exécute le toolchain Go spécifié](#download).
Utiliser la forme `<name>+path` de `GOTOOLCHAIN` désactive le fallback par téléchargement,
amenant la commande `go` à s'arrêter après avoir cherché dans le chemin d'exécution.

Exécuter `go version` affiche la version du toolchain Go sélectionné
(en exécutant l'implémentation de `go version` du toolchain sélectionné).

Exécuter `GOTOOLCHAIN=local go version` affiche la version du toolchain Go intégré.

À partir de Go 1.24, tu peux tracer le processus de sélection de toolchain de la commande `go`
en ajoutant `toolchaintrace=1` à la variable d'environnement `GODEBUG` lors de l'exécution de la
commande `go`.

## Basculements de toolchain Go {#switch}

Pour la plupart des commandes, le `go.work` du workspace ou le `go.mod` du module principal
aura une ligne `go` au moins aussi récente que la ligne `go` de n'importe quelle dépendance de module,
en raison des [exigences de configuration](#config) d'ordonnancement des versions.
Dans ce cas, la sélection de toolchain au démarrage exécute un toolchain Go suffisamment récent
pour compléter la commande.

Certaines commandes intègrent de nouvelles versions de modules dans le cadre de leur fonctionnement :
`go get` ajoute de nouvelles dépendances de modules au module principal ;
`go work use` ajoute de nouveaux modules locaux au workspace ;
`go work sync` resynchronise un workspace avec des modules locaux qui ont pu être mis à jour
depuis la création du workspace ;
`go install package@version` et `go run package@version`
s'exécutent effectivement dans un module principal vide et ajoutent `package@version` comme nouvelle dépendance.
Toutes ces commandes peuvent rencontrer un module avec une ligne `go` dans `go.mod`
exigeant une version de Go plus récente que la version de Go actuellement exécutée.

Quand une commande rencontre un module exigeant une version de Go plus récente
et que `GOTOOLCHAIN` permet l'exécution de différents toolchains
(c'est l'une des formes `auto` ou `path`),
la commande `go` choisit et bascule vers un toolchain plus récent approprié
pour continuer à exécuter la commande courante.

À chaque fois que la commande `go` bascule vers un autre toolchain après la sélection initiale,
elle affiche un message expliquant pourquoi. Par exemple :

	go: module example.com/widget@v1.2.3 requires go >= 1.24rc1; switching to go 1.27.9

Comme le montre l'exemple, la commande `go` peut basculer vers un toolchain
plus récent que l'exigence découverte.
En général, la commande `go` vise à basculer vers un toolchain Go supporté.

Pour choisir le toolchain, la commande `go` obtient d'abord une liste des toolchains disponibles.
Pour la forme `auto`, la commande `go` télécharge une liste de toolchains disponibles.
Pour la forme `path`, la commande `go` parcourt le PATH à la recherche d'exécutables
nommés pour des toolchains valides et utilise une liste de tous les toolchains qu'elle trouve.
En utilisant cette liste de toolchains, la commande `go` identifie jusqu'à trois candidats :

 - la dernière release candidate d'une version de langage Go non publiée (1.*N*₃rc*R*₃),
 - le dernier patch release de la version de langage Go la plus récemment publiée (1.*N*₂.*P*₂), et
 - le dernier patch release de la version de langage Go précédente (1.*N*₁.*P*₁).

Ce sont les publications Go supportées conformément à la [politique de publication](/doc/devel/release#policy) de Go.
Conformément à la [sélection de version minimale](https://research.swtch.com/vgo-mvs),
la commande `go` utilise alors de façon conservatrice le candidat avec la version _minimale_ (la plus ancienne)
qui satisfait la nouvelle exigence.

Par exemple, supposons que `example.com/widget@v1.2.3` exige Go 1.24rc1 ou ultérieur.
La commande `go` obtient la liste des toolchains disponibles
et trouve que les derniers patch releases des deux toolchains Go les plus récents sont
Go 1.28.3 et Go 1.27.9,
et que la release candidate Go 1.29rc2 est également disponible.
Dans cette situation, la commande `go` choisira Go 1.27.9.
Si `widget` avait exigé Go 1.28 ou ultérieur, la commande `go` aurait choisi Go 1.28.3,
car Go 1.27.9 est trop ancien.
Si `widget` avait exigé Go 1.29 ou ultérieur, la commande `go` aurait choisi Go 1.29rc2,
car Go 1.27.9 et Go 1.28.3 sont tous deux trop anciens.

Les commandes qui intègrent de nouvelles versions de modules exigeant de nouvelles versions de Go
écrivent la nouvelle exigence de version `go` minimale dans le fichier `go.work` du workspace courant
ou dans le fichier `go.mod` du module principal, en mettant à jour la ligne `go`.
Pour la [reproductibilité](https://research.swtch.com/vgo-principles#repeatability),
toute commande qui met à jour la ligne `go` met également à jour la ligne `toolchain`
pour enregistrer son propre nom de toolchain.
La prochaine fois que la commande `go` s'exécute dans ce workspace ou module,
elle utilisera cette ligne `toolchain` mise à jour lors de la [sélection du toolchain](#select).

Par exemple, `go get example.com/widget@v1.2.3` peut afficher un message de basculement
comme ci-dessus et basculer vers Go 1.27.9.
Go 1.27.9 complétera le `go get` et mettra à jour la ligne `toolchain`
pour indiquer `toolchain go1.27.9`.
La prochaine commande `go` exécutée dans ce module ou workspace sélectionnera `go1.27.9`
au démarrage et n'affichera aucun message de basculement.

En général, si une commande `go` est exécutée deux fois et que la première affiche un message de basculement,
la seconde ne l'affichera pas, parce que la première a également mis à jour `go.work` ou `go.mod`
pour sélectionner le bon toolchain au démarrage.
L'exception concerne les formes `go install package@version` et `go run package@version`,
qui s'exécutent sans workspace ni module principal et ne peuvent pas écrire de ligne `toolchain`.
Elles affichent un message de basculement chaque fois qu'elles doivent basculer
vers un toolchain plus récent.

## Téléchargement de toolchains {#download}

Lors de l'utilisation de `GOTOOLCHAIN=auto` ou `GOTOOLCHAIN=<name>+auto`, la commande Go
télécharge les toolchains plus récents selon les besoins.
Ces toolchains sont empaquetés comme des modules spéciaux
avec le chemin de module `golang.org/toolchain`
et la version <code>v0.0.1-go<i>VERSION</i>.<i>GOOS</i>-<i>GOARCH</i></code>.
Les toolchains sont téléchargés comme n'importe quel autre module,
ce qui signifie que les téléchargements de toolchains peuvent être proxifiés en définissant `GOPROXY`
et que leurs checksums sont vérifiés par la base de données de checksums Go.
Parce que le toolchain spécifique utilisé dépend du toolchain par défaut propre au système
ainsi que du système d'exploitation et de l'architecture locaux (GOOS et GOARCH),
il n'est pas pratique d'écrire les checksums des modules de toolchain dans `go.sum`.
À la place, les téléchargements de toolchains échouent faute de vérification si `GOSUMDB=off`.
Les patterns `GOPRIVATE` et `GONOSUMDB` ne s'appliquent pas aux téléchargements de toolchains.

## Gestion des exigences de version de module Go avec `go get` {#get}

En général, la commande `go` traite les lignes `go` et `toolchain`
comme des déclarations de dépendances versionnées du toolchain pour le module principal.
La commande `go get` peut gérer ces lignes de la même façon qu'elle gère
les lignes `require` qui spécifient les dépendances versionnées envers les modules.

Par exemple, `go get go@1.22.1 toolchain@1.24rc1` modifie le fichier `go.mod`
du module principal pour indiquer `go 1.22.1` et `toolchain go1.24rc1`.

La commande `go` comprend que la dépendance `go` exige une dépendance `toolchain`
avec une version de Go supérieure ou égale.

En continuant l'exemple, un `go get go@1.25.0` ultérieur mettra également à jour
le toolchain à `go1.25.0`.
Quand le toolchain correspond exactement à la ligne `go`, il peut être
omis et implicite, donc ce `go get` supprimera la ligne `toolchain`.

La même exigence s'applique en sens inverse lors d'une rétrogradation :
si `go.mod` commence à `go 1.22.1` et `toolchain go1.24rc1`,
alors `go get toolchain@go1.22.9` ne mettra à jour que la ligne `toolchain`,
mais `go get toolchain@go1.21.3` rétrogradera également la ligne `go` à
`go 1.21.3`.
L'effet sera de laisser simplement `go 1.21.3` sans ligne `toolchain`.

La forme spéciale `toolchain@none` signifie supprimer toute ligne `toolchain`,
comme dans `go get toolchain@none` ou `go get go@1.25.0 toolchain@none`.

La commande `go` comprend la syntaxe de version pour
les dépendances `go` et `toolchain` ainsi que les requêtes.

Par exemple, tout comme `go get example.com/widget@v1.2` utilise
la dernière version `v1.2` de `example.com/widget` (peut-être `v1.2.3`),
`go get go@1.22` utilise la dernière publication disponible de la version de langage Go 1.22
(peut-être `1.22rc3`, ou peut-être `1.22.3`).
La même chose s'applique à `go get toolchain@go1.22`.

Les commandes `go get` et `go mod tidy` maintiennent la ligne `go`
supérieure ou égale à la ligne `go` de tout module de dépendance requis.

Par exemple, si le module principal a `go 1.22.1` et qu'on exécute
`go get example.com/widget@v1.2.3` qui déclare `go 1.24rc1`,
alors `go get` mettra à jour la ligne `go` du module principal à `go 1.24rc1`.

En continuant l'exemple, un `go get go@1.22.1` ultérieur
rétrogradera `example.com/widget` vers une version compatible avec Go 1.22.1
ou supprimera entièrement l'exigence,
tout comme il le ferait en rétrogradant n'importe quelle autre dépendance de `example.com/widget`.

Avant Go 1.21, la façon suggérée de mettre à jour un module vers une nouvelle version de Go (disons, Go 1.22)
était `go mod tidy -go=1.22`, pour s'assurer que tous les ajustements
spécifiques à Go 1.22 soient apportés au `go.mod` en même temps que la mise à jour de la
ligne `go`.
Cette forme est toujours valide, mais le plus simple `go get go@1.22` est maintenant préféré.

Quand `go get` est exécuté dans un module dans un répertoire contenu dans un workspace racine,
`go get` ignore essentiellement le workspace,
mais il met à jour le fichier `go.work` pour mettre à niveau la ligne `go`
quand le workspace aurait sinon une ligne `go` trop ancienne.

## Gestion des exigences de version du workspace Go avec `go work` {#work}

Comme noté dans la section précédente, `go get` exécuté dans un répertoire
à l'intérieur d'un workspace racine prendra soin de mettre à jour la ligne `go` du fichier `go.work`
selon les besoins pour qu'elle soit supérieure ou égale à celle de tout module à l'intérieur de ce racine.
Cependant, les workspaces peuvent également faire référence à des modules en dehors du répertoire racine ;
exécuter `go get` dans ces répertoires peut résulter en une configuration de workspace invalide,
dans laquelle la version `go` déclarée dans `go.work` est inférieure
à celle d'un ou plusieurs des modules dans les directives `use`.

La commande `go work use`, qui ajoute de nouvelles directives `use`, vérifie également
que la version `go` dans le fichier `go.work` est suffisamment récente pour toutes les
directives `use` existantes.
Pour mettre à jour un workspace dont la version `go` est désynchronisée
avec ses modules, exécute `go work use` sans arguments.

Les commandes `go work init` et `go work sync` mettent également à jour la version `go`
selon les besoins.

Pour supprimer la ligne `toolchain` d'un fichier `go.work`, utilise
`go work edit -toolchain=none`.
