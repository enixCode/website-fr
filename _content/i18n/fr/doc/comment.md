---
title: "Commentaires de doc Go"
layout: article
date: 2022-06-01T00:00:00Z
template: true
---

Table des matières :

 [Packages](#package)\
 [Commandes](#cmd)\
 [Types](#type)\
 [Fonctions](#func)\
 [Constantes](#const)\
 [Variables](#var)\
 [Syntaxe](#syntax)\
 [Erreurs fréquentes et pièges](#mistakes)

Les "commentaires de doc" sont des commentaires qui apparaissent immédiatement avant les déclarations de package, const, func, type et var de premier niveau, sans ligne vide intercalée.
Tout nom exporté (avec majuscule) devrait avoir un commentaire de doc.

Les packages [go/doc](/pkg/go/doc) et [go/doc/comment](/pkg/go/doc/comment)
permettent d'extraire la documentation depuis le code source Go,
et de nombreux outils exploitent cette fonctionnalité.
La commande [`go` `doc`](/cmd/go#hdr-Show_documentation_for_package_or_symbol)
recherche et affiche le commentaire de doc d'un package ou d'un symbole donné.
(Un symbole est une const, func, type ou var de premier niveau.)
Le serveur web [pkg.go.dev](https://pkg.go.dev/) affiche la documentation
des packages Go publics (lorsque leurs licences le permettent).
Le programme qui sert ce site est
[golang.org/x/pkgsite/cmd/pkgsite](https://pkg.go.dev/golang.org/x/pkgsite/cmd/pkgsite),
qui peut aussi être exécuté localement pour consulter la documentation de modules privés
ou sans connexion internet.
Le serveur de langage [gopls](https://pkg.go.dev/golang.org/x/tools/gopls)
fournit la documentation lors de l'édition de fichiers source Go dans les IDEs.

Le reste de cette page explique comment écrire des commentaires de doc Go.

## Packages {#package}

Chaque package devrait avoir un commentaire de package qui le présente.
Il fournit des informations pertinentes pour le package dans son ensemble
et pose généralement les attentes vis-à-vis du package.
En particulier dans les grands packages, il peut être utile que le commentaire de package
donne une vue d'ensemble des parties les plus importantes de l'API,
avec des liens vers d'autres commentaires de doc si nécessaire.

Si le package est simple, le commentaire de package peut être bref.
Par exemple :

	// Package path implements utility routines for manipulating slash-separated
	// paths.
	//
	// The path package should only be used for paths separated by forward
	// slashes, such as the paths in URLs. This package does not deal with
	// Windows paths with drive letters or backslashes; to manipulate
	// operating system paths, use the [path/filepath] package.
	package path

Les crochets dans `[path/filepath]` créent un [lien de documentation](#links).

Comme on peut le voir dans cet exemple, les commentaires de doc Go utilisent des phrases complètes.
Pour un commentaire de package, cela signifie que la [première phrase](/pkg/go/doc/#Package.Synopsis)
commence par "Package <nom>".

Pour les packages composés de plusieurs fichiers, le commentaire de package ne doit figurer que dans un seul fichier source.
Si plusieurs fichiers contiennent des commentaires de package, ils sont concaténés pour former un
grand commentaire unique pour l'ensemble du package.

## Commandes {#cmd}

Le commentaire de package d'une commande est similaire, mais il décrit le comportement
du programme plutôt que les symboles Go du package.
La première phrase commence conventionnellement par le nom du programme lui-même,
avec une majuscule puisqu'il est en début de phrase.
Par exemple, voici une version abrégée du commentaire de package de [gofmt](/cmd/gofmt) :

	/*
	Gofmt formats Go programs.
	It uses tabs for indentation and blanks for alignment.
	Alignment assumes that an editor is using a fixed-width font.

	Without an explicit path, it processes the standard input. Given a file,
	it operates on that file; given a directory, it operates on all .go files in
	that directory, recursively. (Files starting with a period are ignored.)
	By default, gofmt prints the reformatted sources to standard output.

	Usage:

		gofmt [flags] [path ...]

	The flags are:

		-d
			Do not print reformatted sources to standard output.
			If a file's formatting is different than gofmt's, print diffs
			to standard output.
		-w
			Do not print reformatted sources to standard output.
			If a file's formatting is different from gofmt's, overwrite it
			with gofmt's version. If an error occurred during overwriting,
			the original file is restored from an automatic backup.

	When gofmt reads from standard input, it accepts either a full Go program
	or a program fragment. A program fragment must be a syntactically
	valid declaration list, statement list, or expression. When formatting
	such a fragment, gofmt preserves leading indentation as well as leading
	and trailing spaces, so that individual sections of a Go program can be
	formatted by piping them through gofmt.
	*/
	package main

Le début du commentaire est rédigé avec les
[semantic linefeeds](https://rhodesmill.org/brandon/2012/one-sentence-per-line/),
où chaque nouvelle phrase ou longue expression occupe sa propre ligne,
ce qui peut rendre les diffs plus lisibles à mesure que le code et les commentaires évoluent.
Les paragraphes suivants ne respectent pas cette convention
et ont été mis en forme manuellement.
Ce qui convient le mieux à ta base de code est parfaitement valable.
Dans tous les cas, `go` `doc` et `pkgsite` reformatent le texte des commentaires de doc lors de l'affichage.
Par exemple :

	$ go doc gofmt
	Gofmt formats Go programs. It uses tabs for indentation and blanks for
	alignment. Alignment assumes that an editor is using a fixed-width font.

	Without an explicit path, it processes the standard input. Given a file, it
	operates on that file; given a directory, it operates on all .go files in that
	directory, recursively. (Files starting with a period are ignored.) By default,
	gofmt prints the reformatted sources to standard output.

	Usage:

		gofmt [flags] [path ...]

	The flags are:

		-d
			Do not print reformatted sources to standard output.
			If a file's formatting is different than gofmt's, print diffs
			to standard output.
	...

Les lignes indentées sont traitées comme du texte préformaté :
elles ne sont pas reformatées et sont affichées en police monospace
dans les rendus HTML et Markdown.
(La section [Syntaxe](#syntax) ci-dessous donne les détails.)

## Types {#type}

Le commentaire de doc d'un type devrait expliquer ce que chaque instance de ce type représente ou fournit.
Si l'API est simple, le commentaire de doc peut être très bref.
Par exemple :

	package zip

	// A Reader serves content from a ZIP archive.
	type Reader struct {
		...
	}

Par défaut, les programmeurs devraient supposer qu'un type n'est sûr à utiliser
que par une seule goroutine à la fois.
Si un type offre des garanties plus fortes, le commentaire de doc devrait les mentionner.
Par exemple :

	package regexp

	// Regexp is the representation of a compiled regular expression.
	// A Regexp is safe for concurrent use by multiple goroutines,
	// except for configuration methods, such as Longest.
	type Regexp struct {
		...
	}

Les types Go devraient aussi s'efforcer de donner un sens utile à la valeur zéro.
Si ce sens n'est pas évident, il devrait être documenté. Par exemple :

	package bytes

	// A Buffer is a variable-sized buffer of bytes with Read and Write methods.
	// The zero value for Buffer is an empty buffer ready to use.
	type Buffer struct {
		...
	}

Pour une struct avec des champs exportés, le commentaire de doc ou les commentaires de champ
devraient expliquer la signification de chaque champ exporté.
Par exemple, le commentaire de doc de ce type explique les champs :

{{raw `
	package io

	// A LimitedReader reads from R but limits the amount of
	// data returned to just N bytes. Each call to Read
	// updates N to reflect the new amount remaining.
	// Read returns EOF when N <= 0.
	type LimitedReader struct {
		R   Reader // underlying reader
		N   int64  // max bytes remaining
	}
`}}

En revanche, le commentaire de doc de ce type laisse les explications aux commentaires de champ :

{{raw `
	package comment

	// A Printer is a doc comment printer.
	// The fields in the struct can be filled in before calling
	// any of the printing methods
	// in order to customize the details of the printing process.
	type Printer struct {
		// HeadingLevel is the nesting level used for
		// HTML and Markdown headings.
		// If HeadingLevel is zero, it defaults to level 3,
		// meaning to use <h3> and ###.
		HeadingLevel int
		...
	}
`}}

Comme pour les packages (ci-dessus) et les fonctions (ci-dessous), les commentaires de doc des types
commencent par des phrases complètes nommant le symbole déclaré.
Un sujet explicite rend souvent la formulation plus claire,
et facilite la recherche dans le texte, que ce soit sur une page web
ou en ligne de commande.
Par exemple :

	$ go doc -all regexp | grep pairs
	pairs within the input string: result[2*n:2*n+2] identifies the indexes
	    FindReaderSubmatchIndex returns a slice holding the index pairs identifying
	    FindStringSubmatchIndex returns a slice holding the index pairs identifying
	    FindSubmatchIndex returns a slice holding the index pairs identifying the
	$

## Fonctions {#func}

Le commentaire de doc d'une fonction devrait expliquer ce que la fonction retourne,
ou, pour les fonctions appelées pour leurs effets de bord, ce qu'elle fait.
Les paramètres et résultats nommés peuvent être cités directement dans
le commentaire, sans syntaxe particulière comme des guillemets obliques.
(Une conséquence de cette convention est que les noms comme `a`,
qui pourraient être confondus avec des mots ordinaires, sont généralement évités.)
Par exemple :

	package strconv

	// Quote returns a double-quoted Go string literal representing s.
	// The returned string uses Go escape sequences (\t, \n, \xFF, Ā)
	// for control characters and non-printable characters as defined by IsPrint.
	func Quote(s string) string {
		...
	}

Et :

	package os

	// Exit causes the current program to exit with the given status code.
	// Conventionally, code zero indicates success, non-zero an error.
	// The program terminates immediately; deferred functions are not run.
	//
	// For portability, the status code should be in the range [0, 125].
	func Exit(code int) {
		...
	}

Les commentaires de doc utilisent typiquement la formule `reports whether`
pour décrire les fonctions qui retournent un booléen.
La formule `or not` est inutile.
Par exemple :

	package strings

	// HasPrefix reports whether the string s begins with prefix.
	func HasPrefix(s, prefix string) bool

Si un commentaire de doc doit expliquer plusieurs résultats,
nommer les résultats peut rendre le commentaire de doc plus compréhensible,
même si les noms ne sont pas utilisés dans le corps de la fonction.
Par exemple :

	package io

	// Copy copies from src to dst until either EOF is reached
	// on src or an error occurs. It returns the total number of bytes
	// written and the first error encountered while copying, if any.
	//
	// A successful Copy returns err == nil, not err == EOF.
	// Because Copy is defined to read from src until EOF, it does
	// not treat an EOF from Read as an error to be reported.
	func Copy(dst Writer, src Reader) (n int64, err error) {
		...
	}

Inversement, quand les résultats n'ont pas besoin d'être nommés dans le commentaire de doc,
ils sont généralement omis dans le code également, comme dans l'exemple `Quote` ci-dessus,
pour ne pas alourdir la présentation.

Ces règles s'appliquent aussi bien aux fonctions simples qu'aux méthodes.
Pour les méthodes, utiliser le même nom de receiver évite les variations inutiles
lors de la liste de toutes les méthodes d'un type :

	$ go doc bytes.Buffer
	package bytes // import "bytes"

	type Buffer struct {
		// Has unexported fields.
	}
	    A Buffer is a variable-sized buffer of bytes with Read and Write methods.
	    The zero value for Buffer is an empty buffer ready to use.

	func NewBuffer(buf []byte) *Buffer
	func NewBufferString(s string) *Buffer
	func (b *Buffer) Bytes() []byte
	func (b *Buffer) Cap() int
	func (b *Buffer) Grow(n int)
	func (b *Buffer) Len() int
	func (b *Buffer) Next(n int) []byte
	func (b *Buffer) Read(p []byte) (n int, err error)
	func (b *Buffer) ReadByte() (byte, error)
	...

Cet exemple montre aussi que les fonctions de premier niveau retournant un type `T` ou un pointer `*T`,
éventuellement avec un résultat d'erreur supplémentaire,
sont affichées avec le type `T` et ses méthodes,
sous l'hypothèse qu'elles sont les constructeurs de `T`.

Par défaut, les programmeurs peuvent supposer qu'une fonction de premier niveau
est sûre à appeler depuis plusieurs goroutines simultanément ;
ce fait n'a pas besoin d'être mentionné explicitement.

En revanche, comme indiqué dans la section précédente,
utiliser une instance d'un type de quelque manière que ce soit,
y compris en appelant une méthode, est typiquement supposé
être limité à une seule goroutine à la fois.
Si les méthodes sûres pour un usage concurrent
ne sont pas documentées dans le commentaire de doc du type,
elles devraient l'être dans les commentaires par méthode.
Par exemple :

	package sql

	// Close returns the connection to the connection pool.
	// All operations after a Close will return with ErrConnDone.
	// Close is safe to call concurrently with other operations and will
	// block until all other operations finish. It may be useful to first
	// cancel any used context and then call Close directly after.
	func (c *Conn) Close() error {
		...
	}

Note que les commentaires de doc de fonctions et méthodes se concentrent sur
ce que l'opération retourne ou fait,
en détaillant ce que l'appelant doit savoir.
Les cas particuliers peuvent être particulièrement importants à documenter.
Par exemple :

{{raw `
	package math

	// Sqrt returns the square root of x.
	//
	// Special cases are:
	//
	//	Sqrt(+Inf) = +Inf
	//	Sqrt(±0) = ±0
	//	Sqrt(x < 0) = NaN
	//	Sqrt(NaN) = NaN
	func Sqrt(x float64) float64 {
		...
	}
`}}

Les commentaires de doc ne devraient pas expliquer les détails internes
comme l'algorithme utilisé dans l'implémentation actuelle.
Ces informations sont mieux placées dans des commentaires à l'intérieur du corps de la fonction.
Il peut être approprié d'indiquer des bornes de complexité temporelle ou spatiale asymptotique
lorsque ce détail est particulièrement important pour les appelants.
Par exemple :

	package sort

	// Sort sorts data in ascending order as determined by the Less method.
	// It makes one call to data.Len to determine n and O(n*log(n)) calls to
	// data.Less and data.Swap. The sort is not guaranteed to be stable.
	func Sort(data Interface) {
		...
	}

Parce que ce commentaire de doc ne mentionne pas quel algorithme de tri est utilisé,
il est plus facile de modifier l'implémentation pour utiliser un algorithme différent à l'avenir.

## Constantes {#const}

La syntaxe de déclaration Go permet de regrouper des déclarations,
auquel cas un seul commentaire de doc peut introduire un groupe de constantes liées,
les constantes individuelles n'étant documentées que par de courts commentaires en fin de ligne.
Par exemple :

	package scanner // import "text/scanner"

	// The result of Scan is one of these tokens or a Unicode character.
	const (
		EOF = -(iota + 1)
		Ident
		Int
		Float
		Char
		...
	)

Parfois le groupe n'a besoin d'aucun commentaire de doc. Par exemple :

	package unicode // import "unicode"

	const (
		MaxRune         = '\U0010FFFF' // maximum valid Unicode code point.
		ReplacementChar = '\uFFFD'     // represents invalid code points.
		MaxASCII        = '\u007F'     // maximum ASCII value.
		MaxLatin1       = '\u00FF'     // maximum Latin-1 value.
	)

En revanche, les constantes non regroupées méritent généralement un commentaire de doc complet
commençant par une phrase complète. Par exemple :

	package unicode

	// Version is the Unicode edition from which the tables are derived.
	const Version = "13.0.0"

Les constantes typées sont affichées à côté de la déclaration de leur type
et omettent donc souvent un commentaire de doc de groupe de constantes au profit
du commentaire de doc du type.
Par exemple :

	package syntax

	// An Op is a single regular expression operator.
	type Op uint8

	const (
		OpNoMatch        Op = 1 + iota // matches no strings
		OpEmptyMatch                   // matches empty string
		OpLiteral                      // matches Runes sequence
		OpCharClass                    // matches Runes interpreted as range pair list
		OpAnyCharNotNL                 // matches any character except newline
		...
	)

(Voir [pkg.go.dev/regexp/syntax#Op](https://pkg.go.dev/regexp/syntax#Op) pour la présentation HTML.)

## Variables {#var}

Les conventions pour les variables sont les mêmes que pour les constantes.
Par exemple, voici un ensemble de variables regroupées :

	package fs

	// Generic file system errors.
	// Errors returned by file systems can be tested against these errors
	// using errors.Is.
	var (
		ErrInvalid    = errInvalid()    // "invalid argument"
		ErrPermission = errPermission() // "permission denied"
		ErrExist      = errExist()      // "file already exists"
		ErrNotExist   = errNotExist()   // "file does not exist"
		ErrClosed     = errClosed()     // "file already closed"
	)

Et une variable seule :

	package unicode

	// Scripts is the set of Unicode script tables.
	var Scripts = map[string]*RangeTable{
		"Adlam":                  Adlam,
		"Ahom":                   Ahom,
		"Anatolian_Hieroglyphs":  Anatolian_Hieroglyphs,
		"Arabic":                 Arabic,
		"Armenian":               Armenian,
		...
	}

## Syntaxe {#syntax}

Les commentaires de doc Go sont écrits dans une syntaxe simple qui supporte
les paragraphes, les titres, les liens, les listes et les blocs de code préformatés.
Pour garder les commentaires légers et lisibles dans les fichiers source,
il n'y a pas de support pour des fonctionnalités complexes comme les changements de police ou le HTML brut.
Les amateurs de Markdown peuvent voir cette syntaxe comme un sous-ensemble simplifié de Markdown.

Le formateur standard [gofmt](/cmd/gofmt) reformate les commentaires de doc
pour utiliser un formatage canonique pour chacune de ces fonctionnalités.
Gofmt vise la lisibilité et laisse à l'utilisateur le contrôle sur la façon dont les commentaires
sont rédigés dans le code source, mais ajuste la présentation pour rendre
la signification sémantique d'un commentaire particulier plus claire,
par analogie avec le reformatage de `1+2 * 3` en `1 + 2*3` dans du code ordinaire.

Gofmt supprime les lignes vides en début et en fin de commentaires de doc.
Si toutes les lignes d'un commentaire de doc commencent par la même séquence
d'espaces et de tabulations, gofmt supprime ce préfixe.

### Paragraphes {#paragraphs}

Un paragraphe est une suite de lignes non vides non indentées.
Nous avons déjà vu de nombreux exemples de paragraphes.

Une paire de backticks consécutifs (\` U+0060)
est interprétée comme un guillemet gauche Unicode (" U+201C),
et une paire de guillemets simples consécutifs (\' U+0027)
est interprétée comme un guillemet droit Unicode (" U+201D).

Gofmt préserve les sauts de ligne dans le texte des paragraphes : il ne reforte pas le texte.
Cela permet l'utilisation des [semantic linefeeds](https://rhodesmill.org/brandon/2012/one-sentence-per-line/),
comme vu précédemment.
Gofmt remplace les lignes vides consécutives entre les paragraphes
par une seule ligne vide.
Gofmt reformate aussi les backticks ou guillemets simples consécutifs
en leurs interprétations Unicode.

#### Notes {#notes}

Les notes sont des commentaires spéciaux de la forme `MARKER(uid): body`.
MARKER doit être composé de 2 lettres majuscules ou plus `[A-Z]`,
identifiant le type de note, tandis que uid est au moins 1 caractère,
généralement le nom d'utilisateur de quelqu'un pouvant fournir plus d'informations.
Le `:` qui suit l'uid est optionnel.

Les notes sont collectées et affichées dans leur propre section sur pkg.go.dev.

Par exemple :

	// TODO(user1): refactor to use standard library context
	// BUG(user2): not cleaned up
	var ctx context.Context

#### Dépréciations {#deprecations}

Les paragraphes commençant par `Deprecated: ` sont traités comme des avis de dépréciation.
Certains outils avertiront lorsque des identifiants dépréciés sont utilisés.
[pkg.go.dev](https://pkg.go.dev) masquera leur documentation par défaut.

Les avis de dépréciation sont suivis d'informations sur la dépréciation,
et d'une recommandation sur ce qu'il faut utiliser à la place, le cas échéant.
Le paragraphe n'a pas besoin d'être le dernier paragraphe du commentaire de doc.

Par exemple :

	// Package rc4 implements the RC4 stream cipher.
	//
	// Deprecated: RC4 is cryptographically broken and should not be used
	// except for compatibility with legacy systems.
	//
	// This package is frozen and no new functionality will be added.
	package rc4

	// Reset zeros the key data and makes the Cipher unusable.
	//
	// Deprecated: Reset can't guarantee that the key will be entirely removed from
	// the process's memory.
	func (c *Cipher) Reset()

### Titres {#headings}

Un titre est une ligne commençant par un dièse (U+0023), puis un espace et le texte du titre.
Pour être reconnu comme un titre, la ligne doit être non indentée et séparée du texte de paragraphe adjacent
par des lignes vides.

Par exemple :

	// Package strconv implements conversions to and from string representations
	// of basic data types.
	//
	// # Numeric Conversions
	//
	// The most common numeric conversions are [Atoi] (string to int) and [Itoa] (int to string).
	...
	package strconv

En revanche :

	// #This is not a heading, because there is no space.
	//
	// # This is not a heading,
	// # because it is multiple lines.
	//
	// # This is not a heading,
	// because it is also multiple lines.
	//
	// The next paragraph is not a heading, because there is no additional text:
	//
	// #
	//
	// In the middle of a span of non-blank lines,
	// # this is not a heading either.
	//
	//     # This is not a heading, because it is indented.

La syntaxe `#` a été ajoutée dans Go 1.19.
Avant Go 1.19, les titres étaient identifiés implicitement par des paragraphes d'une seule ligne
satisfaisant certaines conditions, notamment l'absence de ponctuation finale.

Gofmt reformate les [lignes traitées comme des titres implicites](https://github.com/golang/proposal/blob/master/design/51082-godocfmt.md#headings)
par les versions antérieures de Go pour utiliser des titres `#` à la place.
Si le reformatage n'est pas approprié, c'est-à-dire si la ligne n'était pas censée être un titre, la façon la plus simple
de la transformer en paragraphe est d'introduire une ponctuation finale
comme un point ou deux-points, ou de la diviser en deux lignes.

### Liens {#links}

Une suite de lignes non vides non indentées définit des cibles de liens
lorsque chaque ligne est de la forme "[Texte]: URL".
Dans le reste du même commentaire de doc,
"[Texte]" représente un lien vers l'URL avec le texte donné, soit en HTML :
\<a href="URL">Texte\</a>.
Par exemple :

	// Package json implements encoding and decoding of JSON as defined in
	// [RFC 7159]. The mapping between JSON and Go values is described
	// in the documentation for the Marshal and Unmarshal functions.
	//
	// For an introduction to this package, see the article
	// "[JSON and Go]."
	//
	// [RFC 7159]: https://tools.ietf.org/html/rfc7159
	// [JSON and Go]: https://golang.org/doc/articles/json_and_go.html
	package json

En plaçant les URLs dans une section séparée,
ce format n'interrompt que minimalement le flux du texte principal.
Il correspond aussi approximativement au format Markdown
[shortcut reference link](https://spec.commonmark.org/0.30/#shortcut-reference-link),
sans le texte de titre optionnel.

S'il n'y a pas de déclaration d'URL correspondante,
alors (sauf pour les liens de doc, décrits dans la section suivante)
"[Texte]" n'est pas un hyperlien, et les crochets sont préservés
lors de l'affichage.
Chaque commentaire de doc est considéré indépendamment :
les définitions de cibles de liens dans un commentaire n'affectent pas les autres commentaires.

Bien que les blocs de définition de cibles de liens puissent être intercalés avec
des paragraphes ordinaires, gofmt déplace toutes les définitions de cibles de liens vers
la fin du commentaire de doc,
en deux blocs au maximum : d'abord un bloc contenant toutes les cibles de liens
référencées dans le commentaire, puis un bloc
contenant toutes les cibles _non_ référencées dans le commentaire.
Le bloc séparé rend les cibles inutilisées faciles
à repérer et corriger (au cas où les liens ou les définitions contiendraient des fautes de frappe)
ou à supprimer (au cas où les définitions ne sont plus nécessaires).

Le texte brut reconnu comme une URL est automatiquement transformé en lien dans les rendus HTML.

### Liens de doc {#doclinks}

Les liens de doc sont des liens de la forme "[Nom1]" ou "[Nom1.Nom2]" pour référencer
des identifiants exportés dans le package courant, ou "[pkg]",
"[pkg.Nom1]", ou "[pkg.Nom1.Nom2]" pour référencer des identifiants dans d'autres packages.

Par exemple :

	package bytes

	// ReadFrom reads data from r until EOF and appends it to the buffer, growing
	// the buffer as needed. The return value n is the number of bytes read. Any
	// error except [io.EOF] encountered during the read is also returned. If the
	// buffer becomes too large, ReadFrom will panic with [ErrTooLarge].
	func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error) {
		...
	}

Le texte entre crochets d'un lien symbolique
peut inclure une étoile initiale optionnelle, facilitant la référence aux
types pointer, comme \[\*bytes.Buffer\].

Lors de la référence à d'autres packages, "pkg" peut être soit un chemin d'import complet,
soit le nom de package supposé d'un import existant. Le nom de package supposé
est soit l'identifiant d'un import renommé, soit
[le nom supposé par
goimports](https://pkg.go.dev/golang.org/x/tools/internal/imports#ImportPathToAssumedName).
(Goimports insère des renommages lorsque cette hypothèse est incorrecte, donc
cette règle devrait fonctionner pour pratiquement tout code Go.)
Par exemple, si le package courant importe encoding/json,
alors "[json.Decoder]" peut être écrit à la place de "[encoding/json.Decoder]"
pour lier vers la documentation de Decoder dans encoding/json.
Si différents fichiers source d'un package importent différents packages en utilisant le même nom,
alors le raccourci est ambigu et ne peut pas être utilisé.

Un "pkg" est supposé être un chemin d'import complet uniquement
s'il commence par un nom de domaine (un élément de chemin avec un point) ou s'il fait partie
des packages de la bibliothèque standard ("[os]", "[encoding/json]", etc.).
Par exemple, `[os.File]` et `[example.com/sys.File]` sont des liens de documentation
(ce dernier sera un lien cassé),
mais `[os/sys.File]` n'en est pas un, car il n'existe pas de package os/sys dans la bibliothèque standard.

Pour éviter les problèmes avec
les maps, les génériques et les types de tableaux, les liens de doc doivent être précédés et
suivis de ponctuation, espaces, tabulations, ou du début ou de la fin d'une ligne.
Par exemple, le texte "map[ast.Expr]TypeAndValue" ne contient pas
de lien de doc.

### Listes {#lists}

Une liste est une suite de lignes indentées ou vides
(qui serait autrement un bloc de code,
comme décrit dans la section suivante)
dont la première ligne indentée commence par
un marqueur de liste à puces ou un marqueur de liste numérotée.

Un marqueur de liste à puces est une étoile, un plus, un tiret ou une puce Unicode
(*, +, -, •; U+002A, U+002B, U+002D, U+2022)
suivi d'un espace ou d'une tabulation, puis du texte.
Dans une liste à puces, chaque ligne commençant par un marqueur de liste à puces
commence un nouvel élément de liste.

Par exemple :

	package url

	// PublicSuffixList provides the public suffix of a domain. For example:
	//   - the public suffix of "example.com" is "com",
	//   - the public suffix of "foo1.foo2.foo3.co.uk" is "co.uk", and
	//   - the public suffix of "bar.pvt.k12.ma.us" is "pvt.k12.ma.us".
	//
	// Implementations of PublicSuffixList must be safe for concurrent use by
	// multiple goroutines.
	//
	// An implementation that always returns "" is valid and may be useful for
	// testing but it is not secure: it means that the HTTP server for foo.com can
	// set a cookie for bar.com.
	//
	// A public suffix list implementation is in the package
	// golang.org/x/net/publicsuffix.
	type PublicSuffixList interface {
		...
	}

Un marqueur de liste numérotée est un nombre décimal de longueur quelconque
suivi d'un point ou d'une parenthèse fermante, puis d'un espace ou d'une tabulation, et du texte.
Dans une liste numérotée, chaque ligne commençant par un marqueur de numéro commence un nouvel élément de liste.
Les numéros d'éléments sont laissés tels quels, jamais renumérotés.

Par exemple :

	package path

	// Clean returns the shortest path name equivalent to path
	// by purely lexical processing. It applies the following rules
	// iteratively until no further processing can be done:
	//
	//  1. Replace multiple slashes with a single slash.
	//  2. Eliminate each . path name element (the current directory).
	//  3. Eliminate each inner .. path name element (the parent directory)
	//     along with the non-.. element that precedes it.
	//  4. Eliminate .. elements that begin a rooted path:
	//     that is, replace "/.." by "/" at the beginning of a path.
	//
	// The returned path ends in a slash only if it is the root "/".
	//
	// If the result of this process is an empty string, Clean
	// returns the string ".".
	//
	// See also Rob Pike, "[Lexical File Names in Plan 9]."
	//
	// [Lexical File Names in Plan 9]: https://9p.io/sys/doc/lexnames.html
	func Clean(path string) string {
		...
	}

Les éléments de liste ne contiennent que des paragraphes, pas de blocs de code ni de listes imbriquées.
Cela évite toute subtilité de comptage d'espaces ainsi que les questions sur
combien d'espaces compte une tabulation dans une indentation incohérente.

Gofmt reformate les listes à puces pour utiliser un tiret comme marqueur de puce,
deux espaces d'indentation avant le tiret,
et quatre espaces d'indentation pour les lignes de continuation.

Gofmt reformate les listes numérotées pour utiliser un seul espace avant le numéro,
un point après le numéro, et de nouveau
quatre espaces d'indentation pour les lignes de continuation.

Gofmt préserve mais n'exige pas de ligne vide entre une liste et le paragraphe précédent.
Il insère une ligne vide entre une liste et le paragraphe ou le titre suivant.

### Blocs de code {#code}

Un bloc de code est une suite de lignes indentées ou vides
ne commençant pas par un marqueur de liste à puces ou un marqueur de liste numérotée.
Il est rendu comme du texte préformaté (un bloc \<pre> en HTML).

Les blocs de code contiennent souvent du code Go. Par exemple :

{{raw `
	package sort

	// Search uses binary search...
	//
	// As a more whimsical example, this program guesses your number:
	//
	//	func GuessingGame() {
	//		var s string
	//		fmt.Printf("Pick an integer from 0 to 100.\n")
	//		answer := sort.Search(100, func(i int) bool {
	//			fmt.Printf("Is your number <= %d? ", i)
	//			fmt.Scanf("%s", &s)
	//			return s != "" && s[0] == 'y'
	//		})
	//		fmt.Printf("Your number is %d.\n", answer)
	//	}
	func Search(n int, f func(int) bool) int {
		...
	}
`}}

Bien sûr, les blocs de code contiennent aussi souvent du texte préformaté autre que du code. Par exemple :

{{raw `
	package path

	// Match reports whether name matches the shell pattern.
	// The pattern syntax is:
	//
	//	pattern:
	//		{ term }
	//	term:
	//		'*'         matches any sequence of non-/ characters
	//		'?'         matches any single non-/ character
	//		'[' [ '^' ] { character-range } ']'
	//		            character class (must be non-empty)
	//		c           matches character c (c != '*', '?', '\\', '[')
	//		'\\' c      matches character c
	//
	//	character-range:
	//		c           matches character c (c != '\\', '-', ']')
	//		'\\' c      matches character c
	//		lo '-' hi   matches character c for lo <= c <= hi
	//
	// Match requires pattern to match all of name, not just a substring.
	// The only possible returned error is [ErrBadPattern], when pattern
	// is malformed.
	func Match(pattern, name string) (matched bool, err error) {
		...
	}
`}}

Gofmt indente toutes les lignes d'un bloc de code d'une seule tabulation,
en remplaçant toute autre indentation commune aux lignes non vides.
Gofmt insère également une ligne vide avant et après chaque bloc de code,
distinguant clairement le bloc de code du texte de paragraphe environnant.

### Directives {#directives}

Les commentaires de directive comme `//go:generate` ne sont pas
considérés comme faisant partie d'un commentaire de doc et sont omis de
la documentation rendue.
Gofmt déplace les commentaires de directive à la fin du commentaire de doc,
précédés d'une ligne vide.
Par exemple :

	package regexp

	// An Op is a single regular expression operator.
	//
	//go:generate stringer -type Op -trimprefix Op
	type Op uint8

Un commentaire de directive est une ligne commençant par l'expression régulière
`//(line |extern |export |[a-z0-9]+:[a-z0-9])`.

Les outils peuvent définir leurs propres commentaires de directive en utilisant la forme
`//nomoutil:directive arguments`.
Les directives d'outil correspondent à l'expression régulière
`//([a-z0-9]+):([a-z0-9]\PZ*)($|\pZ+)(.*)`, où le premier groupe
est le nom de l'outil et le deuxième groupe est le nom de la directive.
Les arguments optionnels sont séparés du nom de la directive par
un ou plusieurs caractères d'espace Unicode.
Chaque outil peut définir sa propre syntaxe d'arguments, mais une convention courante est une
séquence d'arguments séparés par des espaces, où un argument peut être
un mot simple, ou une string Go entre guillemets doubles ou backticks.
Le nom d'outil `go` est réservé à l'utilisation par la chaîne d'outils Go.

La fonction [`go/ast.ParseDirective`](/pkg/go/ast#ParseDirective) et ses
types associés analysent la syntaxe des directives d'outil.

## Erreurs fréquentes et pièges {#mistakes}

La règle selon laquelle toute suite de lignes indentées ou vides
dans un commentaire de doc est rendue comme un bloc de code
remonte aux tout premiers jours de Go.
Malheureusement, l'absence de support pour les commentaires de doc dans gofmt
a conduit à de nombreux commentaires existants qui utilisent l'indentation
sans vouloir créer un bloc de code.

Par exemple, cette liste non indentée a toujours été interprétée
par godoc comme un paragraphe de trois lignes suivi d'un bloc de code d'une ligne :

	package http

	// cancelTimerBody is an io.ReadCloser that wraps rc with two features:
	// 1) On Read error or close, the stop func is called.
	// 2) On Read failure, if reqDidTimeout is true, the error is wrapped and
	//    marked as net.Error that hit its timeout.
	type cancelTimerBody struct {
		...
	}

Cela s'est toujours rendu dans `go` `doc` comme :

	cancelTimerBody is an io.ReadCloser that wraps rc with two features:
	1) On Read error or close, the stop func is called. 2) On Read failure,
	if reqDidTimeout is true, the error is wrapped and

	    marked as net.Error that hit its timeout.

De même, la commande dans ce commentaire est un paragraphe d'une ligne
suivi d'un bloc de code d'une ligne :

	package smtp

	// localhostCert is a PEM-encoded TLS cert generated from src/crypto/tls:
	//
	// go run generate_cert.go --rsa-bits 1024 --host 127.0.0.1,::1,example.com \
	//     --ca --start-date "Jan 1 00:00:00 1970" --duration=1000000h
	var localhostCert = []byte(`...`)

Cela s'est rendu dans `go` `doc` comme :

	localhostCert is a PEM-encoded TLS cert generated from src/crypto/tls:

	go run generate_cert.go --rsa-bits 1024 --host 127.0.0.1,::1,example.com \

	    --ca --start-date "Jan 1 00:00:00 1970" --duration=1000000h

Et ce commentaire est un paragraphe de deux lignes (la deuxième ligne est "{"),
suivi d'un bloc de code indenté de six lignes et d'un paragraphe d'une ligne ("}").

	// On the wire, the JSON will look something like this:
	// {
	//	"kind":"MyAPIObject",
	//	"apiVersion":"v1",
	//	"myPlugin": {
	//		"kind":"PluginA",
	//		"aOption":"foo",
	//	},
	// }

Et cela s'est rendu dans `go` `doc` comme :

	On the wire, the JSON will look something like this: {

	    "kind":"MyAPIObject",
	    "apiVersion":"v1",
	    "myPlugin": {
	    	"kind":"PluginA",
	    	"aOption":"foo",
	    },

	}

Une autre erreur fréquente était une définition de fonction Go ou une instruction de bloc non indentée,
également encadrée par "{" et "}".

L'introduction du reformatage des commentaires de doc dans gofmt de Go 1.19 rend
ces erreurs plus visibles en ajoutant des lignes vides autour des blocs de code.

Une analyse effectuée en 2022 a révélé que seuls 3 % des commentaires de doc dans les modules Go publics
ont été reformatés du tout par la version préliminaire de gofmt de Go 1.19.
En se limitant à ces commentaires, environ 87 % des reformatages de gofmt
ont préservé la structure qu'une personne déduirait en lisant le commentaire ;
environ 6 % ont été perturbés par ces types de listes non indentées,
commandes shell multilignes non indentées et blocs de code délimités par des accolades non indentées.

Sur la base de cette analyse, le gofmt de Go 1.19 applique quelques heuristiques pour fusionner
les lignes non indentées dans une liste ou un bloc de code indenté adjacent.
Avec ces ajustements, le gofmt de Go 1.19 reformate les exemples ci-dessus en :

	// cancelTimerBody is an io.ReadCloser that wraps rc with two features:
	//  1. On Read error or close, the stop func is called.
	//  2. On Read failure, if reqDidTimeout is true, the error is wrapped and
	//     marked as net.Error that hit its timeout.

	// localhostCert is a PEM-encoded TLS cert generated from src/crypto/tls:
	//
	//	go run generate_cert.go --rsa-bits 1024 --host 127.0.0.1,::1,example.com \
	//	    --ca --start-date "Jan 1 00:00:00 1970" --duration=1000000h

	// On the wire, the JSON will look something like this:
	//
	//	{
	//		"kind":"MyAPIObject",
	//		"apiVersion":"v1",
	//		"myPlugin": {
	//			"kind":"PluginA",
	//			"aOption":"foo",
	//		},
	//	}

Ce reformatage rend la signification plus claire tout en permettant aux commentaires de doc
de s'afficher correctement dans les versions antérieures de Go.
Si l'heuristique prend jamais une mauvaise décision, elle peut être contournée en insérant
une ligne vide pour séparer clairement le texte de paragraphe du texte non-paragraphe.

Même avec ces heuristiques, d'autres commentaires existants nécessiteront des ajustements manuels
pour corriger leur rendu.
L'erreur la plus courante est d'indenter une ligne de texte non indentée avec retour à la ligne.
Par exemple :

	// TODO Revisit this design. It may make sense to walk those nodes
	//      only once.

	// According to the document:
	// "The alignment factor (in bytes) that is used to align the raw data of sections in
	//  the image file. The value should be a power of 2 between 512 and 64 K, inclusive."

Dans ces deux cas, la dernière ligne est indentée, ce qui en fait un bloc de code.
La correction consiste à désindenter les lignes.

Une autre erreur courante est de ne pas indenter une ligne avec retour à la ligne d'une liste ou d'un bloc de code indenté.
Par exemple :

	// Uses of this error model include:
	//
	//   - Partial errors. If a service needs to return partial errors to the
	// client,
	//     it may embed the `Status` in the normal response to indicate the
	// partial
	//     errors.
	//
	//   - Workflow errors. A typical workflow has multiple steps. Each step
	// may
	//     have a `Status` message for error reporting.

La correction consiste à indenter les lignes avec retour à la ligne.

Les commentaires de doc Go ne supportent pas les listes imbriquées, donc gofmt reformate

	// Here is a list:
	//
	//  - Item 1.
	//    * Subitem 1.
	//    * Subitem 2.
	//  - Item 2.
	//  - Item 3.

en

	// Here is a list:
	//
	//  - Item 1.
	//  - Subitem 1.
	//  - Subitem 2.
	//  - Item 2.
	//  - Item 3.

Réécrire le texte pour éviter les listes imbriquées améliore généralement
la documentation et constitue la meilleure solution.
Une autre solution de contournement potentielle est de mélanger les marqueurs de liste,
puisque les marqueurs de puces n'introduisent pas d'éléments de liste dans une liste numérotée,
et vice versa.
Par exemple :

	// Here is a list:
	//
	//  1. Item 1.
	//
	//     - Subitem 1.
	//
	//     - Subitem 2.
	//
	//  2. Item 2.
	//
	//  3. Item 3.
