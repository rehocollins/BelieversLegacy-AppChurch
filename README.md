# Conception fonctionnelle des modules

## Objet

Ce document décrit les fonctions, utilisateurs, écrans et règles métier prévus pour Believer's Legacy App Church. Les comptes et rôles, le registre des fidèles et les départements sont enregistrés dans SQLite. Les modules de présence, de finances et de ressources restent à concevoir.

Le cahier des charges « Registre des Fidèles », version 2.0 de novembre 2026, constitue la source des besoins métier. Les choix techniques qu'il propose sont des recommandations du document. Pour le projet en cours, la conception reste compatible avec l'application Laravel déjà commencée.

## Principes de conception

- L'interface est en français, lisible sur téléphone Android et ordinateur, et conçue pour des personnes ayant différents niveaux d'aisance numérique.
- Chaque personne utilise son propre compte. Les droits sont vérifiés côté serveur pour chaque action.
- Les informations pastorales et financières sont traitées comme des données sensibles et visibles seulement par les personnes autorisées.
- Les écritures financières sont traçables : elles sont corrigées ou annulées avec une nouvelle écriture, jamais effacées sans trace.
- Le périmètre est livré par étapes. Les fonctions avancées ne doivent pas bloquer la première version utile.
- Le fonctionnement hors ligne complet et la synchronisation sont un objectif à concevoir séparément, car ils influencent fortement l'architecture et les écrans.

## Utilisateurs et responsabilités

| Rôle | Responsabilité principale |
| --- | --- |
| Super administrateur | Accède à tous les modules, crée les comptes et attribue tous les rôles, y compris celui de super administrateur. |
| Administrateur | Paramètre l'application, les comptes et les rôles. |
| Pasteur | Supervise les données de l'église, le suivi pastoral et les validations qui lui sont réservées. |
| Trésorier | Saisit les opérations financières, suit les budgets et prépare les rapports financiers. |
| Secrétaire | Gère les fiches des fidèles, les présences et certaines ressources. |
| Chef de département | Suit son département, ses membres, ses présences et son budget autorisé. |
| Adjoint | Aide le chef de département selon les droits qui lui sont accordés. |
| Responsable matériel | Suit l'inventaire, les prêts et la maintenance. |
| Auditeur | Consulte les données financières et les journaux autorisés en lecture seule. |
| Membre | Consulte son profil et son historique de dons lorsque cette fonction sera activée. |

Un compte commence sans privilège métier. **Rôle** : ensemble cohérent de responsabilités. **Privilège** : capacité générale (consulter, créer, modifier, valider, exporter…). **Permission** : privilège précis sur un domaine (par exemple `expenses.approve`). **Politique** : condition qui limite une permission selon la personne, le groupe, le dossier, le montant ou l’état du processus. Un rôle ne suffit donc jamais à autoriser l’accès à toutes les fiches d’un module.

### Modèle de contrôle d’accès

Chaque tentative suit cet ordre : compte actif et authentifié → église concernée → permission accordée par rôle → périmètre de la fiche (personnel ou département) → règles de confidentialité et état du processus. Un refus explicite ou une règle de sécurité l’emporte toujours. Les contrôles sont appliqués côté serveur à l’écran, aux recherches, aux liens directs, aux tableaux de bord, aux exports et aux impressions.

| Domaine | Super administrateur | Administrateur | Pasteur | Trésorier | Secrétaire | Chef de département | Adjoint | Responsable matériel | Auditeur | Membre |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Comptes et rôles | Attribuer tous | Gérer sauf super admin | Consulter | — | — | — | — | — | — | — |
| Fiches membres | Gérer | Gérer | Gérer | Consulter | Gérer | Consulter son groupe | Consulter son groupe | — | — | Son profil |
| Notes pastorales | Gérer | — | Gérer | — | — | — | — | — | — | — |
| Présences | Gérer | Consulter | Gérer | — | Gérer | Gérer son groupe | Saisir pour son groupe | — | — | — |
| Entrées financières | Gérer | — | Consulter, valider | Saisir, modifier | — | — | — | — | Lire les agrégats | — |
| Dépenses | Gérer | — | Consulter, valider | Saisir, modifier | — | Demander dans son budget | — | — | Lire les agrégats | — |
| Budgets | Gérer | — | Consulter, valider | Saisir, modifier | — | Consulter son groupe | — | — | Lire les agrégats | — |
| Dons individuels | Gérer | — | Consulter | Consulter | — | — | — | — | — | Consulter les siens |
| Matériel et prêts | Gérer | Consulter | Consulter | — | Consulter | — | — | Gérer | — | — |
| Journal d’audit | Consulter | Consulter | Consulter | — | — | — | — | — | Consulter | — |

« Gérer » signifie consulter, créer et modifier dans le périmètre indiqué; cela n’autorise pas automatiquement une suppression. Les suppressions sensibles suivent une politique de conservation et laissent une trace. Les exports et impressions sont des permissions à part entière, accordées selon le même périmètre que les données. L’administrateur gère les comptes mais ne reçoit pas de droit implicite sur les notes pastorales ou les dons nominatifs. Il ne peut ni créer ni rétrograder un super administrateur. Le pasteur peut consulter les comptes, mais ne peut pas changer leurs rôles.

### Politiques de sécurité et de validation

- **Moindre privilège et refus par défaut :** aucun accès métier sans attribution explicite; une délégation est limitée à un périmètre, motivée et automatiquement expirée.
- **Confidentialité :** les notes pastorales sont réservées au pasteur dans la première version. Les dons individuels ne sont visibles qu’au pasteur, au trésorier et au membre concerné; l’auditeur reçoit des rapports agrégés.
- **Séparation des tâches :** la personne qui soumet une dépense ne peut pas l’approuver. Au-dessus d’un seuil qui reste à définir, deux approbateurs distincts sont nécessaires. Le comptage d’une offrande est confirmé par deux personnes.
- **Intégrité financière :** une opération validée n’est pas effacée; une correction ou annulation est portée par une nouvelle écriture traçable. Une clôture mensuelle est verrouillée; seule une réouverture par le pasteur, avec motif obligatoire, est permise.
- **Traçabilité :** connexions, changements de rôle, consultations sensibles, validations, corrections et exports sont enregistrés avec auteur, date, appareil et détails pertinents. Le journal est en ajout seul.
- **Comptes et revue :** désactiver un compte conserve son historique. Les accès sont revus chaque trimestre par le pasteur avec l’administrateur. Les accès d’urgence sont temporaires, signalés et journalisés.

Pour simplifier la première version, chaque compte aura un rôle principal; les besoins temporaires passeront par une délégation bornée dans le temps. L’éventuel cumul de rôles devra être évalué avec les règles de séparation des tâches avant d’être autorisé.

## Modules

### 1. Accueil, connexion et tableau de bord

**Utilisateurs :** tous les utilisateurs connectés ; l'accueil public présente le projet.

**Fonctions :** présenter Believer's Legacy App Church, ouvrir une session, afficher un espace adapté au rôle et signaler les tâches importantes.

**Écrans :** page d'accueil publique, connexion, récupération d'accès, tableau de bord personnel, notifications et préférences de compte.

**Règles :** la connexion réussie mène au tableau de bord. Un utilisateur ne voit que les raccourcis et indicateurs autorisés par son rôle. Les données du tableau de bord ne contournent jamais les droits des modules.

### 2. Comptes, rôles et appareils

**Utilisateurs :** administrateur ; pasteur pour les opérations de supervision prévues.

**Fonctions :** créer et désactiver les comptes, attribuer un rôle, régler les droits par module, déléguer temporairement un rôle et valider un nouvel appareil.

**Écrans :** liste des comptes, fiche d'un compte, attribution des rôles, matrice des privilèges, délégations, appareils associés et revue des accès.

**Règles :** un compte par personne ; un rôle principal en V1 ; les inscriptions publiques reçoivent le rôle Membre ; le super administrateur et l’administrateur peuvent créer un compte et attribuer un rôle, mais seul le super administrateur peut attribuer le rôle Super administrateur ; un administrateur ne peut pas modifier un compte super administrateur ; le dernier super administrateur ne peut pas être rétrogradé. Les permissions de module sont vérifiées côté serveur. Les contrôles de groupe sont définis dans la politique, mais l’application des périmètres par fiche attend la conception des liens entre utilisateurs, départements et fidèles. Une délégation expire automatiquement et garde son motif ; le départ d'un responsable désactive son compte sans supprimer son historique. Les comptes d’administration ne contournent pas les restrictions sur les notes pastorales et les dons nominatifs.

### 3. Registre des fidèles

**Utilisateurs :** secrétaire, pasteur, chef de département dans le périmètre autorisé, membre pour son propre profil.

**Fonctions :** créer une fiche complète ou une fiche rapide, attribuer automatiquement un numéro, rechercher et filtrer, mettre à jour les coordonnées et statuts, rattacher une personne à plusieurs départements et exporter le registre.

**Écrans :** liste et recherche, formulaire d'ajout rapide, fiche détaillée, modification de profil, filtres, import/export et impression.

**Règles :** prénom et nom sont requis ; chaque fiche reçoit un numéro unique ; toute nouvelle fiche a par défaut le statut principal « Membre » ; les statuts complémentaires (nouveau venu, nouveau converti, baptisé, bénévole, responsable ou inactif) sont ajoutés ou retirés après création ; les fiches supprimées sont archivées afin de conserver l'historique ; les notes pastorales n'apparaissent jamais dans la fiche générale.

### 4. Suivi pastoral

**Utilisateurs :** pasteur et personnes expressément autorisées.

**Fonctions :** suivre les statuts (visiteur, nouveau converti, membre, inactif), consigner des visites, planifier des relances, suivre un parcours de quatre semaines, enregistrer une formation ou un baptême, préparer les rappels d'anniversaire.

**Écrans :** liste de suivi, fiche confidentielle, historique des visites, calendrier et rappels, parcours et baptêmes.

**Règles :** les notes pastorales sont confidentielles et leur consultation est journalisée ; un rappel indique son responsable et son échéance ; les messages d'anniversaire sont préparés pour validation avant envoi.

### 5. Départements, groupes et présences

**Utilisateurs :** secrétaire, pasteur, chefs de département et adjoints dans leur périmètre.

**Fonctions :** créer et rechercher les groupes, choisir le chef et l'adjoint parmi les fiches du registre des fidèles, rattacher plusieurs personnes à un ou plusieurs groupes et consulter les effectifs. L'organisation des réunions, des présences et des calculs d'assiduité sera développée dans le module Présences.

**Écrans :** liste des départements, fiche de groupe, membres du groupe, création d'un culte ou d'une réunion, feuille de présence et historique des présences.

**Règles :** les responsables et adjoints n'accèdent qu'aux groupes qui leur sont confiés et aux fidèles qui y sont rattachés ; les secrétaires, pasteurs, administrateurs et super administrateurs gèrent les groupes ; les groupes supprimés sont archivés ; une présence sera rattachée à une activité et corrigée par un utilisateur autorisé.

### 6. Finances

**Utilisateurs :** trésorier, pasteur, auditeur en lecture seule et chefs de département pour leurs demandes ou budgets autorisés.

**Fonctions :** saisir dîmes, offrandes, actions de grâce, dons et autres entrées ; enregistrer demandes et dépenses ; joindre les reçus ; suivre caisse, banque, fonds dédiés et budgets ; effectuer les clôtures ; consulter les rapports.

**Écrans :** synthèse financière, saisie rapide de collecte, comptage croisé, journal des entrées, demandes de dépense, validation, pièces justificatives, comptes et fonds, budgets, clôtures et rapports.

**Règles :** le comptage des offrandes est confirmé par deux personnes ; une personne ne valide pas sa propre demande ; au-delà d'un seuil à définir, deux approbateurs distincts sont requis ; la clôture mensuelle bloque les modifications ordinaires ; seule une réouverture par le pasteur, avec motif, est permise ; les opérations ne sont jamais effacées mais corrigées par écriture traçable ; les montants individuels des dons sont visibles uniquement aux rôles autorisés ; aucune dépense ne peut dépasser les fonds ou budgets applicables sans traitement explicite.

### 7. Ressources humaines bénévoles et services

**Utilisateurs :** secrétariat, responsables de département et personnes autorisées.

**Fonctions :** recenser les bénévoles, leurs compétences et disponibilités ; préparer les rotations de service ; signaler les changements et rappels.

**Écrans :** répertoire des bénévoles, fiche de compétences, disponibilités, planning des services et vue personnelle du planning.

**Règles :** une affectation respecte les disponibilités connues ; toute rotation automatique reste révisable par un responsable ; les coordonnées personnelles ne sont pas diffusées au-delà du besoin opérationnel.

### 8. Matériel, prêts et maintenance

**Utilisateurs :** responsable matériel, secrétaire, pasteur pour consultation.

**Fonctions :** inventorier les biens, suivre leur état, valeur et lieu, enregistrer un prêt, son emprunteur et son retour prévu, rappeler les retours et suivre les pannes ou entretiens.

**Écrans :** inventaire, fiche matériel, nouveau prêt, prêts en cours, retours, maintenance et historique.

**Règles :** chaque prêt a un responsable, une date de sortie et une échéance ; un matériel indisponible ou en maintenance ne peut être prêté ; la remise et le retour sont confirmés.

### 9. Salles et événements

**Utilisateurs :** secrétaire, responsables d'événement, pasteur et personnes autorisées.

**Fonctions :** créer un événement, réserver une salle, éviter les conflits d'horaire, prévoir budget et matériel, joindre les documents utiles.

**Écrans :** calendrier, création d'événement, réservation de salle, fiche événement, matériel affecté et documents associés.

**Règles :** une salle ne peut pas avoir deux réservations incompatibles ; une réservation précise l'organisateur et les horaires ; les dépenses de l'événement restent soumises aux règles financières ; les documents sont accessibles selon leur niveau de confidentialité.

### 10. Aide sociale

**Utilisateurs :** pasteur et équipe sociale expressément autorisée.

**Fonctions :** suivre les demandes et aides apportées aux familles, coordonner visites et soutien, associer les actions à une fiche de membre lorsqu'il y a lieu.

**Écrans :** demandes à examiner, fiche de suivi confidentielle, aides attribuées, visites et historique.

**Règles :** informations limitées aux personnes en charge ; tout décaissement suit le module financier ; les rapports publics utilisent des données agrégées et n'exposent pas la situation personnelle d'une famille.

### 11. Tableaux de bord, alertes et rapports

**Utilisateurs :** tous les rôles, avec un contenu adapté à leurs autorisations.

**Fonctions :** suivre le nombre de membres, nouveaux venus, présences, activité financière autorisée, prêts, réservations, budgets, anniversaires et rappels.

**Écrans :** tableau de bord par rôle, centre d'alertes, rapports mensuels et annuels, rapports par département ou culte, exports PDF/Excel et impression.

**Règles :** chaque indicateur applique les mêmes droits que les données détaillées ; les chiffres affichent leur période et date de mise à jour ; les exports sont journalisés quand ils concernent des données sensibles.

### 12. Journal d'audit, confidentialité et sauvegardes

**Utilisateurs :** administrateur, pasteur et auditeur selon les catégories d'événements autorisées.

**Fonctions :** enregistrer les connexions, changements de rôles, modifications, validations, exports et consultations sensibles ; effectuer et restaurer les sauvegardes.

**Écrans :** recherche dans le journal, détail d'une action, état des sauvegardes et procédure de restauration réservée aux responsables.

**Règles :** le journal est en ajout seul, conserve l'auteur, l'action, la date, l'appareil et les valeurs avant/après lorsque cela est pertinent ; les utilisateurs ordinaires ne peuvent ni modifier ni effacer le journal ; les sauvegardes sont protégées et leur restauration est vérifiée périodiquement.

### 13. Mode hors ligne et synchronisation

**Utilisateurs :** tous les utilisateurs disposant de droits sur les fonctions concernées.

**Fonctions :** permettre les saisies courantes sans réseau, indiquer l'état de synchronisation, envoyer les modifications au retour de la connexion et signaler les conflits.

**Écrans :** indicateur hors ligne/synchronisation/à jour, file des éléments en attente, détail des erreurs et résolution des conflits autorisés.

**Règles :** chaque élément créé hors ligne reçoit un identifiant unique ; les conflits financiers sont signalés et résolus manuellement par un rôle autorisé ; les droits mis en cache expirent au maximum après sept jours sans vérification en ligne ; les données sensibles locales sont protégées et effacées à la déconnexion ou lors de la désactivation selon les capacités de la solution retenue.

### 14. Notifications, paiements et gestion multi-églises

**Utilisateurs :** utilisateurs abonnés aux alertes ; administrateurs d'organisation ; trésorier pour les paiements.

**Fonctions :** notifications Web Push, liens WhatsApp préremplis, SMS groupés, paiements mobiles et gestion de plusieurs églises ou antennes.

**Écrans :** préférences de notification, modèles de messages, suivi d'envoi, configuration des moyens de paiement et sélecteur d'église/antenne.

**Règles :** aucun message ou paiement n'est envoyé sans consentement et validation nécessaire ; les secrets des fournisseurs restent côté serveur ; une opération de paiement est rapprochée d'une écriture financière sans exposer de secret de paiement ; les données d'une église sont isolées de celles des autres.

## Livraisons proposées

| Étape | Modules et résultat attendu |
| --- | --- |
| V1 — Fondations et registre | Accueil, connexion, comptes et rôles essentiels, fiches des fidèles, recherche, ajout rapide, premier tableau de bord et exports simples. |
| V2 — Vie communautaire | Suivi pastoral, départements, présences, alertes, indicateurs et rappels. |
| V3 — Gestion et contrôle | Finances, budgets, validations, clôtures, ressources, journal d'audit et rapports avancés. |
| V4 — Extensions | Paiements mobiles, SMS, gestion multi-églises et abonnements. |

Le mode hors ligne est une exigence transversale : son niveau de couverture doit être précisé avant de figer l'architecture de synchronisation.

## Décisions fonctionnelles à valider avant la base de données

1. Quels champs d'une fiche membre sont obligatoires, et quels champs exigent un consentement distinct ?
2. Quel identifiant membre faut-il afficher et peut-il être réutilisé après une annulation ?
3. Quels rôles exacts auront accès aux notes pastorales et à l'aide sociale ?
4. Quel seuil déclenche une double validation de dépense, et quelles personnes peuvent approuver ?
5. Quelles devises et quel format de numérotation financière seront utilisés par défaut ?
6. Le mode hors ligne couvre-t-il toutes les saisies du cahier des charges dès la première version, ou seulement les membres et présences au départ ?
7. La première version doit-elle gérer plusieurs églises/antennes, ou cette fonction vient-elle en V4 ?
8. Quelle durée de conservation s'applique aux fiches, pièces justificatives et journaux, selon les pays d'utilisation ?

Une fois ces règles validées, la prochaine étape sera le parcours détaillé de la V1. La conception de la base de données commencera après la validation fonctionnelle des modules et règles métier.
