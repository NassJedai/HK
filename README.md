# Landing page — séance d’essai offerte · HK Health Center (Braine-l’Alleud)

Page passerelle (*bridge page*) affichée **après** le formulaire Lead Ads Facebook / Instagram.
Objectif unique : faire cliquer le prospect sur **WhatsApp**.

Parcours : `Formulaire Meta Ads → cette page → WhatsApp`

Elle reprend la structure et le niveau de finition du projet **C-Fitness Barchon**
(même squelette, même tracking, même approche performance), transposés à l’identité
HK et enrichis d’une section vidéo pour le *mini VSL*.

---

## 1. Contenu du dossier

```
index.html                  ← la page (HTML + CSS + JS inline, 1 seule requête)
assets/
  brand/                    favicons HK (le monogramme est inline dans la page)
  fonts/                    Bebas Neue Pro + Outfit (auto-hébergées, woff2)
  img/                      photos optimisées AVIF / WebP / JPEG, 2 largeurs
  video/                    le mini VSL recompressé (720p + 540p)
_sources/                   sources d'origine — NON déployées (voir .gitignore)
  Mini-VSL-V2.mp4           la vidéo 4K d'origine (1,5 Go)
  frames/                   images extraites de la vidéo en 4K
  photos/  fonts/           médias et polices récupérés sur hkhealthcenter.be
  build-img.py              script qui régénère assets/img/ (voir §10)
README.md
_headers                    en-têtes HTTP (Cloudflare Pages / Netlify)
.gitignore
```

**À déployer : `index.html`, `assets/`, `_headers` — soit ≈ 39 Mo** (dont 36 Mo de
vidéo). Le dossier `_sources/` (1,5 Go) reste sur votre disque.

## 2. Mise en ligne (GitHub privé → Cloudflare Pages)

Page 100 % statique, aucune étape de build.

### a. Pousser sur GitHub

Le dépôt n’est pas encore initialisé. Depuis le dossier du projet :

```bash
git init && git add . && git commit -m "Landing séance offerte HK Health Center"
```

1. Créer un dépôt **privé** et **vide** sur https://github.com/new
   (ne cocher ni README, ni .gitignore, ni licence).
2. Puis :

```bash
git remote add origin https://github.com/VOTRE-COMPTE/hk-landing-seance-offerte.git && git branch -M main && git push -u origin main
```

Git demandera votre identifiant GitHub puis un mot de passe : ce n’est **pas**
votre mot de passe de compte mais un **jeton d’accès personnel**, à créer sur
https://github.com/settings/tokens (« Generate new token (classic) », portée
`repo`). Le jeton sera mémorisé dans le trousseau macOS : une seule fois.

> `_sources/` est exclu par `.gitignore` : le dépôt pèse ≈ 39 Mo au lieu de 1,6 Go.
> Sans cette exclusion, GitHub refuserait le push (la vidéo 4K fait 1,5 Go, la
> limite est de 100 Mo par fichier).

### b. Brancher Cloudflare Pages

1. https://dash.cloudflare.com → **Workers & Pages** → **Create application** →
   lien **« Continue to Pages »** en bas → **Import an existing Git repository**,
   puis choisir le dépôt. (Cloudflare pousse désormais le flux Workers ; c'est bien
   le flux **Pages** qu'il faut, comme pour cfitness, vlsport et polaris.)
2. Réglages de build :

   | Champ | Valeur |
   |---|---|
   | Framework preset | **None** |
   | Build command | *(laisser vide)* |
   | Build output directory | `/` |

3. **Save and Deploy**. Le site est en ligne sur `https://NOM-DU-PROJET.pages.dev`.

Chaque `git push` sur `main` redéploie automatiquement.

### c. C'est en ligne

Le site est en production sur **https://hk-health-center.pages.dev**
(dépôt : https://github.com/NassJedai/HK, privé).

`og:url` et `og:image` sont déjà en URL absolues sur ce domaine — sans ça,
**aucun aperçu ne s'affiche** quand le lien est partagé, WhatsApp compris.
Si vous changez de domaine, ce sont les deux seules lignes à mettre à jour.

Si vous ajoutez un domaine personnalisé (onglet **Custom domains** du projet
Cloudflare) — par exemple `essai.hkhealthcenter.be` — c’est ce domaine qu’il faut
mettre à ces deux endroits.

`<meta name="robots" content="noindex, follow">` est actif : la page ne remonte
pas sur Google et ne concurrence donc pas hkhealthcenter.be. Pour la rendre
indexable, supprimer cette ligne du `<head>`.

**URL à renseigner dans la campagne Meta Ads** comme page de destination après
l’envoi du formulaire Lead Ads : `https://hk-health-center.pages.dev`

### Tester en local

```bash
python3 -m http.server 8849
```

Puis ouvrir http://localhost:8849/index.html

> Ce serveur de test ne gère pas les requêtes *Range* : on ne peut pas avancer
> dans la vidéo en local. La lecture depuis le début, elle, fonctionne — voir §7
> pour le comportement en production.

## 3. Le lien WhatsApp

Les **5 CTA** de la page pointent vers exactement la même URL :

```
https://wa.me/32494857366?text=Bonjour%2C%20je%20souhaiterais%20r%C3%A9server%20ma%20s%C3%A9ance%20d%E2%80%99essai%20gratuite%20%2B%20mon%20analyse%20corporelle%20au%20HK%20Health%20Center%20%C3%A0%20Braine-l%E2%80%99Alleud.%20Merci%20%21
```

Message pré-rempli : *« Bonjour, je souhaiterais réserver ma séance d’essai gratuite
+ mon analyse corporelle au HK Health Center à Braine-l’Alleud. Merci ! »*

Pour changer le numéro ou le message : rechercher `wa.me` dans `index.html`
(5 occurrences) et remplacer partout la même URL.

Un **repli téléphone** discret figure sous le bouton du CTA final
(`tel:+32494857366`) pour les prospects sans WhatsApp. Il est volontairement en
petit texte et non en bouton, pour ne pas concurrencer l’objectif principal.
Pour le retirer : supprimer le bloc `<p class="final__alt">`.

## 4. Tracking

### Meta Pixel

Le pixel **`1014657624151124`** (celui déjà utilisé sur `free.hkhealthcenter.be`)
est installé dans l’en-tête de `index.html` (code officiel Meta + repli
`<noscript>` en haut du `<body>`).

Au chargement : `PageView` uniquement. **Aucun événement `Lead` n’est déclenché** —
le lead a déjà été enregistré par le formulaire Meta en amont, le déclencher ici
fausserait le comptage.

À chaque clic sur un CTA WhatsApp, deux événements partent :

| Événement | Type | À quoi il sert |
|---|---|---|
| `Contact` | standard Meta | utilisable comme **objectif d’optimisation** de campagne, et visible dans le Gestionnaire de publicités |
| `WhatsAppClick` | personnalisé | donne le détail par emplacement de bouton |

Les deux portent `cta_location` = `hero` \| `video` \| `final` \| `sticky`, et
`Contact` porte aussi `content_name: "Séance d’essai offerte"` et `method`
(`whatsapp` ou `phone`).

La vidéo envoie en plus :

| Événement | Quand |
|---|---|
| `VideoPlay` | au clic sur le bouton de lecture |
| `VideoProgress` | aux paliers 25 / 50 / 75 / 100 % (une fois chacun) |

Vous pouvez ainsi mesurer le **taux de passage formulaire Meta → WhatsApp**,
comparer la performance des 4 emplacements de bouton, et voir si les gens qui
regardent la vidéo cliquent davantage.

> **À savoir :** la page a été testée en local avec le pixel réel actif. Une
> poignée d’événements de test (`Contact`, `WhatsAppClick`, `VideoPlay`,
> `VideoProgress`) sont donc partis vers le pixel à la date de livraison. Sans
> conséquence, mais autant que vous le sachiez si vous regardez les chiffres du jour.

### Vérifier que ça remonte

Avec l’extension **Meta Pixel Helper** (Chrome), ou dans le Gestionnaire
d’événements Meta → *Tester les événements*.

Attention : les bloqueurs de publicité coupent `fbevents.js`. Si vous en avez un,
vous ne verrez rien remonter depuis votre propre navigateur alors que tout
fonctionne pour les visiteurs. Testez en navigation privée sans extension.

### Ajouter GTM ou GA4 plus tard

Le même écouteur alimente déjà `dataLayer` et `gtag` (`whatsapp_click`,
`phone_click`, `video_play`, `video_progress`). Il suffira de coller le conteneur
dans la page, sans toucher au reste.

### Consentement (RGPD)

Le pixel se déclenche au chargement, sans bandeau de consentement. En Belgique,
un traçage publicitaire nécessite en principe le consentement préalable du
visiteur. Si vous voulez vous mettre en conformité, il faudra ajouter un bandeau
et ne charger le pixel qu’après acceptation — dites-le moi, c’est faisable
proprement sans casser la conversion.

## 5. Bandeau d’urgence (« Plus que 6 places »)

Le bandeau en haut de page est en dur dans le HTML. Pour changer le nombre,
modifier **uniquement le chiffre** dans ce bloc de `index.html` :

```html
<span>Plus que <b>6&nbsp;places</b> pour la séance offerte</span>
```

Il n’y a **aucun compteur automatique** : le chiffre ne bouge pas tout seul et ne
se réinitialise pas. C’est un chiffre que vous annoncez, donc il vous appartient
de le tenir à jour pour qu’il reste exact.

Le bandeau peut être supprimé entièrement (le bloc `<div class="urgency">`) :
le hero reprend automatiquement toute la hauteur de l’écran.

## 6. Choix graphiques

Tout est repris du design system réel de **hkhealthcenter.be** (les variables
`--hk-*` de leur feuille `style-manager.css`), pour que la landing soit
reconnaissable comme du HK et pas comme un template :

| Rôle | Valeur | Usage sur la landing |
|---|---|---|
| Bleu nuit HK | `#1A1F3D` → assombri en `#12162E` | fond de page, cartes, surfaces |
| Violet HK | `#7C6AED` | accents, filets, hexagones, halo vidéo |
| Violet clair HK | `#A89AFA` | sur-titres, bandeau d’urgence, libellés |
| Vert WhatsApp | `#25D366` | les 5 boutons d’appel à l’action |
| Angles coupés | `polygon(0 0, calc(100% - 14px) 0, …)` | boutons, cartes, étiquettes, lecteur vidéo |
| Hexagone | `polygon(50% 0%, 100% 25%, …)` | bouton de lecture, numéros des piliers |
| Easing | `cubic-bezier(.16, 1, .3, 1)` | toutes les transitions |
| Titrage | **Bebas Neue Pro** (leur police) | titres, boutons, sur-titres, chiffres |
| Texte courant | **Outfit** | paragraphes |

Deux écarts assumés par rapport au site vitrine :

1. **Le fond est sombre, alors que hkhealthcenter.be est blanc.** Une landing
   sombre fait ressortir la vidéo et le bouton vert, et c’est la logique qui a
   fonctionné sur C-Fitness. Le bleu nuit reste une couleur de leur charte, donc
   on ne trahit pas la marque. Pour repasser en clair, il faudrait inverser les
   variables `--navy*` / `--white` / `--mist` en tête de la balise `<style>` —
   dites-le moi, c’est une demi-heure de travail.
2. **Le texte courant est en Outfit et non en Bebas Neue.** Le site vitrine
   utilise Bebas Neue pour *tout*, y compris les paragraphes : c’est une police
   étroite et sans bas de casse, illisible sur un texte long. Outfit est déjà
   auto-hébergée sur leur serveur, la parenté visuelle est donc conservée.

### Licence des polices

`assets/fonts/bebas-400/600/900.woff2` sont **les fichiers de HK eux-mêmes**
(Bebas Neue **Pro**, une police commerciale de Dharma Type), récupérés sur
hkhealthcenter.be. C’est leur licence, leur marque : sur un sous-domaine
`hkhealthcenter.be` il n’y a rien à faire. Si la page est déployée sur un domaine
qui n’est pas couvert par leur licence webfont (`*.pages.dev` par exemple, en
usage durable), il suffit de remplacer les trois `@font-face` `Bebas HK` par la
version gratuite de Google Fonts — le rendu est quasi identique, seule la graisse
900 n’existe pas et retombe sur la 400.

## 7. La vidéo (mini VSL)

La vidéo d’origine (`Mini VSL V2.mp4`, Google Drive) fait **1,5 Go** : 3840×2160,
2 min 42, 78 Mbit/s. Inutilisable telle quelle sur le web.

Deux versions sont générées :

| Fichier | Définition | Poids | Servi à |
|---|---|---|---|
| `assets/video/vsl-720.mp4` | 1280×720 | 23,5 Mo | écrans ≥ 720 px |
| `assets/video/vsl-540.mp4` | 960×540 | 14,2 Mo | mobiles (< 720 px) |

**Rien n’est téléchargé avant le clic.** Tant que le visiteur ne clique pas, seule
l’image d’affiche (20 Ko en AVIF) est chargée. La balise `<video>` n’est créée
qu’au clic, avec le bon fichier selon la largeur d’écran — c’est ce qui permet de
garder un premier écran à ~117 Ko malgré une vidéo de 2 min 42.

L’image d’affiche est une image extraite de la vidéo à 1,2 s, **recadrée pour
retirer le sous-titre incrusté** (« Bonjour je suis Simon ») qui entrait en
conflit avec l’étiquette « Le mot du gérant ».

### Une limite à connaître : l’avance rapide

Cloudflare Pages **ne répond pas aux requêtes HTTP *Range*** : il renvoie toujours
le fichier entier (vérifié aussi sur cfitness.pages.dev, c’est le comportement de
la plateforme, pas un réglage). Concrètement :

- la lecture depuis le début démarre presque instantanément — le fichier est
  encodé avec `+faststart`, l’en-tête de la vidéo est en tête de fichier ;
- en revanche, **glisser la barre de progression vers un passage non encore
  téléchargé fait patienter** le temps que le téléchargement rattrape.

Pour un VSL de 2 min 42 que les gens regardent en linéaire, c’est sans
conséquence. Si un jour ça devient gênant (vidéo plus longue, chapitrage), les
deux options sont Cloudflare Stream (payant, ~1 €/1000 min vues) ou un bucket R2
derrière un Worker, qui gèrent tous les deux le *Range*.

Les sous-titres étant incrustés dans l’image, la mention *« Vidéo sous-titrée —
vous pouvez la regarder sans le son »* est affichée sous le lecteur : beaucoup de
gens regardent sans son, autant le leur dire.

## 8. Photos

Toutes les photos viennent de vous, du reportage photo présent sur
hkhealthcenter.be. Deux d’entre elles ne sont affichées sur **aucune page** de
votre site : elles ne sont que dans la médiathèque WordPress, je les ai
récupérées via `wp-json/wp/v2/media`. Ce sont justement les deux du hero :

- **desktop** : la salle privée avec la piste bleue et les racks (1200 px)
- **mobile** : un coach et une adhérente à l’élastique devant les espaliers,
  en t-shirts HK — cadrage portrait natif (1000 × 1500 px), donc net sans recadrage

Une première version utilisait des images extraites de la vidéo. Elles montraient
un cours collectif, mais les gens en plein saut sont floutés par le mouvement :
c’était visible et ça faisait « amateur ». Les photos pro les remplacent partout
sauf sur le fond du CTA final, qui est masqué à 95 % par le voile sombre — là,
l’image de cours collectif apporte une texture humaine sans que la netteté compte.

Le hero mobile a été éclairci de 10 % (`BOOST` dans `_sources/build-img.py`) :
le coach et l’adhérente portent du noir sur fond sombre. Le voile du hero mobile
est volontairement très léger dans le haut, et le texte porte une ombre portée
(`text-shadow` sur `.hero__content`) — c’est ce qui permet de garder la photo
lisible **et** le texte lisible sans opacifier tout l’écran.

## 9. Animations

Toutes désactivées automatiquement si `prefers-reduced-motion: reduce` est actif.

- entrée échelonnée des éléments du hero au chargement
- zoom lent (Ken Burns) sur la photo du hero
- reflet qui balaie les CTA principaux toutes les 6 s
- point clignotant sur le bandeau d’urgence
- bandeau défilant des disciplines (« pluridisciplinaire »)
- halo qui pulse autour du bouton de lecture
- apparitions au scroll échelonnées (cartes, piliers, témoignages)
- trait violet des sur-titres qui se dessine
- zoom au survol des cartes photo, relief sur les cartes du protocole
- CTA sticky qui remonte, flèche du hero qui rebondit
- léger dézoom de la photo de fond du CTA final à son apparition

## 10. Choix techniques

- **Poids du premier écran** : ~117 Ko en mobile, ~157 Ko en desktop (HTML
  compressé + 3 polices + image du hero en AVIF). Le HTML fait 62 Ko brut,
  17 Ko compressé.
- **Photos** : AVIF + WebP + JPEG, deux largeurs, `loading="lazy"` sous la ligne
  de flottaison, dimensions explicites (aucun décalage de mise en page).
- **Vidéo** : `preload="none"` de fait — l’élément `<video>` n’existe pas tant
  qu’on n’a pas cliqué.
- **Aucun formulaire, aucun menu, aucune dépendance externe** hors WhatsApp et
  le pixel Meta. Pas de CDN, pas de framework.
- **Accessibilité** : lien d’évitement, `aria-label` sur tous les CTA, FAQ en
  `<details>` natif (fonctionne sans JS), contrastes ≥ 4,5:1 partout — le bandeau
  d’urgence est en violet **clair** exprès, le violet saturé `#7C6AED` avec du
  texte foncé ne donnait que 4,2:1.
- **Sans JavaScript** : tout le contenu reste lisible, les CTA fonctionnent, la
  FAQ s’ouvre. Seules les animations et le lecteur vidéo sont perdus.

## 11. Régénérer les médias

### Images

```bash
python3 _sources/build-img.py
```

Le script lit `_sources/photos/` et `_sources/frames/`, recadre, éclaircit et
réécrit tout `assets/img/` en AVIF + WebP + JPEG. Les recadrages sont dans la
liste `JOBS` (source, nom, ratio, centre X, centre Y, coupe du bas, largeurs).
Le paramètre « coupe du bas » sert à retirer les sous-titres incrustés des images
tirées de la vidéo.

### Vidéo

`ffmpeg` n’est pas installé sur cette machine ; celui embarqué dans le paquet
Python `imageio-ffmpeg` a été utilisé :

```bash
FF=/Users/nassimjedai/Library/Python/3.9/lib/python/site-packages/imageio_ffmpeg/binaries/ffmpeg-macos-aarch64-v7.1
```

Version 720p :

```bash
"$FF" -i "_sources/Mini-VSL-V2.mp4" -vf "scale=1280:720:flags=lanczos" -c:v libx264 -profile:v high -level 4.0 -pix_fmt yuv420p -crf 27 -preset slow -maxrate 1600k -bufsize 3200k -g 60 -c:a aac -b:a 96k -ac 2 -movflags +faststart -y assets/video/vsl-720.mp4
```

Version 540p (mobile) :

```bash
"$FF" -i "_sources/Mini-VSL-V2.mp4" -vf "scale=960:540:flags=lanczos" -c:v libx264 -profile:v high -level 3.1 -pix_fmt yuv420p -crf 28 -preset slow -maxrate 900k -bufsize 1800k -g 60 -c:a aac -b:a 80k -ac 2 -movflags +faststart -y assets/video/vsl-540.mp4
```

## 12. Trois points à valider de votre côté

1. **« ≈ 1000 calories brûlées en 60 min »** — présent sur l’ancienne landing, je
   ne l’ai **pas** repris. C’est une dépense très élevée pour une séance d’essai
   d’une heure, et une promesse chiffrée invérifiable affaiblit le reste de la
   page. Le bloc de chiffres annonce à la place `60 min / 1 coach / 0 € / 0
   engagement`. Si vous y tenez, il suffit de remplacer une des quatre `<li
   class="stat">`.
2. **« Analyse corporelle »** — la page annonce masse musculaire, masse grasse et
   répartition. Votre page « HK Method » mentionne des **mesures InBody** : si
   c’est bien la balance utilisée pendant la séance d’essai, dites-le moi, je
   nomme la marque (c’est un argument fort et vérifiable).
3. **Les 6 places** — chiffre repris de l’ancienne landing. À tenir à jour (§5).

Les trois témoignages (Emilie Vinaimont, Laila Kanawati, Frederic Gonzalez) sont
repris tels quels de votre page actuelle, celui de Frederic légèrement raccourci.
