# Suivi Utilisateurs M365 — Documentation

Outil de suivi des utilisateurs, groupes et listes de diffusion, profils d'accès et matériel d'un tenant Microsoft 365 / Entra ID. Il fonctionne à partir des exports CSV fournis par Microsoft ; il ne se connecte à aucun service.

---

## 1. Présentation

| Point | Détail |
|---|---|
| Fichier | `suivi-utilisateurs-m365.html` (fichier HTML unique) |
| Ouverture | Double-clic sur le fichier, dans un navigateur récent (Edge, Chrome, Firefox) |
| Connexion Microsoft 365 | Aucune. Aucun appel réseau, aucun script externe |
| Données | Stockées dans le navigateur (`localStorage`), propres à chaque profil de navigateur et à chaque poste |
| Langue | Interface en français |

Ce que l'outil fait :

- importe les exports Entra ID (utilisateurs), Exchange / Microsoft 365 (groupes et listes de diffusion) et Intune / Entra (appareils) ;
- affiche les utilisateurs, leurs profils, leurs appareils et leurs groupes ;
- permet de créer, modifier et supprimer des enregistrements **dans l'outil uniquement** ;
- désactive de façon permanente des domaines ou des adresses ;
- gère des demandes de changement de profil approuvées par un administrateur, avec journal.

Ce que l'outil ne fait **pas** :

- il ne modifie ni ne supprime rien dans Microsoft 365 ;
- il ne crée pas de compte, ne change pas d'appartenance à un groupe réel ;
- il ne contrôle pas réellement les droits d'accès (voir section 11).

---

## 2. Exports à fournir

### 2.1 Utilisateurs (Entra ID)

Export des utilisateurs depuis le centre d'administration Entra ID / Microsoft 365, au format CSV.

Colonnes reconnues (le nom de colonne est reconnu quelle que soit la casse, les espaces ou les accents selon les alias listés ci-dessous) :

| Donnée | Colonnes acceptées |
|---|---|
| Identifiant (UPN) | `User principal name`, `UserPrincipalName`, `UPN`, `Email`, `E-mail`, `Adresse e-mail` |
| Nom affiché | `Display name`, `DisplayName`, `Nom complet`, `Nom`, `Nom d'affichage` |
| Prénom / nom | `First name`, `Last name`, `Prénom`, `Nom de famille` |
| Service / fonction / bureau | `Department`, `Service`, `Job title`, `Title`, `Poste`, `Fonction`, `Office`, `Bureau` |
| Téléphone / ville | `Mobile phone`, `Téléphone`, `City`, `Ville` |
| Adresses (alias) | `Proxy addresses` (`SMTP:` = principale, `smtp:` = alias, séparés par `;`) |
| Connexion bloquée | `Block credential` (True = inactif) |
| Corbeille | `Soft deletion time stamp` (rempli = compte supprimé, ignoré à l'import) |
| Profil | `Profil` / `Profile` (optionnel) |

Particularités :

- Les **comptes invités** (`#EXT#`) sont reconnus. Leur adresse réelle est décodée à partir de l'UPN.
- Un compte dont la connexion est bloquée est importé avec le statut **Inactif**.

### 2.2 Groupes et listes de diffusion (Exchange / Microsoft 365)

Export des groupes depuis le centre d'administration Exchange ou Microsoft 365.

| Donnée | Colonnes acceptées |
|---|---|
| Identifiant | `ID de groupe`, `Group ID` |
| Nom | `Nom du groupe`, `Group name`, `Display name` |
| Adresse | `Adresse e-mail principale du groupe`, `Primary SMTP address` |
| Type | `Type de groupe`, `Group type`, `SecurityEnabled` |
| Autres (optionnelles) | `Description`, `Date de création`, `Confidentialité du groupe`, `Appartenance dynamique`, `Inclut Teams` |

Types gérés : Groupe Microsoft 365, Groupe de sécurité, Groupe de sécurité à extension messagerie, Liste de diffusion.

### 2.3 Appareils (Entra ID / Intune)

Export des appareils (Entra ID ou Intune). Colonnes utilisées :

| Donnée | Colonnes acceptées |
|---|---|
| Propriétaire | `owner` (nom affiché **ou** UPN), `Primary user UPN` |
| Nom de l'appareil | `displayName`, `Device name` |
| Identifiant Entra | `deviceId`, `Device ID` |
| Numéro de série | `Serial number` (absent de l'export Entra standard) |
| Modèle / fabricant / système | `model`, `manufacturer`, `operatingSystem`, `operatingSystemVersion` |
| Activé | `accountEnabled` |
| Mode de jonction | `trustType` (`AzureAd` = jointe Entra ID, `Workplace` = inscrite) |
| Inventaire (optionnel) | `Asset tag` |

Remarques :

- L'export Entra standard **ne contient pas de numéro de série**. La clé d'unicité devient alors l'identifiant `deviceId`.
- Si `owner` contient un nom affiché, le rattachement se fait sur ce nom, uniquement s'il correspond à **un seul** compte (voir section 4.4).

### 2.4 Données non importées automatiquement

Les organigrammes et les fichiers de personnel (par exemple un classeur d'effectifs) ne sont pas importés par l'interface. Les profils ont été définis à partir de ces fichiers par un traitement externe, puis importés dans la colonne `Profil` de l'export utilisateurs ou ajoutés à la main.

---

## 3. Interface

### 3.1 Barre d'outils (en haut)

| Bouton | Rôle | Accès |
|---|---|---|
| Sélecteur **Rôle** | Passer en Administrateur ou Lecteur | Tous |
| 🔑 **Mot de passe** | Changer le mot de passe administrateur | Administrateur |
| ➕ **Ajouter** | Créer un utilisateur | Administrateur |
| 📂 **Importer CSV** | Importer un export (détection automatique, section 4) | Administrateur |
| 💾 **Exporter CSV** | Exporter les utilisateurs affichés (après filtres) | Tous |
| ✏️ **Mode Modification** | Activer l'édition dans les fiches | Administrateur |
| 🗑️ **Réinitialiser** | Vider utilisateurs, groupes et profils (pas les demandes ni le journal) | Administrateur |

### 3.2 Zone « Désactivés »

Liste des domaines et adresses désactivés de façon permanente (section 6). Chaque entrée est un badge ; un clic réactive l'entrée.

### 3.3 Onglets

| Onglet | Contenu |
|---|---|
| 👥 **Utilisateurs** | Statistiques, filtres, tableau, sélection multiple, changement de profil en masse |
| 📁 **Groupes & Listes** | Groupes et listes de diffusion importés ou créés, filtres, doublons |
| 🖥️ **Matériel** | Liste à plat de tous les appareils, recherche et filtre par statut |
| 🎭 **Profils** | Profils d'accès, groupes inclus, utilisateurs assignés, couleur |
| 📨 **Demandes** | Demandes de changement de profil et journal des décisions |

---

## 4. Import CSV

### 4.1 Principe

Le bouton **Importer CSV** (ou le bouton équivalent de l'onglet Groupes) lit le fichier et **détecte son type** à partir des en-têtes :

1. Si les en-têtes contiennent des colonnes de groupes → import des **groupes** ;
2. sinon, si les en-têtes contiennent un identifiant d'appareil, un numéro de série ou un nom d'appareil → import du **matériel** ;
3. sinon → import des **utilisateurs**.

Formats acceptés : séparateur `,` ou `;` (détecté automatiquement), encodage UTF-8 (avec ou sans BOM), à défaut Windows-1252 (fichiers Excel).

Le fichier source n'est jamais modifié.

### 4.2 Import des utilisateurs

Une boîte de dialogue propose :

- **OK – Fusionner** : crée les nouveaux comptes et met à jour les existants (clé : UPN, insensible à la casse). Les comptes absents du fichier sont conservés.
- **Annuler – autres options** : puis, **OK – Remplacer** : remplace la liste complète par le fichier (les doublons d'UPN sont dédoublonnés) ; ou **Annuler – Abandonner**.

Points à retenir :

- Les comptes **supprimés** (corbeille) sont ignorés.
- Les comptes **désactivés** (domaine ou adresse désactivé, section 6) sont ignorés, avec un décompte dans le message.
- Pour chaque profil cité dans le fichier qui n'existe pas encore, un **profil est créé automatiquement, sans groupes**. Il faut ensuite compléter ses groupes dans l'onglet Profils.

### 4.3 Import des groupes

- Clé d'unicité : **adresse e-mail** ; à défaut, le **nom**.
- Fusion avec les groupes existants : création ou mise à jour.
- Les groupes dont l'adresse est désactivée sont ignorés, avec un décompte.
- Le message signale : le nombre par type, les groupes **sans adresse**, et les adresses suspectes (« ? » à la place de « @ », fréquent avec certains caractères corrompus).

### 4.4 Import du matériel

- Chaque appareil est rattaché à son **propriétaire** :
  - si la colonne contient une adresse (UPN), la correspondance est exacte ;
  - si elle contient un nom affiché, le rattachement n'a lieu que si **un seul** compte porte ce nom. En cas d'homonymes, l'appareil n'est pas rattaché et le message les liste.
- Clé d'unicité : `deviceId`, à défaut le numéro de série. Un réimport ne crée donc pas de doublon.
- Si un appareil change de propriétaire, il est **déplacé** vers le nouveau compte.
- Un appareil sans propriétaire, ou dont le propriétaire est absent de la liste, n'est pas importé et le nombre est affiché.
- Statut : `Attribué`, ou `Désactivé` si `accountEnabled` vaut False.
- Le type affiché est le modèle, à défaut le système d'exploitation.
- Chaque import est tracé dans le journal (action « Import matériel »).

---

## 5. Filtres, tri et recherche

### 5.1 Utilisateurs

- **Recherche libre** : nom, UPN, service, fonction, bureau, téléphone.
- **Filtres** : statut, profil (y compris « Aucun »), service, présence d'équipements.
- **Domaines** : saisir un domaine (ex. `contoso.com`) pour n'afficher que les comptes de ce domaine.
- **Tri** : cliquer sur l'en-tête d'une colonne triable (les flèches indiquent le sens affiché).
- **Effacer les filtres** remet tous les filtres à zéro.

### 5.2 Groupes et Matériel

- Groupes : recherche, filtres, tri par colonne, bouton *Effacer les filtres*.
- Matériel : recherche libre (nom, utilisateur, UPN, série, identifiant, système) et filtre par statut (Attribués / Désactivés).

---

## 6. Désactivation permanente

Zone « 🚫 Désactivés » en haut de l'écran.

| Saisie | Effet |
|---|---|
| `contoso.com` | Tous les comptes et groupes de ce domaine sont masqués et ignorés à l'import |
| `externe@contoso.com` | Cette adresse précise est masquée |
| `*-external@contoso.com` | Joker `*` : toutes les adresses qui se terminent par `-external@contoso.com` |
| `*.external@contoso.com` | Idem avec un point |

Règles :

- Un joker seul (`*`) est refusé.
- Une adresse désactivée s'applique aussi aux **alias** (`Proxy addresses`) et aux **comptes invités** décodés.
- La désactivation est **persistante** : elle survit aux rechargements et aux imports. Elle ne supprime rien : les éléments masqués restent visibles dans le compteur (« X utilisateur(s) et Y groupe(s) masqués »).
- Un clic sur une entrée la réactive.
- Au moment de la saisie, l'outil indique combien d'éléments sont concernés. Si aucun élément ne correspond, il le signale.

---

## 7. Anti-doublons et suppression

- **Création, modification et import** : un doublon d'UPN (utilisateurs) ou d'adresse / de nom (groupes) est refusé ou fusionné.
- **🧹 Doublons** (onglets Utilisateurs et Groupes) : supprime les doublons en gardant le premier.
- **Supprimer la sélection** : utilise les cases à cocher de chaque ligne ou de l'en-tête (lignes affichées).
- Suppression d'un utilisateur ou d'un groupe : **dans l'outil uniquement**. Rien n'est supprimé dans Microsoft 365.
- Un profil supprimé fait perdre son profil aux utilisateurs concernés (confirmation demandée).

---

## 8. Profils d'accès

Un profil regroupe un nom, une description, une **couleur** et une liste de **groupes / listes** inclus. Chaque utilisateur porte **un seul** profil.

### 8.1 Création et modification

- Onglet **Profils** → *Créer un profil*, ou ✏️ sur une ligne.
- Champs : nom (obligatoire), description, couleur de l'étiquette (sélecteur de couleur ; le texte passe automatiquement en noir ou blanc selon la luminosité), liste des groupes inclus.

### 8.2 Affichage

- Le tableau des profils indique le nombre d'utilisateurs assignés. Un clic sur ce nombre ouvre la **liste détaillée** des utilisateurs du profil (nom, UPN, service, fonction, statut).
- Dans l'onglet Groupes, la colonne *Utilisateurs (via profils)* indique le nombre d'utilisateurs qui reçoivent le groupe par leur profil.

### 8.3 Création automatique

Lors d'un import utilisateurs, les profils cités mais absents sont créés sans groupes. Il faut les compléter.

---

## 9. Changement de profil en masse (Administrateur)

1. Filtrer les utilisateurs (ou cocher une sélection).
2. Cliquer sur **🎭 Changer le profil…**.
3. L'outil indique la portée (sélection, ou ensemble des lignes affichées après filtres) et la répartition actuelle des profils.
4. Choisir le nouveau profil (ou « Aucun » pour retirer le profil), puis **Appliquer**. Une confirmation est demandée.

Seuls les utilisateurs dont le profil change sont modifiés. L'opération est tracée dans le journal.

Par défaut le mot de passe est "administrateur"

---

## 10. Rôles, demandes et journal

### 10.1 Rôles

| Rôle | Droits |
|---|---|
| 👑 **Administrateur** | Toutes les actions : création, modification, suppression, import, approbation des demandes |
| 👁️ **Lecteur** | Consultation, export, filtres, recherche, **demande de changement de profil** |

Les actions réservées sont bloquées côté interface, avec un message explicite.

### 10.2 Mot de passe administrateur

- Pour passer en Administrateur, un mot de passe est demandé s'il a été défini.
- Si aucun mot de passe n'existe, le passage en Administrateur propose d'en créer un (8 caractères minimum, confirmation).
- Le bouton **🔑 Mot de passe** permet de le modifier (le mot de passe actuel est demandé).
- Au rechargement de la page, le mode Administrateur n'est pas rétabli sans le mot de passe. Il reste actif pour la durée de la session du navigateur ; repasser en Lecteur le désactive.
- Le mot de passe n'est jamais stocké en clair : seuls un sel et un hachage PBKDF2-SHA256 (200 000 itérations) sont enregistrés dans le navigateur.

### 10.3 Demande de changement de profil

1. Un lecteur ouvre la fiche d'un utilisateur, puis **📝 Demander un changement de profil**.
2. Il choisit le profil demandé, saisit son nom et un motif (5 caractères minimum).
3. La demande passe au statut **En attente**. Le profil de l'utilisateur **ne change pas**.
4. L'administrateur ouvre l'onglet **Demandes** et :
   - **✔ Approuver** : le profil est appliqué, la demande passe à *Approuvée* ;
   - **✕ Refuser** : un motif facultatif est demandé, la demande passe à *Refusée*.
5. Si le compte n'existe plus au moment de l'approbation, la demande passe à *Obsolète*.

Le filtre en haut de l'onglet affiche les demandes par statut.

### 10.4 Journal des décisions

Chaque action sensible est tracée (date, action, auteur, détail) : demande créée, demande approuvée ou refusée, demande obsolète, modification en masse, import matériel. Le journal affiche les 100 dernières entrées ; il conserve les 500 dernières.

---

## 11. Matériel

- Onglet **🖥️ Matériel** : liste de tous les appareils rattachés aux utilisateurs.
- Colonnes : appareil, propriétaire (nom et UPN), type, système et notes (version, mode de jonction), série, identifiant Entra, statut.
- Recherche et filtre par statut. Lecture seule pour le Lecteur.
- Dans la fiche utilisateur, les équipements sont aussi affichés et, en mode Modification, peuvent être ajoutés ou supprimés à la main (type, inventaire, série, statut, notes).

---

## 12. Export

| Export | Contenu | Remarque |
|---|---|---|
| **Exporter CSV** (Utilisateurs) | Utilisateurs **affichés** après filtres : UPN, nom, prénom, service, profil, bureau, téléphone, ville, fonction, statut, équipements | Séparateur `;`, UTF-8 avec BOM (ouverture directe dans Excel) |
| **Exporter** (Groupes) | Groupes affichés | Même format |

---

## 13. Stockage, sauvegarde et réinitialisation

### 13.1 Clés de stockage (navigateur)

| Clé | Contenu |
|---|---|
| `m365users` | Utilisateurs, équipements inclus |
| `m365groups` | Groupes et listes |
| `m365profiles` | Profils et leurs groupes |
| `m365ignored` | Domaines et adresses désactivés |
| `m365requests` | Demandes de changement de profil |
| `m365audit` | Journal |
| `m365role` | Rôle choisi |
| `m365who` | Dernier nom saisi dans une demande |
| `m365adminpwd` | Sel et hachage du mot de passe administrateur |
| `sessionStorage` `m365adminok` | Session administrateur déverrouillée |

### 13.2 Conséquences

- **Les données ne sont pas partagées** entre les navigateurs, les postes et les utilisateurs. Chacun doit importer ses exports.
- **Effacer les données du site** dans le navigateur supprime tout le contenu de l'outil.
- Un changement de navigateur ou de profil repart de zéro.

### 13.3 Sauvegarde recommandée

Il n'existe pas d'export complet « sauvegarde ». Pour sauvegarder, il faut ré-exporter les données, ou conserver les fichiers CSV sources et réimporter dans l'ordre : utilisateurs, groupes, appareils, puis reprendre les profils et les désactivations.

### 13.4 Réinitialiser

Le bouton **🗑️ Réinitialiser** (Administrateur) vide utilisateurs, groupes et profils. Il **ne vide pas** les demandes, le journal, les domaines désactivés ni le mot de passe administrateur.

---

## 14. Sécurité et limites

Ces points doivent être connus avant tout usage opérationnel.

1. **Contrôle d'accès d'interface, pas de sécurité réelle.** Le rôle et le mot de passe protègent l'affichage et les actions de l'interface. Quiconque peut ouvrir le fichier et modifier le stockage du navigateur (ou le code) peut contourner ces contrôles.
2. **Le journal est une trace de travail**, pas une preuve opposable : il est modifiable par la même voie.
3. **Le mot de passe est local** à chaque navigateur. Il n'est pas partagé.
4. **Le mot de passe est saisi dans une fenêtre standard**, donc visible à l'écran pendant la saisie.
5. **Les données sont en clair** dans le stockage du navigateur (hors mot de passe).
6. **Aucune synchronisation** : deux utilisateurs ne voient pas les mêmes données.
7. **Aucune écriture dans Microsoft 365** : les modifications restent locales et ne sont pas appliquées au tenant.

Pour une gestion réelle des droits et des approbations, les pistes sont :

- **Entra ID Entitlement Management** (packages d'accès, demandes et approbations) ;
- **Microsoft Lists + Power Automate** (formulaire de demande et approbation, avec journal dans Microsoft 365) ;
- groupes Entra ID gérés par des règles d'appartenance dynamiques.

---

## 15. Dépannage

| Message | Cause probable | Action |
|---|---|---|
| « Colonne e-mail / UPN introuvable » | Export utilisateurs sans colonne d'identifiant | Vérifier l'export ; colonne `User principal name` ou `UPN` |
| « Colonne propriétaire … introuvable » (matériel) | Export appareils sans colonne propriétaire | Vérifier que la colonne `owner` ou `Primary user UPN` est présente |
| « Aucun utilisateur / groupe / appareil trouvé » | Fichier vide, séparateur inattendu ou mauvais export | Ouvrir le CSV dans un éditeur de texte ; vérifier l'en-tête |
| Caractères cassés (accents) | Encodage | L'outil relit en Windows-1252 si besoin ; sinon réenregistrer en UTF-8 |
| Homonymes non rattachés (matériel) | Plusieurs comptes portent le même nom affiché | Ajouter l'UPN dans l'export d'appareils, ou rattacher à la main |
| « Action réservée à l'administrateur » | Mode Lecteur actif | Passer en Administrateur (mot de passe) |
| « Mot de passe incorrect » | Erreur de saisie | Réessayer ; le changement passe par 🔑 Mot de passe |
| Profil absent de l'onglet Profils après import | Profil cité mais non créé | Réimporter ; la création est automatique depuis la version courante |

---

## 16. Maintenance (pour un développeur)

### 16.1 Structure du fichier

- **Interface** : barre d'outils, zone de désactivation, onglets, fenêtres modales.
- **Script** : un bloc marqué `===PURE===` / `===END PURE===` contient les fonctions sans accès au DOM (lecture CSV, détection des en-têtes, fusion, clés d'unicité, règles de désactivation, import matériel). Ce bloc est isolé pour être testé sans navigateur.
- Le reste du script gère l'interface, les imports, les profils, les demandes et le rôle.

### 16.2 Ajouter un alias de colonne

Modifier la table correspondante, en **minuscules** et sans espaces superflus :

- `USER_MAP` (utilisateurs), `GROUP_MAP` (groupes), `DEVICE_MAP` (appareils).

Exemple : ajouter `'nom de compte'` à la liste `displayName` de `USER_MAP` pour reconnaître une colonne de ce nom.

### 16.3 Points d'attention

- Toute action admin doit figurer dans la liste `ADMIN_ONLY`, sinon elle reste accessible au Lecteur.
- Les fonctions de rôle (`setRole`, `changeAdminPassword`) sont asynchrones (hachage).
- Les clés de stockage sont listées dans la section 13.1 ; toute nouvelle clé doit y être ajoutée.

### 16.4 Vérifications

Les vérifications réalisées pendant le développement portent sur la logique (fonctions pures) et sur le DOM simulé (jsdom) : import des exports réels (1 035 comptes, 276 appareils, groupes), doublons, désactivations (joker), rôles et demandes, mot de passe administrateur, changement de profil en masse, couleurs, onglet Matériel. **L'interface n'a pas été testée dans un navigateur réel** : une recette manuelle reste nécessaire avant usage.

---

## 17. Glossaire

| Terme | Définition |
|---|---|
| UPN | User Principal Name, identifiant de connexion (adresse de type `prenom.nom@domaine`) |
| Alias | Autre adresse d'un compte (`Proxy addresses`) |
| Compte invité | Utilisateur externe invité dans le tenant (UPN contenant `#EXT#`) |
| Liste de diffusion | Groupe de messagerie sans sécurité, qui distribue les courriels |
| Groupe de sécurité | Groupe utilisé pour attribuer des droits (SharePoint, applications…) |
| Profil | Ensemble de groupes attribué à un utilisateur |
| Appareil « Attribué » | Appareil rattaché à un utilisateur et actif |
| Trust type | Mode de jonction : `AzureAd` (Entra ID joint), `Workplace` (inscrit) |
