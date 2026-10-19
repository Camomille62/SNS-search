# SNS-search

Outil **100 % local et hors-ligne** pour analyser, nettoyer et compléter la configuration d'un pare-feu **Stormshield SNS** à partir de ses exports CSV (objets réseau et politique de filtrage / NAT).

Un seul fichier HTML : pas d'installation, pas de serveur, aucune donnée envoyée sur le réseau.

---

## À quoi ça sert ?

Sur un pare-feu SNS qui vit depuis plusieurs années, les objets s'accumulent : doublons, objets orphelins, groupes qui ne servent plus. SNS-search permet de répondre rapidement à :

- **Où cet objet est-il utilisé ?** (règles de filtrage, règles NAT, directement ou via un groupe)
- **Quels objets ne sont plus utilisés** et peuvent être nettoyés ?
- **Quels objets sont en doublon** (même valeur sous des noms différents, présents dans plusieurs groupes) ?
- **Quel objet couvre telle adresse IP ?** et dans quels groupes se trouve-t-il ?
- Quelles règles utilisent telle valeur (IP, port, protocole, interface…) ?

Il permet aussi de **créer de nouveaux objets et de nouvelles règles** puis de ré-exporter un CSV au format SNS, ré-importable dans le pare-feu.

---

## Démarrage rapide

1. Exportez depuis votre SNS les CSV des **objets** et de la **Filter Policy**.
2. Ouvrez `SNS-search_v0_6_secure.html` dans un navigateur moderne (Chrome, Edge, Firefox…).
3. Onglet **Import** : glissez-déposez (ou cliquez pour choisir) les deux fichiers CSV.
4. Parcourez les onglets **Objets**, **Filter** et **Dashboard**.

> Le CSV des objets seul suffit pour explorer les objets. Le CSV de filtrage est nécessaire pour calculer l'**utilisation** des objets (utilisé / indirect / non utilisé).

---

## Fonctionnalités

### 🗂️ Objets
- Recherche plein texte (nom, IP, MAC, membre, règle…), filtre par type d'objet, pagination, colonnes configurables et redimensionnables.
- **Statut d'utilisation** : utilisé directement, utilisé via un groupe, non utilisé.
- Filtres de nettoyage :
  - objets membres d'au moins un groupe inutilisé ;
  - objets dont **tous** les groupes parents sont inutilisés (candidats au nettoyage) ;
  - objets sans groupe ;
  - objets présents dans plusieurs groupes ;
  - **même valeur, noms différents**.
- Fiche détaillée par objet : groupes d'appartenance, règles de filtrage et NAT concernées (source / destination), arbre des membres et valeurs finales résolues.
- **Recherche d'IP** : trouve les objets qui couvrent une adresse ou une plage, et les groupes qui les contiennent.
- **Rapports de doublons** exportables en CSV.
- Création d'objets (avec contrôle des champs : IPv4/IPv6, MAC, longueur et caractères du nom…) et édition des groupes, avec détection de l'erreur Stormshield *« Both objects are groups and have common elements »*.

### 🛡️ Filter Policy
- Affichage des règles regroupées par **séparateurs / sections** (dépliables).
- Recherche par règle, source, destination, port, protocole.
- Filtres par action (pass, block…) et par état.
- Résolution des groupes en valeurs finales, détection des objets introuvables et des références circulaires.
- Création de règles de filtrage, y compris via des champs avancés (`nat_*`, `asq_*`, `route`, `tos`, `from_geo`…).

### 📊 Dashboard
Vue d'ensemble : nombre d'objets par type, répartition utilisé / indirect / non utilisé, actions des règles, groupes d'objets et de services.

### 🌍 Interface multilingue
Français, English, Deutsch, Español, Italiano, Português, Nederlands, Polski (sélecteur en haut à droite).

---

## Import / Export

| | Détail |
|---|---|
| **Formats acceptés** | `.csv` et `.txt` uniquement |
| **Taille max** | 25 Mo par fichier |
| **Lignes max** | 100 000 par fichier |
| **Cellule** | tronquée au-delà de 5 000 caractères |
| **Export « ⬇ CSV »** | exporte **toutes** les données (filtres ignorés), y compris les objets/règles créés dans l'outil, au format SNS ré-importable |
| **Rapports de doublons** | CSV neutralisés contre l'injection de formules (Excel/LibreOffice) |

> ⚠️ Les données vivent **en mémoire** : si vous créez un objet ou une règle, pensez à cliquer sur **⬇ CSV** pour l'enregistrer, sinon tout est perdu à la fermeture de l'onglet.

**Limite connue** : les champs CSV multi-lignes ne sont pas gérés.

---

## Sécurité et confidentialité

Les exports d'un pare-feu sont des données sensibles ; l'outil est conçu pour rester inoffensif :

- **Aucun réseau** : Content-Security-Policy stricte (`connect-src 'none'`, pas de formulaire, pas de ressource externe), aucune bibliothèque tierce.
- **Aucune donnée transmise** : tout est traité localement dans le navigateur.
- **Anti-injection** : toute donnée issue du CSV est échappée avant affichage ; jamais d'`innerHTML` brut.
- **Import durci** : fichiers binaires refusés, en-têtes interdits (`__proto__`…), caractères de contrôle et d'inversion de sens (bidi) supprimés, objets sans prototype.
- **Garde-fous de performance** : saisies temporisées, profondeur d'imbrication des groupes plafonnée (64), protection contre les boucles circulaires.

---

## Débogage

Ouvrez la console du navigateur (**F12**) : les erreurs sont préfixées par `[SNS-search]`.

---

## Structure du fichier

Le fichier HTML est organisé ainsi : `<style>` (CSS) → `<body>` (panneaux et modales) → `<script>` : sécurité (helpers) → constantes → état (`S`) → lecture CSV → index objets / usage → rendu des panneaux → modales → export → dashboard → traductions.

---

## Avertissement

Outil non officiel, sans lien avec Stormshield. Vérifiez toujours les objets et règles générés avant de les réimporter sur un pare-feu en production.
