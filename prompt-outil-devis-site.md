# Prompt : simulateur de prix pour le site Let's Up

> Copie tout ce qui suit dans l'outil qui construira la page (Claude, un développeur, Webflow, WordPress…).
> Les règles viennent de l'outil Devis de Pilotage. Elles sont vérifiées sur le vrai moteur de calcul le 23/11/2026.
> Les coûts internes et la marge sont volontairement absents : seuls les prix de vente sont donnés.

---

## Ta mission

Crée un **simulateur de prix très simple** pour le site de Let's Up, une agence de contenu vidéo et photo basée à Chaponost (69630), près de Lyon.

Le visiteur répond à 3 ou 4 questions. L'outil lui affiche une **fourchette de prix HT** (par exemple « entre 560 € et 720 € HT »), puis l'invite à demander un devis précis.

L'outil n'est **pas** un devis : c'est une estimation. Le vrai devis est fait ensuite par l'équipe dans son outil interne, avec exactement les mêmes règles. La fourchette doit donc retomber juste.

## Ce que l'outil doit être

- **Une seule page, en 3 ou 4 étapes courtes**, sur ordinateur comme sur téléphone (écran étroit d'abord).
- **Le prix se met à jour en direct** à chaque choix. Pas de bouton « calculer ».
- **Français simple**, phrases courtes. On vouvoie le visiteur.
- **Accessible :**
  - vrais boutons et vraies cases ;
  - focus clavier visible ;
  - contrastes suffisants ;
  - le prix annoncé aux lecteurs d'écran (`aria-live="polite"`).
- **Aucune donnée personnelle n'est demandée pour voir le prix.** Le formulaire de contact vient après, et il est facultatif.
- **Sans dépendance lourde :** HTML, CSS et un peu de JavaScript suffisent. Aucun traceur ajouté.
- **Le calcul est fait en centimes entiers**, jamais en euros à virgule : 0,1 + 0,2 doit donner 0,3.

## Les étapes du simulateur

### 1. « Que voulez-vous ? »

Une liste de contenus, chacun avec un compteur − / +, de 0 à 20. Plusieurs contenus peuvent être choisis ensemble.

| Contenu (libellé affiché) | Ce qui est inclus (texte d'aide) |
|---|---|
| Reel court (moins de 45 s) | Montage dynamique |
| Vidéo réseaux sociaux (1 à 1'30, sous-titrée) | Montage rythmé |
| Interview ou témoignage (jusqu'à 1'30) | Tournage de l'interview et montage |
| Vidéo récap d'événement (1'30 à 2 min) | Couverture de l'événement et montage d'un récap |
| Vidéo de présentation (2 à 3 min, avec motion design) | Préparation, tournage à 2 personnes et montage avec motion design |
| Reportage / mini-documentaire (3 à 5 min) | Préparation (écriture, repérage, questions), tournage et montage complet : dérushage, étalonnage, mixage, sous-titres |
| Pack photo (30 à 50 photos retouchées) | 30 à 50 photos retouchées en haute définition |
| Photos d'événement (30 photos) | 30 photos retouchées prises pendant l'événement |
| Capsule tirée d'une vidéo existante | Montage d'une capsule à partir d'une vidéo que vous avez déjà |
| Captation seule, sans montage | Images livrées brutes, sans montage |

Sous la liste, affiche une fois : « Toujours inclus : sous-titres, formats vertical et horizontal, droits d'utilisation. »

**Une question en plus seulement si besoin :** si le visiteur choisit « Vidéo récap d'événement », « Photos d'événement » ou « Captation seule », demande « Combien de temps durera l'événement ? ».
- Les choix sont 1 h, 2 h, 3 h, 4 h, 5 h, 6 h, 8 h.
- La valeur par défaut est 3 h.

**Le reportage a sa propre question :** s'il choisit « Reportage / mini-documentaire », demande « Combien de temps de tournage ? » (et non la durée de l'événement).
- Les choix sont 4 h, 8 h, 2 jours (= 16 h).
- La valeur par défaut est 8 h.

### 2. « Votre secteur »

Choix unique : Restauration, Commerce, Immobilier, Industrie et BTP, Tertiaire et bureaux, Autre. « Autre » vaut comme Commerce.

### 3. « Quand ? »

Choix unique : « En semaine, en journée » ou « Le soir ou le week-end ».

On ne demande **pas** le lieu : le simulateur ne parle jamais de déplacement (voir « À ne pas faire »).

### 4. (Facultatif) « Des options ? »

Un volet replié « Ajouter des options ». Il ne montre que les options des contenus choisis (liste plus bas).

### Le résultat (toujours visible : en bas sur téléphone, à droite sur ordinateur)

- En gros : **« Entre 560 € et 720 € HT »**.
- En dessous, en petit : « soit entre 672 € et 864 € TTC », avec la TVA à 20 %.
- Une ligne de résumé : « Tournage : 1 journée à 1 personne (3 h). »
- La mention : « Estimation indicative, non contractuelle. Votre devis précis vous est envoyé après un court échange. »
- Un bouton **« Recevoir mon devis précis »**. Il ouvre le formulaire de contact du site et pré-remplit le message avec le récapitulatif des choix et la fourchette.

Si rien n'est choisi, affiche « Choisissez au moins un contenu pour voir une estimation. »

---

## Les règles de calcul (à suivre exactement)

### Les constantes

| Constante | Valeur |
|---|---|
| Prix d'une heure de travail d'une personne | **78 € HT** (7 800 centimes) |
| Durée minimale d'un tournage | 2 h |
| Durée maximale d'une journée de tournage | 8 h (au-delà, on compte des journées en plus) |
| Majoration soir et week-end | +25 %, **sur le tournage seulement** (pas sur le montage ni la préparation) |
| TVA | 20 % |
| Fourchette | du prix HT −10 % au prix HT +15 %, chaque borne arrondie à la dizaine d'euros la plus proche |

### Les contenus

Pour chaque contenu, trois données :
- son **tournage** (heures par unité) ;
- son **montage** (heures par unité) ;
- sa **préparation** (heures par unité).

| id | Contenu | Façon de compter le tournage | Tournage (h) | Montage (h) | Préparation (h) |
|---|---|---|---|---|---|
| `reel` | Reel court | par unité, × coefficient du secteur | 0,5 | 1,5 | 0 |
| `video_reseaux` | Vidéo réseaux sociaux | par unité, × coefficient | 1 | 3 | 0 |
| `interview` | Interview ou témoignage | par unité, × coefficient | 1,5 | 3 | 0 |
| `recap_evenement` | Vidéo récap d'événement | durée de l'événement (étape 1), sans coefficient | durée saisie | 6 | 0 |
| `video_presentation` | Vidéo de présentation | une journée à part : 8 h à 2 personnes, par vidéo | 8 (× 2 personnes) | 16 | 4 |
| `reportage` | Reportage / mini-documentaire | durée de tournage saisie (4 h, 8 h, 16 h), sans coefficient | durée saisie | 16 | 4 |
| `pack_photo` | Pack photo | **seul** : 3 h × coefficient ; **avec un autre tournage** : +1 h fixe | 3 ou +1 | 5 | 0 |
| `photos_evenement` | Photos d'événement | aucun tournage en plus ; il faut une durée de présence | 0 | 1,5 | 0 |
| `capsule` | Capsule d'une vidéo existante | aucun tournage | 0 | 1,5 | 0 |
| `captation` | Captation seule | durée de l'événement, sans coefficient | durée saisie | 0 | 0 |

**Coefficients de secteur.** Ils ne s'appliquent qu'aux contenus « par unité » et au pack photo seul :
- Restauration : 1
- Commerce : 1
- Immobilier : 1,2
- Industrie et BTP : 1,3
- Tertiaire et bureaux : 1,1
- Autre : 1

**Photos d'événement sans présence.** Si « Photos d'événement » est choisi sans « Vidéo récap » ni « Captation », compte quand même la durée de l'événement comme une présence. Fais-le exactement comme une captation de cette durée, sans montage.

### Le calcul, étape par étape

1. **Le nombre de personnes.** C'est 1, ou 2 dès qu'une vidéo de présentation est choisie : **tout** le tournage passe alors à 2 personnes.

2. **Le tournage commun**, c'est-à-dire tout sauf les vidéos de présentation. On additionne :
   - pour chaque contenu « par unité » : quantité × heures de tournage × coefficient du secteur ;
   - les durées d'événement :
     - **une seule fois** pour l'ensemble de l'événement, même si récap, captation et photos d'événement sont choisis ensemble ;
     - deux événements distincts ne se prévoient pas dans ce simulateur, ce cas passe par le devis ;
   - la durée de tournage du reportage (4 h, 8 h ou 16 h), **ajoutée telle quelle** à ce total, sans coefficient et sans se confondre avec la durée d'un événement ;
   - si **aucun autre tournage** n'existe : pack photo, quantité × 3 h × coefficient ;
   - les heures de tournage des options cochées (voir plus bas).

   On arrondit ce total **à l'heure supérieure**. S'il est entre 0 et 2 h, on le porte à **2 h**.

3. **Le pack photo s'ajoute à un autre tournage.** S'il existe un autre tournage (y compris une vidéo de présentation), chaque pack photo ajoute **+1 h** au tournage. Cette heure n'est **pas** multipliée par le nombre de personnes.

4. **Les heures de tournage facturées :**
   `tournage commun × personnes + heures ajoutées par les packs photo + (vidéos de présentation × 8 h × 2 personnes)`

5. **Le prix du tournage :** heures de tournage facturées × 78 €, × 1,25 si « soir ou week-end ».

6. **Le prix du montage et de la préparation :** somme de (quantité × heures de montage + heures de montage des options) × 78 €, plus somme de (quantité × heures de préparation) × 78 €. **Jamais majoré.**

7. **Les options à prix fixe :** elles s'ajoutent telles quelles.

8. **Le total :** prix HT = tournage + montage + préparation + options à prix fixe.

9. **La fourchette :**
    - bas = arrondi(HT × 0,90, à 10 € près) ;
    - haut = arrondi(HT × 1,15, à 10 € près).

    Arrondi au plus proche : 563 € donne 560 €, 565 € donne 570 €. Pour le TTC, applique la même chose sur HT × 1,2.

10. **Le résumé :**
    - nombre de journées = arrondi supérieur(tournage commun ÷ 8, ajout photo compris) + nombre de vidéos de présentation ;
    - durée affichée = tournage commun + ajouts photo + 8 h par vidéo de présentation.

    Exemple : « Tournage : 1 journée à 1 personne (3 h). » N'affiche pas cette ligne s'il n'y a aucun tournage (capsule seule).

### Les options (étape 4)

Voici ce que chaque option ajoute. « par vidéo » ou « par pack » veut dire × la quantité du contenu.
- **Heures de montage :** × 78 €, jamais majorées.
- **Heures de tournage :** elles entrent dans le tournage commun, avant l'arrondi.

| Contenu | Option | Effet |
|---|---|---|
| Reel | Voix off | +0,5 h de montage par vidéo |
| Reel | Musique sous licence premium | +40 € |
| Reel | Habillage motion design | +1 h de montage par vidéo |
| Reel | Format carré (1:1) en plus | +0,25 h de montage par vidéo |
| Reel | Livraison express (48 h) | +2 h de montage (une fois) |
| Vidéo réseaux | Voix off | +0,5 h de montage par vidéo |
| Vidéo réseaux | Version longue (3 min) | le montage passe de 3 h à 4,5 h par vidéo |
| Vidéo réseaux | Prises de vues par drone | +150 € |
| Vidéo réseaux | Musique sous licence premium | +40 € |
| Vidéo réseaux | Teaser de 15 s | +1 h de montage par vidéo |
| Interview | Interview à deux caméras | +0,5 h de tournage et +1 h de montage par vidéo |
| Interview | Maquillage ou figurant | +120 € |
| Interview | Version longue (3 min) | le montage passe de 3 h à 4,5 h par vidéo |
| Interview | Teaser de 15 s | +1 h de montage par vidéo |
| Récap d'événement | Prises de vues par drone | +150 € |
| Récap d'événement | Teaser de 15 s | +1,5 h de montage par vidéo |
| Récap d'événement | Musique sous licence premium | +40 € |
| Récap d'événement | Livraison express (48 h) | +2 h de montage (une fois) |
| Vidéo de présentation | Voix off | +1 h de montage par vidéo |
| Vidéo de présentation | Motion design avancé | +4 h de montage par vidéo |
| Vidéo de présentation | Prises de vues par drone | +180 € |
| Vidéo de présentation | Maquillage ou figurant | +150 € |
| Vidéo de présentation | Version longue (5 min) | le montage passe de 16 h à 22 h par vidéo |
| Reportage | Version longue (8 à 10 min) | le montage passe de 16 h à 24 h par reportage |
| Reportage | Version courte de 60 s pour les réseaux | +2 h de montage par reportage |
| Reportage | Prises de vues par drone | +150 € |
| Reportage | Voix off | +1 h de montage par reportage |
| Reportage | Musique sous licence premium | +40 € |
| Pack photo | 10 photos en plus | +1 h de montage par pack |
| Pack photo | Retouches avancées | +2 h de montage par pack |
| Pack photo | Éclairage studio | +1 h de tournage (une fois) |
| Pack photo | Maquillage ou figurant | +120 € |
| Pack photo | Livraison express (48 h) | +1,5 h de montage (une fois) |
| Photos d'événement | 10 photos en plus | +0,5 h de montage |
| Photos d'événement | Retouches avancées | +1 h de montage |
| Photos d'événement | Livraison express (48 h) | +1 h de montage (une fois) |
| Capsule | Voix off | +0,5 h de montage par capsule |
| Capsule | Format carré (1:1) en plus | +0,25 h de montage par capsule |
| Capsule | Livraison express (48 h) | +1 h de montage (une fois) |
| Captation | Deuxième caméra | +90 € |
| Captation | Prise de son avec micro-cravate | +45 € |
| Captation | Prises de vues par drone | +150 € |

Règles des options :
- **Une seule « version longue »** par contenu.
- **Les montants à prix fixe** (drone, musique, maquillage, deuxième caméra, micro) sont ajoutés **une fois**, quelle que soit la quantité.

---

## Cas de test (le résultat doit tomber exactement)

Ces chiffres sortent du vrai moteur de Pilotage. Écris des tests automatiques qui vérifient chacun d'eux.

| Choix | Prix HT | Fourchette HT |
|---|---|---|
| 1 Reel, restauration, semaine | 273 € | 250 € – 310 € |
| 4 Reels, restauration | 624 € | 560 € – 720 € |
| 10 Reels, restauration | 1 560 € | 1 400 € – 1 790 € |
| 1 Vidéo réseaux, commerce | 390 € | 350 € – 450 € |
| 3 Vidéos réseaux, immobilier | 1 014 € | 910 € – 1 170 € |
| 2 Vidéos réseaux + voix off, commerce | 702 € | 630 € – 810 € |
| 1 Interview, tertiaire | 390 € | 350 € – 450 € |
| 1 Interview à deux caméras, industrie et BTP | 546 € | 490 € – 630 € |
| Récap d'événement, 4 h | 780 € | 700 € – 900 € |
| Récap d'événement, 4 h, soir ou week-end | 858 € | 770 € – 990 € |
| 1 Vidéo de présentation, commerce | 2 808 € | 2 530 € – 3 230 € |
| 1 Reportage, 8 h de tournage, semaine | 2 184 € | 1 970 € – 2 510 € |
| Pack photo seul, restauration | 624 € | 560 € – 720 € |
| 2 Reels + pack photo, restauration | 858 € | 770 € – 990 € |
| Récap d'événement 3 h + photos d'événement | 819 € | 740 € – 940 € |
| Captation 5 h + photos d'événement, week-end | 604,50 € | 540 € – 700 € |
| 1 Capsule | 117 € | 110 € – 130 € |
| Captation seule, 3 h | 234 € | 210 € – 270 € |

Exemple déroulé, « 2 Reels + pack photo, restauration » :
1. Tournage commun : 2 × 0,5 h × 1 = 1 h, porté au minimum de 2 h.
2. Le pack photo s'ajoute à un tournage : +1 h, soit 3 h de tournage facturées.
3. Tournage : 3 h × 78 € = 234 €.
4. Montage : (2 × 1,5 h + 5 h) × 78 € = 624 €.
5. Total : 234 + 624 = **858 € HT**.
6. Fourchette : 858 × 0,9 = 772,20, arrondi à 770 € ; 858 × 1,15 = 986,70, arrondi à 990 €.

---

## À ne pas faire

- N'affiche jamais les heures de montage, un taux horaire ou un « prix exact » : seulement la fourchette.
- N'ajoute aucune remise, aucun code promo, aucune « offre ».
- N'invente pas de contenus ni d'options hors de cette liste.
- Ne demande ni e-mail ni téléphone pour voir le prix.
- Ne parle jamais de déplacement : pas de question sur le lieu, pas de ligne « déplacement », pas de supplément. Let's Up le chiffre dans le devis précis quand le tournage est à plus d'une heure de Lyon.
- N'enregistre rien côté navigateur (pas de cookie, pas de stockage local), sauf le récapitulatif passé au formulaire de contact quand le visiteur clique.

## Bonus (si simple à faire)

- Un lien de partage qui recharge les mêmes choix (paramètres dans l'adresse).
- Toutes les règles regroupées dans un seul objet de configuration (contenus, options, secteurs, constantes), pour que Let's Up puisse changer un prix sans toucher au code.
