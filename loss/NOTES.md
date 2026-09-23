# loss — notes de contexte (pour Claude et pour moi)

Petit utilitaire pour **compter le temps que l'équipe perd quand les postes rament**
(PC Sopra masterisés Airbus), et le **chiffrer en euros** — argument pour demander des Macs.

## Où ça vit
- Fichier unique : `loss/index.html` dans le repo `GhyslainBruno/presentation` (GitHub Pages, Jekyll).
- URL publique : https://ghyslainbruno.github.io/presentation/loss/
- Déploiement : **automatique** à chaque push sur `main` (rien à lancer).
- Le fichier n'a **pas de front-matter YAML** → Jekyll le sert tel quel, ne pas en ajouter.

## Contraintes
- Doit rester **ultra-léger** : un seul fichier HTML, zéro dépendance, zéro build.
- Doit marcher sur **PC et mobile** (mobile-first), et **depuis le réseau Airbus**
  (GitHub Pages y est joignable ; le backend Firebase doit l'être aussi — à vérifier via
  `DB_URL/devs.json` qui doit renvoyer du JSON).

## Fonctionnement
- **Chrono manuel uniquement** : on appuie quand on attend, on ré-appuie quand on reprend
  la main. (Une ancienne détection « auto » par dérive de timer a été **retirée** car elle
  gonflait le compteur toute seule → non crédible.)
- **Interrupteur « garder l'écran allumé »** (API Wake Lock) pour taper le bouton sur mobile
  sans déverrouiller. Masqué si le navigateur ne supporte pas.
- **Coût** = temps × taux horaire chargé (`€/h`, réglable, défaut 60).
- Export CSV équipe + bouton « Résumé » (texte à coller pour la hiérarchie/Sopra).

## Partage (Firebase Realtime Database, en REST pur, pas de SDK)
- Config : constante `DB_URL` en haut du `<script>` (vide = mode local, pas de partage).
- Règles : `{ "rules": { "devs": { ".read": true, ".write": true } } }`.
- **L'identité, c'est le NOM** (normalisé via `slugify`), pas l'appareil : PC + tél au même
  nom = un seul utilisateur.
- Modèle de données :
  ```
  /devs/<slug>/nom              = "Ghyslain"
  /devs/<slug>/dev/<idAppareil> = { secondes, maj }   // une branche par appareil → les temps s'ADDITIONNENT
  /devs/<slug>/reset            = <timestamp>          // marqueur de reset propagé
  ```
- Écriture : `PATCH /devs/<slug>.json` avec une clé `"dev/<id>"` (met à jour le nom + CET
  appareil sans écraser les autres).
- **Source de vérité de l'affichage = le serveur** (`myServerSec`), pour que PC et tél
  montrent la même valeur. Le `localStorage` ne sert qu'à la contribution de l'appareil courant.
- **Reset propagé** : « Effacer » pose `reset=<now>` et supprime `dev`. Les autres appareils
  au même nom voient `reset > resetSeen` au prochain pull et vident leur local (pas de résurrection).
- Rythme : push si modifié toutes les 10 s, pull de l'équipe toutes les 15 s.

## Limites connues / pièges
- Deux personnes avec le **même prénom** fusionneraient → utiliser des noms distincts.
- Supprimer le nœud `devs` **à la main dans la console Firebase** peut être ré-écrit par un
  appareil resté ouvert : le nettoyage fiable est le **bouton « Effacer »** de l'app.
- Wake Lock : actif seulement **page au premier plan** ; peut être refusé en batterie faible.

## Pour modifier ce projet depuis le téléphone
- Ouvrir une session Claude Code sur le repo `GhyslainBruno/presentation`, dossier `loss/`.
- Décrire le changement ; Claude édite `loss/index.html`, vérifie la syntaxe JS, commit + push
  sur `main` → Pages redéploie automatiquement.
- Garder le fichier auto-suffisant (pas de dépendance externe hors Google Fonts).
