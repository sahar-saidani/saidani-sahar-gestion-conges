# Cahier des charges : Gestion des notes de frais et des congés avec circuit de validation

**Auteur :** [Prénom NOM]
**Module :** ERP : IGL5
**Version :** 1.0 : [JJ/MM/AAAA]
**Outils :** Microsoft Dynamics NAV, Power Apps, Power Automate

---

## 1. Contexte et problématique

Dans une PME fictive, **[NOM DE L'ENTREPRISE, ex. TechNova SARL]** (environ 50 employés), les demandes de congés et les notes de frais sont traitées par email et par fichiers Excel. Cette organisation pose plusieurs problèmes :

- pas de traçabilité des décisions (qui a validé, quand, pourquoi) ;
- soldes de congés erronés ou mis à jour tardivement ;
- demandes perdues ou validées avec retard ;
- saisie en double dans l'outil de comptabilité ;
- aucun contrôle automatique des règles (chevauchement, solde, plafonds).

Le projet propose une **solution intégrée** : Dynamics NAV comme système de référence, Power Apps comme interface utilisateur, Power Automate pour les circuits de validation et les notifications.

> Toutes les données utilisées sont **fictives**.

## 2. Objectifs

### 2.1 Objectif général
Digitaliser et sécuriser le traitement des demandes de congés et des notes de frais avec un circuit de validation à plusieurs niveaux.

### 2.2 Objectifs spécifiques
- O1 : permettre à l'employé de soumettre et suivre ses demandes depuis une application mobile.
- O2 : automatiser la validation hiérarchique (manager, puis RH ou Finance selon un seuil).
- O3 : centraliser les données et les soldes dans NAV.
- O4 : notifier automatiquement les acteurs et relancer les validations en retard.
- O5 : fournir des états de synthèse pour la RH et la Finance.

## 3. Périmètre

### 3.1 Inclus
- Demandes de congés : création, modification, annulation, validation, suivi du solde.
- Notes de frais : création avec plusieurs lignes, justificatif, validation.
- Circuit de validation à deux niveaux avec seuil d'escalade.
- Rôles et permissions dans NAV et dans l'application.
- Notifications, relances planifiées, gestion des erreurs.
- Un état de synthèse (Report NAV).

### 3.2 Exclus
- Calcul de la paie et remboursement bancaire réel.
- Intégration avec un pointage ou un système biométrique.
- Gestion des contrats, des heures supplémentaires et des congés fractionnés par demi-journée (hypothèse de simplification).
- Utilisation de données réelles.

## 4. Acteurs

| Acteur | Description | Droits principaux |
|---|---|---|
| **Employé** | Salarié de l'entreprise | Crée, consulte, modifie (brouillon) et annule ses propres demandes |
| **Manager** | Responsable d'équipe | Consulte et valide ou refuse les demandes de son équipe |
| **RH** | Ressources humaines | Valide les congés au-dessus du seuil, gère types de congés et soldes, consulte les états |
| **Finance** | Service financier | Valide les notes de frais au-dessus du seuil, consulte les états |
| **Système (Power Automate)** | Acteur technique | Notifications, relances, mise à jour de statuts |

> Pour simplifier, RH et Finance peuvent être incarnés par un seul profil « Validateur de second niveau » dans les comptes de démonstration.

## 5. Description des processus

### 5.1 Processus congé
1. L'employé crée une demande (type, date de début, date de fin, commentaire). Statut : **Brouillon**.
2. Il la soumet. Le système vérifie les règles (solde, chevauchement, dates). Statut : **En attente manager**.
3. Le manager approuve ou refuse (avec commentaire).
4. Si approuvé et que la durée dépasse le seuil RH (ou si le type l'exige), statut **En attente RH**, puis décision RH.
5. Si approuvé : statut **Approuvé**, le solde est déduit dans NAV.
6. L'employé est notifié à chaque étape.
7. L'employé peut **annuler** tant que la date de début n'est pas atteinte. Le solde est alors recrédité.

### 5.2 Processus note de frais
1. L'employé crée une note avec une ou plusieurs lignes (date, catégorie, montant, justificatif). Statut : **Brouillon**.
2. Il la soumet. Le système calcule le total et vérifie les règles. Statut : **En attente manager**.
3. Le manager approuve ou refuse.
4. Si le total dépasse le seuil Finance, statut **En attente Finance**, puis décision Finance.
5. Si approuvée : statut **Approuvé**, puis **Payé** (marquage manuel par la Finance).
6. L'employé est notifié à chaque étape.

### 5.3 Relance
Chaque lundi, un flux planifié relance les validateurs pour toute demande en attente depuis plus de 3 jours.

## 6. Règles de gestion

### Congés
| Code | Règle |
|---|---|
| RG-C1 | Seuls les jours ouvrés sont décomptés (samedi, dimanche et jours fériés exclus). |
| RG-C2 | La date de fin doit être supérieure ou égale à la date de début. |
| RG-C3 | Une demande ne peut pas chevaucher une autre demande active du même employé. |
| RG-C4 | Le solde restant doit couvrir le nombre de jours demandés (sauf type « sans solde »). |
| RG-C5 | Une validation RH est requise si la durée dépasse **5 jours ouvrés** ou si le type de congé l'impose. |
| RG-C6 | L'annulation est possible uniquement avant la date de début. |
| RG-C7 | Le solde est déduit à l'approbation finale et recrédité en cas d'annulation. |
| RG-C8 | Un manager ne peut pas valider sa propre demande. |

### Notes de frais
| Code | Règle |
|---|---|
| RG-F1 | Une note contient au moins une ligne. |
| RG-F2 | Chaque ligne a une date, une catégorie, un montant strictement positif et un justificatif. |
| RG-F3 | La date d'une dépense ne peut pas être dans le futur ni antérieure à 60 jours. |
| RG-F4 | Une validation Finance est requise si le total dépasse **500 €** (seuil paramétrable). |
| RG-F5 | Une note refusée peut être corrigée et resoumise. |
| RG-F6 | Une note approuvée n'est plus modifiable. |
| RG-F7 | Un manager ne peut pas valider sa propre note. |

### Générales
| Code | Règle |
|---|---|
| RG-G1 | Chaque changement de statut est horodaté et associé à son auteur. |
| RG-G2 | Un refus exige un commentaire. |
| RG-G3 | Un employé ne voit que ses propres demandes ; un manager voit celles de son équipe. |

## 7. Exigences fonctionnelles

| ID | Exigence | Priorité |
|---|---|---|
| EF-01 | Créer, consulter, modifier, supprimer une demande de congé (brouillon) | Haute |
| EF-02 | Calculer automatiquement le nombre de jours ouvrés | Haute |
| EF-03 | Afficher le solde de congés par employé et par type | Haute |
| EF-04 | Soumettre une demande et déclencher le circuit de validation | Haute |
| EF-05 | Approuver ou refuser (manager, RH, Finance) avec commentaire | Haute |
| EF-06 | Créer une note de frais avec plusieurs lignes et justificatif | Haute |
| EF-07 | Calculer le total d'une note et appliquer le seuil d'escalade | Haute |
| EF-08 | Annuler une demande de congé selon RG-C6 | Moyenne |
| EF-09 | Notifier l'employé et le validateur à chaque changement de statut | Haute |
| EF-10 | Relancer automatiquement les validations en retard | Moyenne |
| EF-11 | Gérer les types de congés, catégories de frais et soldes (RH/Finance) | Moyenne |
| EF-12 | Afficher l'interface selon le rôle de l'utilisateur | Haute |
| EF-13 | Éditer un état de synthèse par employé et par département | Moyenne |
| EF-14 | Historiser les décisions (qui, quand, commentaire) | Moyenne |

## 8. Exigences non fonctionnelles

| ID | Exigence |
|---|---|
| ENF-01 | **Sécurité** : permissions NAV par rôle, principe du moindre privilège. |
| ENF-02 | **Ergonomie** : application utilisable sur mobile et tablette, interface cohérente. |
| ENF-03 | **Fiabilité** : gestion des erreurs dans les flux (Try/Catch, notifications d'échec). |
| ENF-04 | **Traçabilité** : historique des statuts et des approbations. |
| ENF-05 | **Performance** : affichage d'une liste de demandes en moins de 3 secondes (jeu de données de test). |
| ENF-06 | **Maintenabilité** : nommage clair des objets, contrôles et variables ; documentation à jour. |
| ENF-07 | **Confidentialité** : données fictives uniquement. |

## 9. Architecture cible

```
Utilisateurs (Employé / Manager / RH / Finance)
        |
   Power Apps (application Canvas)
        |                    \
        |                 Power Automate (approbations, notifications, relances)
        |                    /
   Web Services (OData pour les données, SOAP pour les actions métier)
        |
   Dynamics NAV (tables, pages, codeunits, report, permissions)
        |
   Teams / Outlook (notifications et approbations)
```

Connexion à NAV : via la passerelle de données locale ou via un point d'accès publié. Si la connexion directe est impossible, une source intermédiaire (SharePoint ou Dataverse) sera utilisée et justifiée dans le rapport.

## 10. Modèle de données (aperçu)

| Table | Contenu principal |
|---|---|
| Leave Type | Code, description, jours par an, validation RH requise |
| Leave Request | N°, employé, type, début, fin, nb jours, statut, approbateur, commentaire |
| Leave Balance | Employé, année, type, droit, pris, restant |
| Expense Header | N°, employé, date, total, statut, approbateur |
| Expense Line | N° note, n° ligne, date, catégorie, montant, justificatif |
| Approval Entry (journal) | Demande, niveau, approbateur, décision, date, commentaire |
| Leave & Expense Setup | Séries de numéros, seuil RH (jours), seuil Finance (montant) |

Les employés, départements et utilisateurs s'appuient sur la table standard **Employee** de NAV.

## 11. Statuts

- **Congé :** Brouillon, En attente manager, En attente RH, Approuvé, Refusé, Annulé.
- **Note de frais :** Brouillon, En attente manager, En attente Finance, Approuvée, Refusée, Payée.

## 12. Hypothèses et contraintes

- Travail individuel, données fictives, environnement de développement ou d'essai.
- Un employé a un seul manager direct.
- Les jours fériés sont saisis dans un calendrier de base paramétrable.
- Le projet est versionné sur GitHub (branches par fonctionnalité, Pull Requests, Issues, Kanban).

## 13. Risques

| Risque | Impact | Parade |
|---|---|---|
| Connexion Power Platform vers NAV impossible | Élevé | Test de connectivité dès le début ; source intermédiaire en secours |
| Périmètre trop large (deux processus) | Moyen | Congés d'abord, notes de frais ensuite |
| Licences ou environnement d'essai expirés | Moyen | Exporter régulièrement objets, application et flux |
| Retard de livraison | Moyen | Suivi Kanban, commits réguliers |

## 14. Livrables

Dépôt GitHub complet, solution fonctionnelle (NAV, Power Apps, Power Automate), guide utilisateur, documentation technique, rapport PDF, support de présentation, vidéo de démonstration (5 à 8 minutes), soutenance avec démonstration en direct.

## 15. Planning prévisionnel

| Phase | Période |
|---|---|
| Cadrage, cahier des charges, modélisation | [dates] |
| Paramétrage et développement NAV | [dates] |
| Power Apps | [dates] |
| Power Automate et intégration | [dates] |
| Tests, bonus | [dates] |
| Documentation, rapport, vidéo, soutenance | [dates] |

## 16. Sources et références

- Énoncé du projet ERP : IGL5.
- Documentation Microsoft : Dynamics NAV, Power Apps, Power Automate. [Ajouter les liens utilisés au fur et à mesure.]
