---
title: Le langage de programmation Go
summary: Go est un langage de programmation open source qui rend simple la création de systèmes sécurisés et scalables.
template: true
---

{{$canShare := not googleCN}}

<section class="Hero bluebg">
  <div class="Hero-gridContainer">
    <div class="Hero-blurb">
      <h1>Construis des systèmes simples, sécurisés et scalables avec Go</h1>
      <ul class="Hero-blurbList">
        <li>
          <svg width="12" height="10" viewBox="0 0 12 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M10.8519 0.52594L3.89189 7.10404L1.14811 4.51081L0 5.59592L3.89189 9.27426L12 1.61105L10.8519 0.52594Z" fill="white" fill-opacity="0.87">
          </svg>
          Un langage de programmation open source soutenu par Google
        </li>
        <li>
          <svg width="12" height="10" viewBox="0 0 12 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M10.8519 0.52594L3.89189 7.10404L1.14811 4.51081L0 5.59592L3.89189 9.27426L12 1.61105L10.8519 0.52594Z" fill="white" fill-opacity="0.87">
          </svg>
          Facile à apprendre et idéal pour les équipes
        </li>
        <li>
          <svg width="12" height="10" viewBox="0 0 12 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M10.8519 0.52594L3.89189 7.10404L1.14811 4.51081L0 5.59592L3.89189 9.27426L12 1.61105L10.8519 0.52594Z" fill="white" fill-opacity="0.87">
          </svg>
          Concurrence intégrée et bibliothèque standard robuste
        </li>
        <li>
          <svg width="12" height="10" viewBox="0 0 12 10" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M10.8519 0.52594L3.89189 7.10404L1.14811 4.51081L0 5.59592L3.89189 9.27426L12 1.61105L10.8519 0.52594Z" fill="white" fill-opacity="0.87">
          </svg>
          Large écosystème de partenaires, de communautés et d'outils
        </li>
      </ul>
    </div>
    <div class="Hero-actions">
      <div
        data-version=""
        class="js-latestGoVersion">
        <a class="Primary" href="/learn/" aria-label="Commencer" aria-describedby="getStarted-description" role="button">Commencer</a>
        <a class="Secondary js-downloadBtn" href="/dl" aria-label="Télécharger" aria-describedby="download-description" role="button">Télécharger</a>
        <div class="screen-reader-only" id="getStarted-description" hidden>
          Ouvre une nouvelle fenêtre avec le guide de démarrage.
        </div>
        <div class="screen-reader-only" id="download-description" hidden>
          Ouvre une nouvelle fenêtre pour télécharger Go.
        </div>
      </div>
      <div class="Hero-footnote">
        <p>
          Télécharge les packages pour
          <a class="js-downloadWin">Windows 64-bit</a>,
          <a class="js-downloadMac">macOS</a>,
          <a class="js-downloadLinux">Linux</a> et
          <a href="/dl/" aria-describedby="newwindow-description">plus</a>
        </p>
        <p>
          La commande <code>go</code> télécharge et authentifie par défaut
          les modules via le miroir de modules Go et la base de données de sommes
          de contrôle Go, gérés par Google. <a href="/dl" aria-describedby="newwindow-description">En savoir plus.</a>
        </p>
      </div>
    </div>
    <div class="screen-reader-only" id="newwindow-description" hidden>
          Ouvre dans une nouvelle fenêtre.
    </div>
    <div class="Hero-gopher">
      <img class="Hero-gopherLadder" src="/images/gophers/ladder.svg" alt="La mascotte Go Gopher grimpant une échelle.">
    </div>
  </div>
</section>
<section class="WhoUses">
  <div class="WhoUses-gridContainer">
    <div class="WhoUses-header">
      <h2 class="WhoUses-headerH2">Entreprises utilisant Go</h2>
      <p class="WhoUses-subheader">Des organisations de tous les secteurs utilisent Go pour alimenter leurs logiciels et services
        <a href="/solutions/" class="WhoUsesCaseStudyList-seeAll" aria-describedby="newwindow-description">
        Voir toutes les études de cas
       </a>
     </p>
    </div>
  <div class="WhoUsesCaseStudyList">
    <ul class="WhoUsesCaseStudyList-gridContainer">
    {{- range newest (pages "/solutions/*")}}{{if eq .series "Case Studies"}}
      {{- if .link }}
        {{- if .inLandingPageGrid }}
          <li class="WhoUsesCaseStudyList-caseStudy">
            <a href="{{.link}}" aria-label="Voir l'étude de cas de {{.company}}, (ouvre dans une nouvelle fenêtre)" target="_blank" rel="noopener"
              class="WhoUsesCaseStudyList-caseStudyLink">
              <img
                loading="lazy"
                height="48"
                width="30%"
                src="/images/logos/{{.logoSrc}}"
                class="WhoUsesCaseStudyList-logo"
                alt="">
            </a>
          </li>
        {{- end}}
      {{- else}}
        <li class="WhoUsesCaseStudyList-caseStudy">
          <a href="{{.URL}}" aria-label="Voir l'étude de cas de {{.company}}, (ouvre dans une nouvelle fenêtre)" class="WhoUsesCaseStudyList-caseStudyLink">
            <img
              loading="lazy"
              height="48"
              width="30%"
              src="/images/logos/{{.logoSrc}}"
              class="WhoUsesCaseStudyList-logo"
              alt="">
            <p>Voir l'étude de cas</p>
          </a>
        </li>
      {{- end}}
    {{- end}}
    {{- end}}
    </ul>
  </div>
</section>
<section class="TestimonialsGo">
  <div class="GoCarousel">
    <div class="GoCarousel-controlsContainer">
      <div class="GoCarousel-wrapper">
        <ul class="js-testimonialsGoQuotes TestimonialsGo-quotes">
          {{- range $index, $element := data "/testimonials.yaml"}}
            <li class="TestimonialsGo-quoteGroup GoCarousel-slide" id="quote_slide{{$index}}">
              <div class="TestimonialsGo-quoteSingleItem">
                <div class="TestimonialsGo-quoteSection">
                  <p class="TestimonialsGo-quote">{{raw .quote}}</p>
                  <div class="TestimonialsGo-author">— {{.name}},
                    <span class="NoWrapSpan">{{.title}}</span>
                    <span class="NoWrapSpan"> at {{.company}}</span>
                  </div>
                </div>
              </div>
            </li>
          {{- end}}
        </ul>
      </div>
    <button class="js-testimonialsPrev GoCarousel-controlPrev" hidden>
      <i class="GoCarousel-icon material-icons">navigate_before</i>
    </button>
    <button class="js-testimonialsNext GoCarousel-controlNext">
      <i class="GoCarousel-icon material-icons">navigate_next</i>
    </button>
  </div>
  </div>
</section>
<section class="Playground">
  <div class="Playground-gridContainer">
    <div class="Playground-headerContainer">
      <h2 class="HomeSection-header">Essaie Go</h2>
    </div>
    <div class="Playground-inputContainer">
      <div class="Playground-preContainer">
        Appuie sur Échap pour quitter l'éditeur.
      </div>
      <textarea class="Playground-input js-playgroundCodeEl" spellcheck="false" aria-label="Essaie Go" aria-describedby="editor-description" id="code">
// You can edit this code!
// Click here and start typing.
package main
import "fmt"
func main() {
  fmt.Println("Hello, 世界")
}</textarea>
    </div>
    <div class="screen-reader-only" id="editor-description" hidden>
      Appuie sur Échap pour quitter l'éditeur.
    </div>
    <div class="Playground-outputContainer js-playgroundOutputEl">
      <pre class="Playground-output"><noscript>Hello, 世界</noscript></pre>
    </div>
    <div class="Playground-controls">
      <select class="Playground-selectExample js-playgroundToysEl" aria-label="Exemples de code">
      <option value="hello.go">Hello, World!</option>
      <option value="life.go">Jeu de la vie de Conway</option>
      <option value="fib.go">Fermeture de Fibonacci</option>
      <option value="peano.go">Entiers de Peano</option>
      <option value="pi.go">Pi concurrent</option>
      <option value="sieve.go">Crible de nombres premiers concurrent</option>
      <option value="solitaire.go">Solveur de Peg Solitaire</option>
      <option value="tree.go">Comparaison d'arbres</option>
      </select>
      <div class="Playground-buttons">
      <button class="Button Button--primary js-playgroundRunEl Playground-runButton" title="Exécuter ce code [shift-enter]">Exécuter</button>
      <div class="Playground-secondaryButtons">
        {{- if $canShare}}
        <button class="Button js-playgroundShareEl Playground-button" title="Partager dans Go Playground">Partager</button>
        {{- end}}
        <a class="Button tour Playground-button" href="/tour/" title="Découvrir Go depuis ton navigateur">Tour</a>
      </div>
      </div>
    </div>
  </div>
</section>
<section class="WhyGo">
  <div class="WhyGo-gridContainer">
    <div class="WhyGo-header">
      <h2 class="WhyGo-headerH2">Ce qu'on peut faire avec Go</h2>
      <p class="WhyGo-subheader">
        Utilise Go pour tout type de développement logiciel
      </p>
    </div>
    <ul class="WhyGo-reasons">
      {{- range first 4 (data "/resources.yaml")}}
        <li class="WhyGo-reason">
          <div class="WhyGo-reasonDetails">
            <div class="WhyGo-reasonIcon" role="presentation">
              <img class="DarkMode-img" src="{{.iconDark}}" alt="{{.iconName}}">
              <img class="LightMode-img" src="{{.icon}}" alt="{{.iconName}}">
            </div>
            <div class="WhyGo-reasonText">
              <h3 class="WhyGo-reasonTitle">{{.title}}</h3>
              <p>
                {{.description}}
              </p>
            </div>
          </div>
          <div class="WhyGo-reasonFooter">
            <div class="WhyGo-reasonPackages">
              <div class="WhyGo-reasonPackagesHeader">
                <img src="/images/icons/package.svg" alt="Packages.">
                Packages populaires :
              </div>
              <ul class="WhyGo-reasonPackagesList">
                {{- range .packages }}
                  <li class="WhyGo-reasonPackage">
                    <a class="WhyGo-reasonLink" href="{{.url}}" target="_blank" rel="noopener">
                      {{.title}}
                    </a>
                  </li>
                  {{- end}}
              </ul>
            </div>
            <div class="WhyGo-reasonLearnMoreLink">
              <a href="{{.link}}" aria-describedby="newwindow-description">En savoir plus 
              <i class="material-icons WhyGo-forwardArrowIcon" aria-hidden="true">arrow_forward</i></a>
            </div>
          </div>
        </li>
      {{- end}}
      {{- if gt (len (data "resources.yaml")) 3}}
        <li class="WhyGo-reason">
          <div class="WhyGo-reasonShowMore">
            <div class="WhyGo-reasonShowMoreImgWrapper">
              <img
                class="WhyGo-reasonShowMoreImg"
                loading="lazy"
                height="148"
                width="229"
                src="/images/gophers/biplane.svg"
                alt="La mascotte Go Gopher fait du skateboard.">
            </div>
            <div class="WhyGo-reasonShowMoreLink">
              <a href="/solutions/use-cases" aria-describedby="newwindow-description">Plus de cas d'usage 
              <i class="material-icons
              WhyGo-forwardArrowIcon" aria-hidden="true">arrow_forward</i></a>
            </div>
          </div>
        </li>
      {{- end}}
    </ul>
  </div>
</section>
<section class="GettingStartedGo">
  <div class="GettingStartedGo-gridContainer">
    <div class="GettingStartedGo-header">
      <h2 class="GettingStartedGo-headerH2">Débuter avec Go</h2>
      <p class="GettingStartedGo-headerDesc">
        Explore une multitude de ressources d'apprentissage : parcours guidés, cours, livres et bien plus encore.
      </p>
      <div class="GettingStartedGo-ctas">
        <a class="GettingStartedGo-primaryCta" href="/learn/"aria-describedby="newwindow-description">Commencer</a>
        <a href="/doc/install/" aria-describedby="newwindow-description">Télécharger Go</a>
      </div>
    </div>
    <div class="GettingStartedGo-resourcesSection">
      <ul class="GettingStartedGo-resourcesList">
        <li class="GettingStartedGo-resourcesHeader">
          Ressources pour apprendre en autonomie
        </li>
        <li class="GettingStartedGo-resourceItem">
          <a href="/learn#guided-learning-journeys" class="GettingStartedGo-resourceItemTitle" aria-describedby="newwindow-description">
            Parcours d'apprentissage guidés
          </a>
          <div class="GettingStartedGo-resourceItemDescription">
            Des tutoriels pas à pas pour te lancer
          </div>
        </li>
        <li class="GettingStartedGo-resourceItem">
          <a href="/learn#online-learning" class="GettingStartedGo-resourceItemTitle" aria-describedby="newwindow-description">
            Apprentissage en ligne
          </a>
          <div class="GettingStartedGo-resourceItemDescription">
            Parcours les ressources et progresse à ton rythme
          </div>
        </li>
        <li class="GettingStartedGo-resourceItem">
          <a href="/learn#featured-books" class="GettingStartedGo-resourceItemTitle" aria-describedby="newwindow-description">
            Livres recommandés
          </a>
          <div class="GettingStartedGo-resourceItemDescription">
            Lis des chapitres structurés et approfondis les fondamentaux
          </div>
        </li>
        <li class="GettingStartedGo-resourceItem">
          <a href="/learn#self-paced-labs" class="GettingStartedGo-resourceItemTitle" aria-describedby="newwindow-description">
            Labs cloud en autonomie
          </a>
          <div class="GettingStartedGo-resourceItemDescription">
            Lance-toi dans le déploiement d'applications Go sur GCP
          </div>
        </li>
      </ul>
      <ul class="GettingStartedGo-resourcesList">
        <li class="GettingStartedGo-resourcesHeader">
          Formations en présentiel
        </li>
        {{- range first 4 (data "/learn/training.yaml")}}
          <li class="GettingStartedGo-resourceItem">
            <a href="{{.url}}" class="GettingStartedGo-resourceItemTitle" aria-describedby="newwindow-description">
              {{.title}}
            </a>
            <div class="GettingStartedGo-resourceItemDescription">
              {{.blurb}}
            </div>
          </li>
        {{- end}}
      </ul>
    </div>
  </div>
</section>
<script src="/js/index.js" defer></script>
