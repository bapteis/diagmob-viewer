# DiagMob — Viewers de trajets

Trois pages HTML statiques hébergées sur GitHub Pages. Chacune reçoit ses données **uniquement via les paramètres d'URL** — aucune base de données, aucun appel API au moment de l'affichage.

Les URLs sont générées par le script Python `analyse_matchingsV4mapbox.py` et stockées dans la colonne `Carte_Mapbox_URL` de Grist, puis transmises à l'usager via Brevo.

---

## 1. `diagmob-viewer-final.html` — Trajet individuel

Affiche le trajet voiture actuel et le trajet vélo alternatif pour **une personne**.

**Fond de carte** : OpenStreetMap par défaut, bascule possible sur CyclOSM.

**Fonctionnalités** :
- Tracé voiture (rouge) et vélo (vert) décodés depuis les polylines encodées
- Statistiques km / min / D+ / min VAE (~×2/3 du temps vélo)
- Barre de répartition colorée des types de voies cyclables
- Section "En passant au vélo" : économies €, CO₂, kcal, écart de temps vélo et VAE
- Marqueurs domicile 🏠 et travail 🏢
- Légende Forfait Mobilités Durables

**Paramètres URL** :

| Paramètre | Alias | Type | Description |
|---|---|---|---|
| `pv` | `polyline_voiture` | string | Polyline encodée du trajet voiture |
| `pb` | `polyline_velo` | string | Polyline encodée du trajet vélo |
| `dv` | `dist_voiture` | float | Distance voiture (km) |
| `tv` | `duree_voiture` | int | Durée voiture (min) |
| `db` | `dist_velo` | float | Distance vélo (km) |
| `tb` | `duree_velo` | int | Durée vélo (min) |
| `deniv` | — | int | Dénivelé positif (m) |
| `surfaces` | — | string | Types de voies, format `"Piste cyclable 32%, Rue locale 22%, ..."` |
| `lat_dom` | — | float | Latitude domicile |
| `lon_dom` | — | float | Longitude domicile |
| `lat_trav` | — | float | Latitude lieu de travail |
| `lon_trav` | — | float | Longitude lieu de travail |

---

## 2. `diagmob-viewer-equipage.html` — Équipage covoiturage

Affiche les trajets voiture de **2 à 4 covoitureurs** sur une même carte, avec zone de rendez-vous et statistiques de groupe.

**Fond de carte** : OpenStreetMap.

**Fonctionnalités** :
- Tracés voiture par membre (couleurs distinctes)
- Cercle de zone de rendez-vous (centroïde des domiciles)
- Fiche par membre : nom, service, horaires, jours, distance
- Section gains groupe : CO₂, km économisés, coût partagé
- Identifiant d'équipage affiché (`COV-C01-01`)

**Paramètres URL** :

| Paramètre | Type | Description |
|---|---|---|
| `eq_id` | string | Identifiant équipage (ex : `COV-C01-01`) |
| `jours` | string | Jours communs (ex : `lundi,mardi`) |
| `nb` | int | Nombre de membres (max 4, défaut 4) |
| `trav_lat` | float | Latitude lieu de travail commun |
| `trav_lon` | float | Longitude lieu de travail commun |

Pour chaque membre `N` (1 à 4) :

| Paramètre | Type | Description |
|---|---|---|
| `mN_name` | string | Prénom / nom |
| `mN_email` | string | Email (affiché dans la fiche) |
| `mN_service` | string | Service / direction |
| `mN_jours` | string | Jours de présence (ex : `lundi,mercredi`) |
| `mN_h_arr` | string | Heure d'arrivée souhaitée |
| `mN_h_dep` | string | Heure de départ souhaitée |
| `mN_dist` | float | Distance voiture (km) |
| `mN_lat` | float | Latitude domicile |
| `mN_lon` | float | Longitude domicile |
| `mN_pv` | string | Polyline encodée trajet voiture |

---

## 3. `diagmob-viewer-covelotaff.html` — Équipe covélotaff

Affiche les trajets vélo d'un **mentor cycliste** et de ses **candidats voiture**, pour montrer le chemin en commun et donner envie de passer au vélo.

**Fond de carte** : CyclOSM par défaut (affiche les pistes cyclables).

**Fonctionnalités** :
- Tracé vélo du mentor (ligne pleine) et des candidats (ligne pointillée)
- Marqueur ⭐ pour le domicile du mentor, 🏠 pour les candidats, 🏢 pour le travail
- Cercle de zone de rencontre suggérée (800 m)
- Fiche par membre : badge Mentor / Candidat, km, min vélo, min VAE, jours (communs en vert), horaires, barre de surfaces
- Section gains (calculée depuis les données candidats) : CO₂, €, kcal, écart de temps vélo et VAE vs voiture
- Identifiant d'équipage affiché (`CVTAF-C01-01`)
- Légende CyclOSM fixe

**Paramètres URL — équipe** :

| Paramètre | Type | Description |
|---|---|---|
| `eq_id` | string | Identifiant équipe (ex : `CVTAF-C01-01`) |
| `jours_communs` | string | Jours communs mentor + candidats (ex : `lundi,jeudi`) |
| `km_commun` | float | Km de trajet partagé |
| `lat_trav` | float | Latitude lieu de travail |
| `lon_trav` | float | Longitude lieu de travail |

**Paramètres URL — par membre** (N = 1 pour mentor, 2+ pour candidats) :

| Paramètre | Type | Description |
|---|---|---|
| `mN_n` | string | Prénom / nom |
| `mN_jours` | string | Jours de présence |
| `mN_arrivee` | string | Heure d'arrivée souhaitée |
| `mN_depart` | string | Heure de départ souhaitée |
| `mN_pb` | string | Polyline encodée trajet vélo |
| `mN_dist` | float | Distance vélo (km) |
| `mN_duree` | int | Durée vélo (min) |
| `mN_tv` | int | Durée voiture actuelle (min) |
| `mN_dv` | float | Distance voiture actuelle (km) |
| `mN_surfaces` | string | Types de voies, format `"Piste cyclable 32%, ..."` |
| `mN_mentor` | int | `1` si mentor cycliste, `0` si candidat voiture |
| `mN_lat` | float | Latitude domicile |
| `mN_lon` | float | Longitude domicile |

---

## Génération des URLs

Les URLs sont construites par `analyse_matchingsV4mapbox.py` :

| Viewer | Fonction Python | Colonne Grist |
|---|---|---|
| `diagmob-viewer-final.html` | `creer_url_viewer_individuel()` | `Carte_Mapbox_URL` |
| `diagmob-viewer-equipage.html` | `creer_url_viewer_equipage()` | `Carte_Mapbox_URL` |
| `diagmob-viewer-covelotaff.html` | `creer_url_viewer_covelotaff()` | `Carte_Mapbox_URL` |

Les URLs sont limitées à **8 000 caractères** (limite des navigateurs). En cas de dépassement, les paramètres les moins critiques sont tronqués automatiquement.

## Hébergement

Les trois fichiers sont hébergés sur GitHub Pages : `https://bapteis.github.io/diagmob-viewer/`

Aucune dépendance serveur. Les seules ressources externes chargées au runtime sont :
- Leaflet 1.9.4 (carte interactive)
- Tuiles OpenStreetMap / CyclOSM (fond de carte)
