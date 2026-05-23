---
title: Optimisation guidée par profil
layout: article
---

À partir de Go 1.20, le compiler Go prend en charge l'optimisation guidée par profil (Profile-Guided Optimization, PGO) pour optimiser davantage les builds.

Table des matières :

 [Vue d'ensemble](#overview)\
 [Collecte de profils](#collecting-profiles)\
 [Compilation avec PGO](#building)\
 [Notes](#notes)\
 [Foire Aux Questions](#faq)\
 [Annexe : sources de profils alternatives](#alternative-sources)

# Vue d'ensemble {#overview}

L'optimisation guidée par profil (Profile-Guided Optimization, PGO), également connue sous le nom de feedback-directed optimization (FDO), est une technique d'optimisation du compiler qui réinjecte des informations (un profil) provenant d'exécutions représentatives de l'application dans le compiler pour le prochain build de l'application, qui utilise ces informations pour prendre des décisions d'optimisation plus éclairées.
Par exemple, le compiler peut décider d'inliner de façon plus agressive les fonctions que le profil indique comme fréquemment appelées.

Dans Go, le compiler utilise des profils CPU pprof comme profil d'entrée, tels que ceux de [runtime/pprof](https://pkg.go.dev/runtime/pprof) ou [net/http/pprof](https://pkg.go.dev/net/http/pprof).

À partir de Go 1.22, les benchmarks pour un ensemble représentatif de programmes Go montrent que compiler avec PGO améliore les performances d'environ 2 à 14 %.
Nous nous attendons à ce que les gains de performance augmentent généralement avec le temps, à mesure que des optimisations supplémentaires tirent parti de PGO dans les futures versions de Go.


# Collecte de profils {#collecting-profiles}

Le compiler Go attend un profil CPU pprof comme entrée pour PGO.
Les profils générés par le runtime Go (comme ceux de [runtime/pprof](https://pkg.go.dev/runtime/pprof) et [net/http/pprof](https://pkg.go.dev/net/http/pprof)) peuvent être utilisés directement comme entrée du compiler.
Il peut également être possible d'utiliser/convertir des profils d'autres systèmes de profiling. Voir [l'annexe](#alternative-sources) pour des informations supplémentaires.

Pour de meilleurs résultats, il est important que les profils soient _représentatifs_ du comportement réel de l'application dans son environnement de production.
Utiliser un profil non représentatif risque de produire un binaire avec peu ou pas d'amélioration en production.
Ainsi, la collecte de profils directement depuis l'environnement de production est recommandée, et c'est la méthode principale pour laquelle le PGO de Go est conçu.

Le flux de travail typique est le suivant :

1. Compiler et publier un binaire initial (sans PGO).
2. Collecter des profils depuis la production.
3. Quand il est temps de publier un binaire mis à jour, compiler depuis le dernier code source et fournir le profil de production.
4. GOTO 2

Le PGO de Go est généralement robuste aux écarts entre la version du profil de l'application et la version en cours de compilation avec le profil, ainsi qu'à la compilation avec des profils collectés depuis des binaires déjà optimisés.
C'est ce qui rend ce cycle de vie itératif possible.
Voir la section [AutoFDO](#autofdo) pour des détails supplémentaires sur ce flux de travail.

S'il est difficile ou impossible de collecter depuis l'environnement de production (par exemple, un outil en ligne de commande distribué aux utilisateurs finaux), il est également possible de collecter depuis un benchmark représentatif.
Remarque : construire des benchmarks représentatifs est souvent assez difficile (comme l'est le fait de les maintenir représentatifs à mesure que l'application évolue).
En particulier, _les microbenchmarks sont généralement de mauvais candidats pour le profiling PGO_, car ils n'exercent qu'une petite partie de l'application, ce qui produit de petits gains quand on les applique à l'ensemble du programme.

# Compilation avec PGO {#building}

L'approche standard pour compiler consiste à stocker un profil CPU pprof avec le nom de fichier `default.pgo` dans le répertoire du package principal du binaire profilé.
Par défaut, `go build` détectera automatiquement les fichiers `default.pgo` et activera PGO.

Il est recommandé de committer les profils directement dans le dépôt source, car les profils sont une entrée du build importante pour des builds reproductibles (et performants !).
Les stocker avec le source simplifie l'expérience de build car il n'y a pas d'étapes supplémentaires pour obtenir le profil au-delà de récupérer le source.

Pour des scénarios plus complexes, le flag `go build -pgo` contrôle la sélection du profil PGO.
Ce flag vaut par défaut `-pgo=auto` pour le comportement `default.pgo` décrit ci-dessus.
Mettre le flag à `-pgo=off` désactive entièrement les optimisations PGO.

Si tu ne peux pas utiliser `default.pgo` (par exemple, différents profils pour différents scénarios d'un même binaire, impossible de stocker le profil avec le source, etc), tu peux passer directement un chemin vers le profil à utiliser (par exemple, `go build -pgo=/tmp/foo.pprof`).

_Remarque : Un chemin passé à `-pgo` s'applique à tous les packages principaux.
Par exemple, `go build -pgo=/tmp/foo.pprof ./cmd/foo ./cmd/bar` applique `foo.pprof` aux deux binaires `foo` et `bar`, ce qui n'est souvent pas ce que tu souhaites.
En général, les différents binaires doivent avoir des profils différents, passés via des invocations séparées de `go build`._

_Remarque : Avant Go 1.21, la valeur par défaut est `-pgo=off`. PGO doit être explicitement activé._

# Notes {#notes}

## Collecte de profils représentatifs depuis la production

Ton environnement de production est la meilleure source de profils représentatifs pour ton application, comme décrit dans [Collecte de profils](#collecting-profiles).

La façon la plus simple de commencer est d'ajouter [net/http/pprof](https://pkg.go.dev/net/http/pprof) à ton application puis de récupérer `/debug/pprof/profile?seconds=30` depuis une instance arbitraire de ton service.
C'est une excellente façon de démarrer, mais il y a des façons dont cela peut ne pas être représentatif :

* Cette instance peut ne rien faire au moment où elle est profilée, même si elle est habituellement occupée.

* Les patterns de trafic peuvent changer tout au long de la journée, faisant varier le comportement au cours de la journée.

* Les instances peuvent effectuer des opérations longues (par exemple, 5 minutes à faire l'opération A, puis 5 minutes à faire l'opération B, etc).
  Un profil de 30 s ne couvrira probablement qu'un seul type d'opération.

* Les instances peuvent ne pas recevoir une distribution équitable des requêtes (certaines instances reçoivent davantage d'un type de requête que d'autres).

Une stratégie plus robuste consiste à collecter plusieurs profils à différents moments depuis différentes instances pour limiter l'impact des différences entre les profils des instances individuelles.
Plusieurs profils peuvent ensuite être [fusionnés](#merging-profiles) en un seul profil pour utilisation avec PGO.

De nombreuses organisations exécutent des services de « continuous profiling » qui effectuent automatiquement ce type de profiling par échantillonnage à l'échelle de la flotte, qui pourrait ensuite être utilisé comme source de profils pour PGO.

## Fusion de profils {#merging-profiles}

L'outil pprof peut fusionner plusieurs profils comme ceci :

```
$ go tool pprof -proto a.pprof b.pprof > merged.pprof
```

Cette fusion est effectivement une somme directe des échantillons dans l'entrée, quelle que soit la durée en temps réel du profil.
En conséquence, lors du profiling d'une petite tranche de temps d'une application (par exemple, un serveur qui s'exécute indéfiniment), tu voudras probablement t'assurer que tous les profils ont la même durée en temps réel (c'est-à-dire que tous les profils sont collectés pendant 30 s).
Sinon, les profils avec une durée en temps réel plus longue seront surreprésentés dans le profil fusionné.

## AutoFDO {#autofdo}

Le PGO de Go est conçu pour prendre en charge un flux de travail de style « [AutoFDO](https://research.google/pubs/pub45290/) ».

Regardons de plus près le flux de travail décrit dans [Collecte de profils](#collecting-profiles) :

1. Compiler et publier un binaire initial (sans PGO).
2. Collecter des profils depuis la production.
3. Quand il est temps de publier un binaire mis à jour, compiler depuis le dernier code source et fournir le profil de production.
4. GOTO 2

Cela semble trompeusement simple, mais il y a quelques propriétés importantes à noter ici :

* Le développement est toujours en cours, donc le code source de la version profilée du binaire (étape 2) est probablement légèrement différent du dernier code source en cours de compilation (étape 3).
  Le PGO de Go est conçu pour être robuste à cela, ce que nous appelons _stabilité de la source_.

* C'est une boucle fermée.
  C'est-à-dire qu'après la première itération, la version profilée du binaire est déjà optimisée par PGO avec un profil d'une itération précédente.
  Le PGO de Go est également conçu pour être robuste à cela, ce que nous appelons _stabilité itérative_.

La _stabilité de la source_ est obtenue en utilisant des heuristiques pour faire correspondre les échantillons du profil au code source en cours de compilation.
En conséquence, de nombreuses modifications du code source, comme l'ajout de nouvelles fonctions, n'ont aucun impact sur la correspondance du code existant.
Quand le compiler n'est pas en mesure de faire correspondre le code modifié, certaines optimisations sont perdues, mais note que c'est une _dégradation progressive_.
Une seule fonction qui échoue à correspondre peut rater des opportunités d'optimisation, mais le bénéfice global de PGO est généralement réparti sur de nombreuses fonctions. Voir la section [stabilité de la source](#source-stability) pour plus de détails sur la correspondance et la dégradation.

La _stabilité itérative_ est la prévention des cycles de performance variable dans les builds PGO successifs (par exemple, le build #1 est rapide, le build #2 est lent, le build #3 est rapide, etc).
Nous utilisons des profils CPU pour identifier les fonctions chaudes à cibler avec des optimisations.
En théorie, une fonction chaude pourrait être tellement accélérée par PGO qu'elle n'apparaît plus comme chaude dans le profil suivant et ne reçoit pas d'optimisation, la rendant à nouveau lente.
Le compiler Go adopte une approche conservatrice des optimisations PGO, ce qui, selon nous, prévient une variance significative.
Si tu observes ce type d'instabilité, dépose une issue sur [go.dev/issue/new](/issue/new).

Ensemble, la stabilité de la source et la stabilité itérative éliminent l'exigence de builds en deux étapes où un premier build non optimisé est profilé comme canary, puis recompilé avec PGO pour la production (sauf si les performances maximales absolues sont requises).

## Stabilité de la source et refactoring {#source-stability}

Comme décrit ci-dessus, le PGO de Go fait un effort pour continuer à faire correspondre les échantillons des profils plus anciens au code source actuel.
Plus précisément, Go utilise les décalages de ligne dans les fonctions (par exemple, un appel sur la 5e ligne de la fonction foo).

De nombreux changements courants ne casseront pas la correspondance, notamment :

* Les modifications dans un fichier en dehors d'une fonction chaude (ajout/modification de code au-dessus ou en dessous de la fonction).

* Le déplacement d'une fonction vers un autre fichier dans le même package (le compiler ignore entièrement les noms de fichiers source).

Certains changements qui peuvent casser la correspondance :

* Les modifications à l'intérieur d'une fonction chaude (peuvent affecter les décalages de ligne).

* Le renommage de la fonction (et/ou du type pour les méthodes) (change le nom du symbole).

* Le déplacement de la fonction vers un autre package (change le nom du symbole).

Si le profil est relativement récent, les différences n'affectent probablement qu'un petit nombre de fonctions chaudes, limitant l'impact des optimisations manquées dans les fonctions qui échouent à correspondre.
Néanmoins, la dégradation s'accumulera lentement avec le temps car le code est rarement refactorisé _en retour_ à son ancienne forme, il est donc important de collecter régulièrement de nouveaux profils pour limiter l'écart de source par rapport à la production.

Une situation où la correspondance de profil peut se dégrader significativement est un refactoring à grande échelle qui renomme de nombreuses fonctions ou les déplace entre des packages.
Dans ce cas, tu peux subir une baisse de performance à court terme jusqu'à ce qu'un nouveau profil montre la nouvelle structure.

Pour les renommages mécaniques, un profil existant pourrait théoriquement être réécrit pour changer les anciens noms de symboles en nouveaux noms.
[github.com/google/pprof/profile](https://pkg.go.dev/github.com/google/pprof/profile) contient les primitives nécessaires pour réécrire un profil pprof de cette façon, mais au moment de l'écriture, aucun outil prêt à l'emploi n'existe pour cela.

## Performance du nouveau code

Lors de l'ajout de nouveau code ou de l'activation de nouveaux chemins de code avec un changement de flag, ce code ne sera pas présent dans le profil lors du premier build, et ne recevra donc pas les optimisations PGO jusqu'à ce qu'un nouveau profil reflétant le nouveau code soit collecté.
Garde cela à l'esprit lors de l'évaluation du déploiement de nouveau code : la première version ne représentera pas ses performances en régime permanent.

# Foire Aux Questions {#faq}

## Est-il possible d'optimiser les packages de la bibliothèque standard Go avec PGO ?

Oui.
PGO dans Go s'applique à l'ensemble du programme.
Tous les packages sont recompilés pour prendre en compte les optimisations potentielles guidées par profil, y compris les packages de la bibliothèque standard.

## Est-il possible d'optimiser les packages dans les modules dépendants avec PGO ?

Oui.
PGO dans Go s'applique à l'ensemble du programme.
Tous les packages sont recompilés pour prendre en compte les optimisations potentielles guidées par profil, y compris les packages dans les dépendances.
Cela signifie que la façon unique dont ton application utilise une dépendance impacte les optimisations appliquées à cette dépendance.

## PGO avec un profil non représentatif va-t-il rendre mon programme plus lent que sans PGO ?

Non, en principe.
Bien qu'un profil qui n'est pas représentatif du comportement en production entraîne des optimisations dans les parties froides de l'application, cela ne devrait pas ralentir les parties chaudes de l'application.
Si tu rencontres un programme où PGO donne de moins bonnes performances que sans PGO, dépose une issue sur [go.dev/issue/new](/issue/new).

## Puis-je utiliser le même profil pour différents builds GOOS/GOARCH ?

Oui.
Le format des profils est équivalent selon les configurations d'OS et d'architecture, ils peuvent donc être utilisés dans différentes configurations.
Par exemple, un profil collecté depuis un binaire linux/arm64 peut être utilisé dans un build windows/amd64.

Cela dit, les mises en garde de stabilité de la source discutées [ci-dessus](#autofdo) s'appliquent ici également.
Tout code source qui diffère selon ces configurations ne sera pas optimisé.
Pour la plupart des applications, la grande majorité du code est indépendante de la plateforme, donc la dégradation de cette forme est limitée.

Par exemple, les détails internes de la gestion des fichiers dans le package `os` diffèrent entre Linux et Windows.
Si ces fonctions sont chaudes dans le profil Linux, les équivalents Windows ne recevront pas les optimisations PGO car ils ne correspondent pas aux profils.

Tu peux fusionner des profils de différents builds GOOS/GOARCH. Voir la question suivante pour les compromis à considérer.

## Comment gérer un seul binaire utilisé pour différents types de charges de travail ?

Il n'y a pas de choix évident ici.
Un seul binaire utilisé pour différents types de charges de travail (par exemple, une base de données utilisée de façon intensive en lecture dans un service, et en écriture dans un autre service) peut avoir différents composants chauds, qui bénéficient de différentes optimisations.

Il y a trois options :

1. Compiler différentes versions du binaire pour chaque charge de travail : utiliser des profils de chaque charge de travail pour compiler plusieurs builds spécifiques à la charge de travail du binaire.
   Cela fournira les meilleures performances pour chaque charge de travail, mais peut ajouter de la complexité opérationnelle en ce qui concerne la gestion de plusieurs binaires et sources de profils.

2. Compiler un seul binaire en utilisant uniquement les profils de la charge de travail « la plus importante » : sélectionner la charge de travail « la plus importante » (la plus grande empreinte, la plus sensible aux performances), et compiler en utilisant uniquement les profils de cette charge de travail.
   Cela fournit les meilleures performances pour la charge de travail sélectionnée, et probablement encore de modestes améliorations de performances pour les autres charges de travail grâce aux optimisations du code commun partagé entre les charges de travail.

3. Fusionner les profils de toutes les charges de travail : prendre des profils de chaque charge de travail (pondérés par l'empreinte totale) et les fusionner en un seul profil « à l'échelle de la flotte » utilisé pour compiler un seul profil commun.
   Cela fournit probablement de modestes améliorations de performances pour toutes les charges de travail.

## Comment PGO affecte-t-il le temps de compilation ?

L'activation des builds PGO entraînera probablement des augmentations mesurables des temps de compilation des packages.
Le composant le plus notable est que les profils PGO s'appliquent à tous les packages d'un binaire, ce qui signifie que la première utilisation d'un profil nécessite une recompilation de chaque package dans le graphe de dépendances.
Ces builds sont mis en cache comme n'importe quel autre, donc les builds incrémentiels ultérieurs utilisant le même profil ne nécessitent pas de recompilations complètes.

Si tu constates des augmentations extrêmes du temps de compilation, dépose une issue sur [go.dev/issue/new](/issue/new).

## Comment PGO affecte-t-il la taille du binaire ?

PGO peut entraîner des binaires légèrement plus grands en raison de l'inlining supplémentaire de fonctions.

# Annexe : sources de profils alternatives {#alternative-sources}

Les profils CPU générés par le runtime Go (via [runtime/pprof](https://pkg.go.dev/runtime/pprof), etc) sont déjà dans le format correct pour une utilisation directe comme entrées PGO.
Cependant, les organisations peuvent avoir d'autres outils préférés (par exemple, Linux perf), ou des systèmes de continuous profiling à l'échelle de la flotte existants qu'elles souhaitent utiliser avec le PGO de Go.

Les profils provenant de sources alternatives peuvent être utilisés avec le PGO de Go s'ils sont convertis au [format pprof](https://github.com/google/pprof/tree/main/proto), à condition qu'ils respectent ces exigences générales :

* L'un des indices d'échantillon doit avoir le type/unité « samples »/« count » ou « cpu »/« nanoseconds ».

* Les échantillons doivent représenter des échantillons du temps CPU à l'emplacement d'échantillonnage.

* Le profil doit être symbolisé ([Function.name](https://github.com/google/pprof/blob/76d1ae5aea2b3f738f2058d17533b747a1a5cd01/proto/profile.proto#L208) doit être défini).

* Les échantillons doivent contenir des stack frames pour les fonctions inlinées.
  Si les fonctions inlinées sont omises, Go ne pourra pas maintenir la stabilité itérative.

* [Function.start_line](https://github.com/google/pprof/blob/76d1ae5aea2b3f738f2058d17533b747a1a5cd01/proto/profile.proto#L215) doit être défini.
  C'est le numéro de ligne du début de la fonction.
  C'est-à-dire la ligne contenant le mot-clé `func`.
  Le compiler Go utilise ce champ pour calculer les décalages de ligne des échantillons (`Location.Line.line - Function.start_line`).
  **Remarque : de nombreux convertisseurs pprof existants omettent ce champ.**

_Remarque : Avant Go 1.21, les métadonnées DWARF omettent les lignes de début de fonction (`DW_AT_decl_line`), ce qui peut rendre difficile pour les outils de déterminer la ligne de début._

Voir la page [PGO Tools](/wiki/PGO-Tools) sur le wiki Go pour des informations supplémentaires sur la compatibilité PGO d'outils tiers spécifiques.
