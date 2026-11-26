# Modélisation : Gestion des notes de frais et des congés

> Les diagrammes sont écrits en Mermaid et s'affichent directement sur GitHub.
> Entreprise fictive : [NOM DE L'ENTREPRISE]

---

## 1. Diagramme de cas d'utilisation

```mermaid
flowchart LR
    EMP["👤 Employé"]
    MGR["👤 Manager"]
    RH["👤 RH"]
    FIN["👤 Finance"]
    SYS["⚙️ Power Automate"]

    subgraph SYSTEME["Système de gestion des congés et notes de frais"]
        UC1(["Créer une demande de congé"])
        UC2(["Consulter mon solde"])
        UC3(["Annuler une demande de congé"])
        UC4(["Créer une note de frais"])
        UC5(["Ajouter une ligne de frais + justificatif"])
        UC6(["Soumettre une demande"])
        UC7(["Suivre mes demandes"])
        UC8(["Valider ou refuser (niveau 1)"])
        UC9(["Consulter les demandes de l'équipe"])
        UC10(["Valider ou refuser congé (niveau 2)"])
        UC11(["Gérer types de congés et soldes"])
        UC12(["Valider ou refuser note de frais (niveau 2)"])
        UC13(["Marquer une note comme payée"])
        UC14(["Consulter les états de synthèse"])
        UC15(["Vérifier les règles de gestion"])
        UC16(["Notifier les acteurs"])
        UC17(["Relancer les validations en retard"])
    end

    EMP --- UC1
    EMP --- UC2
    EMP --- UC3
    EMP --- UC4
    EMP --- UC7
    MGR --- UC8
    MGR --- UC9
    RH --- UC10
    RH --- UC11
    RH --- UC14
    FIN --- UC12
    FIN --- UC13
    FIN --- UC14
    SYS --- UC16
    SYS --- UC17

    UC4 -.->|include| UC5
    UC1 -.->|include| UC6
    UC4 -.->|include| UC6
    UC6 -.->|include| UC15
    UC8 -.->|include| UC16
    UC10 -.->|include| UC16
    UC12 -.->|include| UC16
```

**Acteurs :** Employé, Manager, RH, Finance (acteurs humains) ; Power Automate (acteur système).

---

## 2. Diagramme d'activité : processus de congé (avec couloirs)

```mermaid
flowchart TD
    subgraph EMP["Couloir Employé"]
        E1(["Début"])
        E2["Créer la demande (Brouillon)"]
        E3["Soumettre la demande"]
        E4["Corriger la demande"]
    end

    subgraph SYS["Couloir Système (NAV + Power Automate)"]
        S1{"Règles respectées ?<br/>solde, chevauchement, dates"}
        S2["Statut = En attente manager<br/>Notifier le manager"]
        S3["Statut = Refusé<br/>Notifier l'employé"]
        S4{"Durée > 5 jours<br/>ou type exigeant RH ?"}
        S5["Statut = En attente RH<br/>Notifier la RH"]
        S6["Statut = Approuvé<br/>Déduire le solde dans NAV<br/>Notifier l'employé"]
        S7(["Fin"])
    end

    subgraph MGR["Couloir Manager"]
        M1{"Manager approuve ?"}
    end

    subgraph RHL["Couloir RH"]
        R1{"RH approuve ?"}
    end

    E1 --> E2 --> E3 --> S1
    S1 -->|Non, message d'erreur| E4
    E4 --> E3
    S1 -->|Oui| S2 --> M1
    M1 -->|Non, commentaire obligatoire| S3
    M1 -->|Oui| S4
    S4 -->|Non| S6
    S4 -->|Oui| S5 --> R1
    R1 -->|Non, commentaire obligatoire| S3
    R1 -->|Oui| S6
    S3 --> S7
    S6 --> S7
```

---

## 3. Diagramme d'activité : processus de note de frais

```mermaid
flowchart TD
    subgraph EMP["Couloir Employé"]
        E1(["Début"])
        E2["Créer la note (Brouillon)"]
        E3["Ajouter les lignes<br/>date, catégorie, montant, justificatif"]
        E4["Soumettre la note"]
        E5["Corriger la note"]
    end

    subgraph SYS["Couloir Système (NAV + Power Automate)"]
        S1{"Règles respectées ?<br/>au moins 1 ligne, justificatif, dates"}
        S2["Calculer le total<br/>Statut = En attente manager<br/>Notifier le manager"]
        S3["Statut = Refusée<br/>Notifier l'employé"]
        S4{"Total > 500 € ?"}
        S5["Statut = En attente Finance<br/>Notifier la Finance"]
        S6["Statut = Approuvée<br/>Notifier l'employé"]
        S7(["Fin"])
    end

    subgraph MGR["Couloir Manager"]
        M1{"Manager approuve ?"}
    end

    subgraph FINL["Couloir Finance"]
        F1{"Finance approuve ?"}
        F2["Marquer la note comme Payée"]
    end

    E1 --> E2 --> E3 --> E4 --> S1
    S1 -->|Non| E5
    E5 --> E4
    S1 -->|Oui| S2 --> M1
    M1 -->|Non, commentaire obligatoire| S3
    M1 -->|Oui| S4
    S4 -->|Non| S6
    S4 -->|Oui| S5 --> F1
    F1 -->|Non, commentaire obligatoire| S3
    F1 -->|Oui| S6
    S6 --> F2 --> S7
    S3 --> S7
```

---

## 4. Diagrammes d'états (cycle de vie)

### 4.1 Demande de congé

```mermaid
stateDiagram-v2
    [*] --> Brouillon
    Brouillon --> EnAttenteManager : Soumettre
    EnAttenteManager --> Refuse : Manager refuse
    EnAttenteManager --> EnAttenteRH : Manager approuve et durée > 5 jours
    EnAttenteManager --> Approuve : Manager approuve et durée <= 5 jours
    EnAttenteRH --> Approuve : RH approuve
    EnAttenteRH --> Refuse : RH refuse
    Brouillon --> Annule : Annuler
    EnAttenteManager --> Annule : Annuler avant la date de début
    EnAttenteRH --> Annule : Annuler avant la date de début
    Approuve --> Annule : Annuler avant la date de début
    Refuse --> [*]
    Annule --> [*]
    Approuve --> [*]
```

### 4.2 Note de frais

```mermaid
stateDiagram-v2
    [*] --> Brouillon
    Brouillon --> EnAttenteManager : Soumettre
    EnAttenteManager --> Refusee : Manager refuse
    EnAttenteManager --> EnAttenteFinance : Manager approuve et total > 500 €
    EnAttenteManager --> Approuvee : Manager approuve et total <= 500 €
    EnAttenteFinance --> Approuvee : Finance approuve
    EnAttenteFinance --> Refusee : Finance refuse
    Refusee --> Brouillon : Corriger
    Approuvee --> Payee : Finance marque payée
    Payee --> [*]
```

---

## 5. Modèle de données

```mermaid
erDiagram
    EMPLOYEE ||--o{ LEAVE_REQUEST : soumet
    EMPLOYEE ||--o{ LEAVE_BALANCE : possede
    EMPLOYEE ||--o{ EXPENSE_HEADER : soumet
    EMPLOYEE ||--o{ EMPLOYEE : manage
    LEAVE_TYPE ||--o{ LEAVE_REQUEST : categorise
    LEAVE_TYPE ||--o{ LEAVE_BALANCE : concerne
    EXPENSE_HEADER ||--|{ EXPENSE_LINE : contient
    EMPLOYEE ||--o{ APPROVAL_ENTRY : decide
    LEAVE_REQUEST ||--o{ APPROVAL_ENTRY : genere
    EXPENSE_HEADER ||--o{ APPROVAL_ENTRY : genere

    EMPLOYEE {
        code No PK
        text Nom
        text Email
        text Departement
        code Manager_No FK
        text Role
    }
    LEAVE_TYPE {
        code Code PK
        text Description
        int Jours_Par_An
        bool Validation_RH
    }
    LEAVE_REQUEST {
        code No PK
        code Employee_No FK
        code Leave_Type FK
        date Date_Debut
        date Date_Fin
        int Nb_Jours
        option Statut
        code Approbateur
        text Commentaire
    }
    LEAVE_BALANCE {
        code Employee_No FK
        int Annee
        code Leave_Type FK
        decimal Droit
        decimal Pris
        decimal Restant
    }
    EXPENSE_HEADER {
        code No PK
        code Employee_No FK
        date Date_Creation
        decimal Total
        option Statut
        code Approbateur
    }
    EXPENSE_LINE {
        code Header_No FK
        int Line_No
        date Date_Depense
        option Categorie
        decimal Montant
        text Justificatif
    }
    APPROVAL_ENTRY {
        int Entry_No PK
        option Type_Document
        code Document_No
        int Niveau
        code Approbateur FK
        option Decision
        datetime Date_Decision
        text Commentaire
    }
    SETUP {
        code Primary_Key PK
        code Leave_Nos
        code Expense_Nos
        int Seuil_Jours_RH
        decimal Seuil_Montant_Finance
    }
```

**Remarque :** `APPROVAL_ENTRY` référence un congé ou une note de frais via `Type_Document` + `Document_No` (relation polymorphe, pas de clé étrangère directe). `Employee` est la table standard NAV (no. 5200), on la réutilise.

---

## 6. Architecture de la solution

```mermaid
flowchart TB
    U["👥 Utilisateurs<br/>Employé, Manager, RH, Finance"]
    PA["📱 Power Apps<br/>Application Canvas"]
    PAU["⚡ Power Automate<br/>Approbation, Notification, Relance"]
    WS["🔌 Web Services NAV<br/>OData (données) + SOAP (actions)"]
    NAV["🗄️ Dynamics NAV<br/>Tables, Pages, Codeunits, Report, Permissions"]
    GW["🔐 Passerelle de données locale"]
    MSG["📧 Outlook / Teams<br/>Notifications et approbations"]

    U --> PA
    PA -->|déclenche un flux| PAU
    PA -->|lecture et écriture| GW
    PAU -->|appels| GW
    GW --> WS
    WS --> NAV
    PAU --> MSG
    MSG --> U
```

**Flux principal :** l'employé soumet dans Power Apps ; Power Apps écrit dans NAV via OData puis déclenche le flux d'approbation ; Power Automate envoie l'approbation au manager (Teams/Outlook) ; la décision met à jour NAV via SOAP ; l'employé est notifié.

---

## 7. Matrice des droits (aperçu)

| Objet | Employé | Manager | RH / Finance |
|---|---|---|---|
| Leave Request | Lire/écrire (les siennes) | Lire (équipe), modifier statut | Lire tout, modifier statut |
| Leave Balance | Lire (le sien) | Lire (équipe) | Lire/écrire |
| Leave Type | Lire | Lire | Lire/écrire |
| Expense Header / Line | Lire/écrire (les siennes) | Lire (équipe), modifier statut | Lire tout, modifier statut |
| Approval Entry | Lire (les siennes) | Lire/insérer | Lire/insérer |
| Setup | Aucun | Aucun | Lire/écrire |
| Report de synthèse | Aucun | Équipe | Tous |
