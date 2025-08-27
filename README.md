# Documentation Complète du Projet et de l'API WAY

Ce document sert de référence centrale pour l'application WAY. Il contient le cahier des charges fonctionnel, le modèle logique de données (MLD) et la spécification technique détaillée de l'API RESTful.

## 1. Cahier des Charges Fonctionnel

### 1.1. Introduction de l'Application WAY

WAY est une plateforme de mise en relation pour la livraison de repas et de colis, connectant des clients, des marchands (restaurants/commerçants) et des livreurs. Le système se compose de plusieurs applications :
-   Application Client
-   Application Marchand/Restaurant
-   Application Livreur
-   Back Office (Administration)

### 1.2. Fonctionnalités par Acteur

#### A. Client
-   **Gestion de Compte :** Inscription, connexion, mise à jour du profil, changement de mot de passe.
-   **Commandes & Livraisons :**
    -   Commander une livraison (repas, colis, courses).
    -   Choisir les lieux de récupération et de livraison.
    -   Gérer plusieurs commandes simultanément.
    -   Programmer des livraisons.
    -   Suivre les livraisons en temps réel.
-   **Options de Livraison :** Choix du type de véhicule (moto, tricycle, voiture, camion, etc.).
-   **Paiement :** Paiement en espèces, Mobile Money (Airtel initialement), carte bancaire.
-   **Portefeuille & Bonus :** Portefeuille interne pour dépôts et utilisation de points bonus.
-   **Historique :** Accès à l'historique complet des commandes et transactions.
-   **Support :** Contacter le service client via un menu dédié.

#### B. Marchand / Restaurant
-   **Inscription :** Processus complet avec validation des informations et des documents justificatifs (pièce d'identité, registre de commerce, etc.).
-   **Gestion des Commandes :**
    -   Voir les zones de livraison et les prix.
    -   Lancer une demande de livreur ("normale" ou "rapide").
    -   Suivre les états de commande en temps réel.
    -   Recevoir des notifications.
    -   Gérer plusieurs demandes en parallèle.
    -   Programmer des commandes.
-   **Tarification :** Le prix de la livraison est dynamique, influencé par la zone, l'heure, la météo et le prix du carburant.
-   **Profil :** Spécifier le type de marchand (restaurant, boutique, grossiste, etc.).

#### C. Livreur
-   **Types de Livreurs :**
    -   **Indépendant :** Possède son propre matériel.
    -   **Entreprise de Livraison (Partenaire) :** Gère une flotte de livreurs employés.
    -   **Livreur Employé :** Rattaché à une entreprise partenaire.
-   **Inscription :** Processus détaillé avec validation des justificatifs par l'administration.
-   **Disponibilité :** Statut "disponible" / "occupé". Mis en indisponibilité après 3 refus consécutifs.
-   **Gestion des Commandes :**
    -   Réception des propositions de course (le plus proche est prioritaire).
    -   Acceptation ou refus des commandes.
    -   Accès aux commandes en attente.
    -   Mise à jour des statuts ("récupérée", "livrée").
    -   Utilisation d'une carte GPS pour la navigation.
-   **Paiement & Commissions :** Compte interne pour les gains, sur lequel une commission est prélevée par la plateforme.
-   **"Street Pickup" :** Possibilité d'ajouter à la plateforme une livraison obtenue en dehors du système.

#### D. Administration (Back Office)
-   **Gestion des Rôles et Droits :** Admin, Gestionnaire, Support, avec possibilité de créer de nouveaux rôles.
-   **Tableau de Bord & Statistiques :** Suivi en temps réel de toutes les activités (inscriptions, commandes, gains, etc.).
-   **Gestion des Utilisateurs :** Valider les inscriptions, suspendre, réintégrer et consulter les profils.
-   **Gestion Financière :** Valider les retraits, gérer les portefeuilles, configurer les moyens de paiement.
-   **Gestion des Commandes :** Superviser, modifier, annuler ou assigner manuellement des commandes.
-   **Support Client :** Gérer les tickets de support.
-   **Configuration Système :** Gérer les paramètres globaux (tarifs, commissions, etc.).
-   **Journal d'Activités (Logs) :** Traçabilité complète de toutes les actions effectuées sur la plateforme.

---

## 2. Modèle Logique de Données (MLD)

Le MLD est conçu pour supporter toutes les fonctionnalités décrites avec une attention particulière à la flexibilité et la traçabilité.

-   **USER :** Entité centrale contenant les informations communes à tous les rôles.
-   **CLIENT_PROFILE, MERCHANT_PROFILE, DELIVERY_PERSON_PROFILE, PARTNER_PROFILE, ADMIN_PROFILE :** Tables spécifiques pour chaque rôle.
-   **DOCUMENT :** Stocke les fichiers justificatifs et leur statut de validation.
-   **ADDRESS :** Adresses enregistrées par les clients.
-   **DELIVERY_ORDER :** Le cœur du système, contient tous les détails d'une commande de livraison.
-   **DELIVERY_STATUS_HISTORY :** Historique de chaque changement de statut d'une commande.
-   **PAYMENT :** Enregistre toutes les transactions financières.
-   **RATING :** Notes et commentaires laissés par les clients.
-   **SYSTEM_SETTING :** Paramètres globaux de l'application.
-   **ACTIVITY_LOG :** Journal d'audit de toutes les actions.
-   ... et autres entités de support (Support, Publicité, etc.).

---

## 3. Architecture et Spécification Complète de l'API

### 3.1. Principes Généraux

-   **URL de base :** `https://api.way.com/v1/`
-   **Authentification :** JWT (JSON Web Tokens). Le token, obtenu après login, doit être inclus dans l'en-tête `Authorization` de chaque requête sécurisée (`Authorization: Bearer <token>`).
-   **Formats :** Les requêtes et réponses utilisent le format `application/json`. Les uploads de fichiers utilisent `multipart/form-data`.
-   **Gestion des Rôles :** L'API vérifie que l'utilisateur a le rôle requis pour accéder à chaque endpoint.

### 3.2. Tableau Complet des Endpoints de l'API

---
### **Module 1 : Authentification et Gestion des Utilisateurs (Commun)**
---

| Méthode | Route | Description | Traitement Backend Détaillé |
| :--- | :--- | :--- | :--- |
| **POST** | `/auth/register` | Inscription de base (numéro + rôle). | 1. Valider le format du `phone_number`. 2. Vérifier si le numéro n'existe pas déjà. 3. Générer un code OTP de 6 chiffres. 4. Stocker temporairement le hash du code OTP avec le numéro. 5. Envoyer l'OTP par SMS. 6. Pré-enregistrer l'utilisateur avec un statut `PENDING_OTP_VERIFICATION`. |
| **POST** | `/auth/verify-otp` | Vérifie le code OTP. | 1. Retrouver le hash de l'OTP stocké pour le `phone_number`. 2. Comparer avec l'OTP fourni. 3. Si OK, mettre à jour le statut de l'utilisateur à `PENDING_PROFILE_COMPLETION`. 4. Retourner un token d'enregistrement temporaire pour l'étape suivante. |
| **POST** | `/auth/complete-profile` | **NOUVEAU :** L'utilisateur soumet toutes ses informations après validation OTP. | 1. Valider le token d'enregistrement temporaire. 2. Valider toutes les données reçues (nom, email, mot de passe, etc.). 3. Hasher le mot de passe. 4. Créer l'entrée finale dans la table `USER` et la table de profil associée (`MERCHANT_PROFILE`, `CLIENT_PROFILE`, etc.). 5. Mettre le statut du compte à `PENDING_VALIDATION` (pour marchands/livreurs) ou `ACTIVE` (pour clients). |
| **POST** | `/auth/login` | Connexion d'un utilisateur. | 1. Chercher l'utilisateur par `phone_number`. 2. Vérifier le mot de passe hashé. 3. Vérifier que le statut du compte est `ACTIVE`. 4. Générer un token JWT incluant `user_id` et `role`. 5. Retourner le token et les infos de base de l'utilisateur. |
| **POST** | `/auth/forgot-password` | Demande de réinitialisation de mot de passe. | 1. Recevoir l'email/numéro de téléphone. 2. Générer un token de réinitialisation sécurisé avec une durée de vie limitée. 3. Envoyer un lien/code de réinitialisation par SMS ou email. |
| **POST** | `/auth/reset-password` | Réinitialise le mot de passe avec le token fourni. | 1. Valider le token de réinitialisation. 2. Valider et hasher le nouveau mot de passe. 3. Mettre à jour le mot de passe de l'utilisateur dans la base de données. |
| **GET** | `/users/me` | Récupérer son propre profil complet. | 1. Décoder le token JWT pour obtenir le `user_id`. 2. Récupérer les informations des tables `USER` et du profil associé (ex: `MERCHANT_PROFILE`). |
| **PUT** | `/users/me` | Mettre à jour son propre profil. | 1. Décoder le JWT. 2. Valider les données reçues. 3. Mettre à jour les informations dans la base de données. |
| **POST** | `/users/me/documents` | Uploader les pièces justificatives. | 1. Gérer l'upload de fichier (`multipart/form-data`). 2. Stocker le fichier sur un service de stockage (ex: S3). 3. Enregistrer l'URL du fichier dans la table `DOCUMENT` avec un statut `PENDING`. |

---
### **Module 2 : Fonctionnalités Marchand**
---

| Méthode | Route | Description | Traitement Backend Détaillé |
| :--- | :--- | :--- | :--- |
| **POST**| `/orders/quote` | Obtenir une estimation du prix d'une course. | 1. Recevoir les coordonnées GPS de départ et d'arrivée. 2. Calculer la distance via un service externe (ex: Google Maps API). 3. Récupérer les règles de tarification depuis `SYSTEM_SETTING` (prix de base, prix/km, majoration nuit, pluie, etc.). 4. Appliquer la formule de calcul. 5. Retourner le prix estimé. |
| **GET** | `/tariffs` | Obtenir la grille tarifaire (zones et prix). | 1. Récupérer l'ID du marchand depuis le JWT. 2. Récupérer la liste des zones de livraison et leurs prix associés, centrés sur l'adresse du marchand. |
| **POST** | `/orders` | Créer une nouvelle commande. | 1. Valider toutes les données de la commande. 2. Calculer le prix final de la livraison (similaire à `/quote`). 3. Créer l'entrée dans `DELIVERY_ORDER` avec `initiator_role: 'MERCHANT'`. 4. Mettre le statut à `SEARCHING_DELIVERY_PERSON` et lancer la recherche de livreur (notification aux livreurs proches via WebSocket). |
| **GET** | `/orders` | Lister les commandes du marchand. | 1. Récupérer `merchant_id` depuis le JWT. 2. Interroger `DELIVERY_ORDER` pour les commandes correspondantes. 3. Gérer les filtres (`?status=...`) et la pagination. |
| **PUT** | `/orders/{orderId}` | Mettre à jour une commande "rapide". | 1. Vérifier que la commande appartient bien au marchand et qu'elle est dans un statut modifiable. 2. Mettre à jour l'adresse de livraison ou le numéro du client. |
| **POST**| `/orders/{orderId}/cancel` | Annuler une commande en cours. | 1. Vérifier les droits du marchand sur la commande. 2. Changer le statut en `CANCELLED` et enregistrer la raison. 3. Notifier le livreur si déjà assigné. |

---
### **Module 3 : Fonctionnalités Client**
---

| Méthode | Route | Description | Traitement Backend Détaillé |
| :--- | :--- | :--- | :--- |
| **GET** | `/addresses` | Lister ses adresses enregistrées. | 1. Récupérer `client_id` depuis le JWT. 2. Récupérer toutes les adresses liées à ce client depuis la table `ADDRESS`. |
| **POST** | `/addresses` | Ajouter une nouvelle adresse. | 1. Valider les données de l'adresse. 2. Enregistrer la nouvelle adresse dans la table `ADDRESS` en l'associant au `client_id`. |
| **PUT** | `/addresses/{addressId}`| Modifier une adresse. | 1. Vérifier que l'adresse appartient bien au client. 2. Mettre à jour l'adresse spécifiée. |
| **DELETE**| `/addresses/{addressId}`| Supprimer une adresse. | 1. Vérifier que l'adresse appartient bien au client. 2. Supprimer l'adresse spécifiée. |
| **POST**| `/orders/{orderId}/pay` | Initier le paiement d'une commande. | 1. S'intégrer à la passerelle de paiement (Mobile Money, CB). 2. Créer une transaction dans la table `PAYMENT` avec un statut `PENDING`. 3. Mettre à jour le statut du paiement dans la commande une fois la transaction confirmée. |
| **POST** | `/orders/{orderId}/rate` | Noter une livraison (livreur/marchand). | 1. Vérifier que la commande est terminée et appartient au client. 2. Enregistrer la note et le commentaire dans la table `RATING`. |
| **GET** | `/wallet` | Consulter son portefeuille. | 1. Récupérer `user_id` depuis le JWT. 2. Lire le solde dans la table de profil (`CLIENT_PROFILE`). 3. Lister l'historique des transactions depuis la table `PAYMENT`. |
| **POST** | `/wallet/deposit` | Déposer de l'argent sur le portefeuille. | 1. Gérer la transaction via une passerelle de paiement. 2. En cas de succès, créditer le solde du portefeuille dans la table de profil. |
| **GET** | `/advertisements` | Récupérer les bannières publicitaires. | 1. Sélectionner les publicités actives où `target_audience_role` est 'CLIENT' ou 'ALL'. |

---
### **Module 4 : Fonctionnalités Livreur**
---

| Méthode | Route | Description | Traitement Backend Détaillé |
| :--- | :--- | :--- | :--- |
| **PUT** | `/riders/me/status` | Mettre à jour sa disponibilité. | 1. Mettre à jour le champ `is_available` dans `DELIVERY_PERSON_PROFILE`. |
| **PUT** | `/riders/me/location` | Le livreur envoie sa position GPS. | 1. Mettre à jour `current_latitude` et `current_longitude`. 2. Diffuser cette position via WebSocket aux clients/marchands concernés pour le suivi en temps réel. |
| **GET** | `/orders/available` | Obtenir les commandes en attente. | 1. Retourner les commandes avec le statut `SEARCHING_DELIVERY_PERSON` dans le rayon de service du livreur. |
| **POST**| `/orders/{orderId}/accept` | Un livreur accepte une course. | 1. Assigner le `delivery_person_id` à la commande. 2. Changer le statut à `ACCEPTED_BY_DELIVERY_PERSON`. 3. Notifier le marchand/client via WebSocket et/ou Push Notification. |
| **POST**| `/orders/{orderId}/refuse` | Un livreur refuse une course. | 1. Incrémenter le compteur `consecutive_refusals`. 2. Si le compteur atteint 3, mettre le statut `is_available` à `false` pour la journée. 3. Relancer le processus de recherche pour un autre livreur. |
| **PUT** | `/orders/{orderId}/status`| Le livreur met à jour le statut (récupéré/livré). | 1. Valider le changement de statut (ex: on ne peut pas passer de `ACCEPTED` à `DELIVERED` sans `PICKED_UP`). 2. Mettre à jour le statut de la commande et créer une entrée dans `DELIVERY_STATUS_HISTORY` avec la position GPS actuelle. 3. Déclencher les notifications associées. |
| **POST** | `/orders` | Créer une commande "Street Pickup". | 1. Créer la commande avec `initiator_role: 'DELIVERY_PERSON'` et `is_self_assigned_by_delivery_person: true`. |
| **POST** | `/wallet/withdraw` | Demander un retrait d'argent. | 1. Créer une demande de retrait dans la table `PAYMENT` avec le statut `PENDING_APPROVAL`. |

---
### **Module 5 : Fonctionnalités Entreprise Partenaire**
---

| Méthode | Route | Description | Traitement Backend Détaillé |
| :--- | :--- | :--- | :--- |
| **GET** | `/partners/me/employees`| Lister les livreurs employés. | 1. Récupérer `partner_id` depuis le JWT. 2. Récupérer tous les `DELIVERY_PERSON_PROFILE` où `partner_id` correspond. |
| **PUT** | `/partners/me/employees/{employeeId}`| Suspendre ou réactiver un employé. | 1. Vérifier que l'employé appartient bien au partenaire. 2. Mettre à jour le `account_status` du livreur employé. |
| **GET** | `/partners/me/orders` | Voir l'historique des commandes des employés. | 1. Agréger toutes les commandes où `assigned_partner_id` correspond au partenaire. |
| **GET** | `/partners/me/stats` | Obtenir les statistiques de l'entreprise. | 1. Agréger les données des commandes (gains, nombre de courses) liées aux employés du partenaire, avec des filtres par période. |

---
### **Module 6 : Fonctionnalités Dashboard Admin**
---

| Méthode | Route | Description | Traitement Backend Détaillé |
| :--- | :--- | :--- | :--- |
| **GET** | `/admin/stats` | Obtenir les statistiques globales. | 1. Agréger les données de toutes les tables (nombre d'utilisateurs par rôle, commandes par statut, revenus, etc.). |
| **GET** | `/admin/users` | Lister tous les utilisateurs. | 1. Récupérer tous les utilisateurs avec des options de recherche, filtrage et pagination. |
| **PUT** | `/admin/users/{userId}/status` | Valider, suspendre ou réintégrer un utilisateur. | 1. Mettre à jour le `account_status` de l'utilisateur. 2. Enregistrer cette action dans `ACTIVITY_LOG`. |
| **GET** | `/admin/documents/pending`| Obtenir les documents en attente de validation. | 1. Récupérer les documents avec le statut `PENDING`. |
| **PUT** | `/admin/documents/{documentId}/validate`| Valider ou rejeter un document. | 1. Mettre à jour le statut du document. 2. Si tous les documents d'un utilisateur sont validés, changer le statut de l'utilisateur à `ACTIVE`. |
| **GET** | `/admin/orders` | Lister toutes les commandes. | 1. Récupérer toutes les commandes avec des options de recherche et de filtrage avancées. |
| **PUT** | `/admin/orders/{orderId}`| Modifier une commande (ex: réassigner). | 1. Permettre la modification de n'importe quel champ de la commande, avec enregistrement dans les logs. |
| **GET** | `/admin/riders/locations`| Obtenir la position des livreurs en temps réel. | 1. Retourner une liste des livreurs disponibles avec leurs coordonnées GPS actuelles. |
| **GET** | `/admin/payouts/pending`| Lister les demandes de retrait en attente. | 1. Récupérer toutes les transactions de type `WALLET_WITHDRAWAL` avec le statut `PENDING_APPROVAL`. |
| **POST**| `/admin/payouts/{payoutId}/approve`| Approuver une demande de retrait. | 1. Mettre à jour le statut du retrait à `COMPLETED`. 2. Initier le transfert d'argent (action manuelle ou via API bancaire). |
| **POST**| `/admin/users/{userId}/wallet/adjust`| Créditer/débiter manuellement un portefeuille. | 1. Ajuster le solde du portefeuille et enregistrer l'opération dans `PAYMENT` et `ACTIVITY_LOG`. |
| **GET** | `/admin/settings` | Récupérer les paramètres système. | 1. Lire toutes les entrées de la table `SYSTEM_SETTING`. |
| **PUT** | `/admin/settings` | Mettre à jour les paramètres système. | 1. Mettre à jour les valeurs dans la table `SYSTEM_SETTING` (ex: `price_per_km`, `commission_rate`). |
| **GET** | `/admin/logs` | Consulter le journal d'activités. | 1. Récupérer les entrées de la table `ACTIVITY_LOG` avec filtres. |


### 3.3. Spécification Détaillée (Requêtes & Réponses)

Cette section détaille la structure des données JSON pour chaque endpoint de l'API WAY.

---
#### **Module 1 : Authentification et Gestion des Utilisateurs (Commun)**
---

#### `POST /auth/register`
-   **Rôles Autorisés :** Public
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "phone_number": "+22912345678",
      "role": "MERCHANT"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Un code de vérification a été envoyé au +22912345678."
    }
    ```

#### `POST /auth/verify-otp`
-   **Rôles Autorisés :** Public
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "phone_number": "+22912345678",
      "otp_code": "123456"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Numéro de téléphone validé. Veuillez compléter votre profil.",
      "registration_token": "UN_TOKEN_TEMPORAIRE_POUR_COMPLETER_LE_PROFIL"
    }
    ```

#### `POST /auth/complete-profile`
-   **Rôles Autorisés :** Public (avec `registration_token`)
-   **Requête d'Entrée (`application/json`)** *(Exemple pour un Marchand Particulier)*
    ```json
    {
      "personal_first_name": "Jean",
      "personal_last_name": "Dupont",
      "personal_birth_date": "1990-05-15",
      "personal_residential_address_street": "Quartier Agla, Cotonou",
      "email": "jean.dupont@email.com",
      "password": "a_strong_password_123",
      "merchant_name": "Restaurant 'Chez Jean'",
      "is_individual": true,
      "merchant_address_street": "123 Rue du Commerce, Cotonou",
      "merchant_address_latitude": 6.3616,
      "merchant_address_longitude": 2.4278
    }
    ```
-   **Réponse de Succès (201 Created)**
    ```json
    {
      "message": "Profil créé avec succès. Votre compte est en attente de validation.",
      "token": "TOKEN_ACCES_COMPLET_eyJhbG...",
      "user": {
        "user_id": "c3a2b1f0-...",
        "active_role": "MERCHANT",
        "account_status": "PENDING_VALIDATION"
      }
    }
    ```

#### `POST /auth/login`
-   **Rôles Autorisés :** Public
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "phone_number": "+22912345678",
      "password": "a_strong_password_123"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "token": "TOKEN_ACCES_COMPLET_eyJhbG...",
      "user": {
        "user_id": "c3a2b1f0-...",
        "personal_first_name": "Jean",
        "active_role": "MERCHANT",
        "account_status": "ACTIVE",
        "profile_picture_url": "https://storage.way.com/profiles/c3a2b1f0.jpg"
      }
    }
    ```

#### `POST /auth/forgot-password`
-   **Rôles Autorisés :** Public
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "phone_number": "+22912345678"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Si ce numéro est associé à un compte, un code de réinitialisation a été envoyé."
    }
    ```

#### `POST /auth/reset-password`
-   **Rôles Autorisés :** Public
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "reset_token": "UN_TOKEN_DE_RESET_RECU_PAR_SMS",
      "new_password": "un_nouveau_mot_de_passe_solide"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Votre mot de passe a été réinitialisé avec succès."
    }
    ```

#### `GET /users/me`
-   **Rôles Autorisés :** `CLIENT`, `MERCHANT`, `DELIVERY_PERSON`, `PARTNER`, `ADMIN`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)** *(Exemple pour un Marchand)*
    ```json
    {
      "user_id": "c3a2b1f0-...",
      "phone_number": "+22912345678",
      "email": "jean.dupont@email.com",
      "registration_date": "2025-08-27T10:00:00Z",
      "active_role": "MERCHANT",
      "account_status": "ACTIVE",
      "profile_picture_url": null,
      "personal_first_name": "Jean",
      "personal_last_name": "Dupont",
      "merchant_profile": {
        "merchant_name": "Restaurant 'Chez Jean'",
        "description": "Les meilleures pizzas de Cotonou.",
        "merchant_status": "OPEN",
        "average_rating": 4.5
      }
    }
    ```

#### `PUT /users/me`
-   **Rôles Autorisés :** `CLIENT`, `MERCHANT`, `DELIVERY_PERSON`, `PARTNER`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "email": "nouveau.email@example.com",
      "merchant_profile": {
        "description": "Ouvert 7j/7, livraison rapide !"
      }
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Profil mis à jour avec succès.",
      "user": {
        "user_id": "c3a2b1f0-...",
        "email": "nouveau.email@example.com",
        "merchant_profile": {
          "description": "Ouvert 7j/7, livraison rapide !"
        }
      }
    }
    ```

#### `POST /users/me/documents`
-   **Rôles Autorisés :** `MERCHANT`, `DELIVERY_PERSON`, `PARTNER`
-   **Requête d'Entrée (`multipart/form-data`)**
    -   `document_type` (String) : `ID_CARD`, `BUSINESS_REGISTRATION`, `PROOF_OF_ADDRESS`, `IFU_SCAN`
    -   `file` (File) : Le fichier image ou PDF.
-   **Réponse de Succès (201 Created)**
    ```json
    {
      "document_id": "doc_a1b2c3d4-...",
      "file_url": "https://storage.way.com/documents/doc_a1b2c3d4.pdf",
      "validation_status": "PENDING"
    }
    ```

---
#### **Module 2 : Fonctionnalités Marchand**
---

#### `POST /orders/quote`
-   **Rôles Autorisés :** `MERCHANT`, `CLIENT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "pickup_latitude": 6.3616,
      "pickup_longitude": 2.4278,
      "delivery_latitude": 6.3716,
      "delivery_longitude": 2.4378
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "estimated_fee": 1500,
      "currency": "FCFA",
      "distance_km": 4.2
    }
    ```

#### `GET /tariffs`
-   **Rôles Autorisés :** `MERCHANT`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        { "zone_name": "Zone A (0-3km)", "price": 1000 },
        { "zone_name": "Zone B (3-5km)", "price": 1500 },
        { "zone_name": "Zone C (5-8km)", "price": 2000 }
      ]
    }
    ```

#### `POST /orders` (Partagé avec Clients et Livreurs)
-   **Rôles Autorisés :** `MERCHANT`, `CLIENT`, `DELIVERY_PERSON`
-   **Requête d'Entrée (`application/json`)** *(Initiée par un Marchand)*
    ```json
    {
      "pickup_address_street": "Restaurant 'Chez Jean', 123 Rue du Commerce",
      "pickup_address_latitude": 6.3616,
      "pickup_address_longitude": 2.4278,
      "delivery_address_street": "Immeuble Kiti, 5ème étage, Ganhi",
      "delivery_address_latitude": 6.3551,
      "delivery_address_longitude": 2.4183,
      "delivery_content_description": "1x Pizza Reine, 1x Coca",
      "client_phone_at_delivery": "+22998765432"
    }
    ```
-   **Réponse de Succès (201 Created)**
    ```json
    {
      "delivery_order_id": "order_f4b3a2c1-...",
      "current_delivery_status": "SEARCHING_DELIVERY_PERSON",
      "total_delivery_fee": 1500,
      "order_date": "2025-08-27T12:30:00Z",
      "estimated_delivery_time_minutes": 25
    }
    ```
#### `GET /orders`
-   **Rôles Autorisés :** `MERCHANT`, `CLIENT`
-   **Requête d'Entrée :** Aucune (paramètres optionnels `?status=...`, `?page=...`)
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "pagination": { "currentPage": 1, "totalPages": 3, "totalItems": 25 },
      "data": [
        {
          "delivery_order_id": "order_f4b3a2c1-...",
          "current_delivery_status": "IN_DELIVERY",
          "order_date": "2025-08-27T10:30:00Z",
          "total_delivery_fee": 1500,
          "pickup_address_street": "Restaurant 'Chez Jean'",
          "delivery_address_street": "Immeuble Kiti, Ganhi"
        }
      ]
    }
    ```

#### `PUT /orders/{orderId}`
-   **Rôles Autorisés :** `MERCHANT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "delivery_address_street": "Nouvelle adresse, Rue 123, Akpakpa",
      "client_phone_at_delivery": "+22999887766"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Commande mise à jour avec succès.",
      "delivery_order_id": "order_f4b3a2c1-..."
    }
    ```

#### `POST /orders/{orderId}/cancel`
-   **Rôles Autorisés :** `MERCHANT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "cancellation_reason": "Le client a annulé par téléphone."
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "La commande a été annulée avec succès."
    }
    ```

---
#### **Module 3 : Fonctionnalités Client**
---

#### `GET /addresses`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "address_id": "addr_123",
          "address_name": "Maison",
          "street": "15 Rue des Flamboyants",
          "city": "Cotonou",
          "is_default": true
        },
        {
          "address_id": "addr_456",
          "address_name": "Bureau",
          "street": "Immeuble Ganhi Tower, 5ème étage",
          "city": "Cotonou",
          "is_default": false
        }
      ]
    }
    ```

#### `POST /addresses`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "address_name": "Travail",
      "street": "Avenue de la Marina",
      "city": "Cotonou",
      "latitude": 6.3533,
      "longitude": 2.4133,
      "is_default": false
    }
    ```
-   **Réponse de Succès (201 Created)**
    ```json
    {
      "address_id": "addr_789",
      "address_name": "Travail",
      "street": "Avenue de la Marina",
      "city": "Cotonou"
    }
    ```

#### `PUT /addresses/{addressId}`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "address_name": "Bureau Principal",
      "is_default": true
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Adresse mise à jour avec succès."
    }
    ```

#### `DELETE /addresses/{addressId}`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (204 No Content)**

#### `POST /orders/{orderId}/pay`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "payment_method": "MOBILE_MONEY"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Paiement initié. Veuillez confirmer sur votre téléphone.",
      "payment_status": "PENDING"
    }
    ```

#### `POST /orders/{orderId}/rate`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "rated_entity_type": "DELIVERY_PERSON",
      "rating_value": 5,
      "comment": "Livreur très rapide et courtois !"
    }
    ```
-   **Réponse de Succès (201 Created)**
    ```json
    { "message": "Votre évaluation a été enregistrée avec succès." }
    ```

#### `GET /wallet`
-   **Rôles Autorisés :** `CLIENT`, `DELIVERY_PERSON`, `PARTNER`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "balance": 5000,
      "currency": "FCFA",
      "transactions": [
        {
          "transaction_id": "txn_abc",
          "type": "DEPOSIT",
          "amount": 10000,
          "date": "2025-08-26T14:00:00Z",
          "status": "COMPLETED"
        },
        {
          "transaction_id": "txn_def",
          "type": "DELIVERY_PAYMENT",
          "amount": -1500,
          "date": "2025-08-27T11:00:00Z",
          "status": "COMPLETED"
        }
      ]
    }
    ```

#### `POST /wallet/deposit`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "amount": 5000,
      "payment_method": "CREDIT_CARD"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Dépôt initié avec succès.",
      "new_balance_pending": 10000
    }
    ```

#### `GET /advertisements`
-   **Rôles Autorisés :** `CLIENT`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "ad_id": "ad_xyz",
          "image_url": "https://storage.way.com/ads/promo_pizza.png",
          "target_url": "https://api.way.com/v1/merchants/merchant_123"
        }
      ]
    }
    ```
---
#### **Module 4 : Fonctionnalités Livreur**
---
#### `PUT /riders/me/status`
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "is_available": true
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Statut mis à jour avec succès.",
      "new_status": "disponible"
    }
    ```

#### `PUT /riders/me/location`
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "latitude": 6.3650,
      "longitude": 2.4250
    }
    ```
-   **Réponse de Succès (204 No Content)**

#### `GET /orders/available`
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "delivery_order_id": "order_g5h4i3-...",
          "pickup_address_street": "Pâtisserie 'Le Délice'",
          "delivery_address_street": "Clinique Mahouna",
          "total_delivery_fee": 1200
        }
      ]
    }
    ```

#### `POST /orders/{orderId}/accept`
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "message": "Course acceptée.",
      "order_details": {
        "delivery_order_id": "order_f4b3a2c1-...",
        "pickup_address_street": "Restaurant 'Chez Jean'",
        "delivery_address_street": "Immeuble Kiti, Ganhi",
        "client_phone_at_delivery": "+22998765432"
      }
    }
    ```

#### `POST /orders/{orderId}/refuse`
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    { "message": "Course refusée." }
    ```

#### `PUT /orders/{orderId}/status`
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "status": "PICKED_UP",
      "latitude": 6.3616,
      "longitude": 2.4278
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "delivery_order_id": "order_f4b3a2c1-...",
      "new_status": "PICKED_UP"
    }
    ```

#### `POST /orders` (Street Pickup)
-   **Rôles Autorisés :** `DELIVERY_PERSON`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "initiator_role": "DELIVERY_PERSON",
      "pickup_address_street": "Devant la pharmacie Camp Guezo",
      "delivery_address_street": "Stade de l'amitié",
      "total_delivery_fee": 1000,
      "client_phone_at_delivery": "+22966554433"
    }
    ```
-   **Réponse de Succès (201 Created)**
    ```json
    {
      "message": "Course 'Street Pickup' créée et assignée.",
      "delivery_order_id": "order_j9k8l7-..."
    }
    ```
#### `POST /wallet/withdraw`
-   **Rôles Autorisés :** `DELIVERY_PERSON`, `PARTNER`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "amount": 25000,
      "withdrawal_method": "MOBILE_MONEY_AIRTEL"
    }
    ```
-   **Réponse de Succès (202 Accepted)**
    ```json
    {
      "message": "Votre demande de retrait a été soumise et est en cours de traitement."
    }
    ```
---
#### **Module 5 : Fonctionnalités Entreprise Partenaire**
---
#### `GET /partners/me/employees`
-   **Rôles Autorisés :** `PARTNER`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "user_id": "user_rider_1",
          "personal_first_name": "Ali",
          "phone_number": "+22997112233",
          "account_status": "ACTIVE",
          "is_available": true
        }
      ]
    }
    ```
#### `PUT /partners/me/employees/{employeeId}`
-   **Rôles Autorisés :** `PARTNER`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "account_status": "SUSPENDED",
      "reason": "Non-respect des consignes."
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    { "message": "Le statut du livreur a été mis à jour." }
    ```

#### `GET /partners/me/orders`
-   **Rôles Autorisés :** `PARTNER`
-   **Requête d'Entrée :** Aucune (paramètres `?employeeId=...`, `?date=...`)
-   **Réponse de Succès (200 OK)** : Similaire à `GET /orders` mais agrégé pour les employés.

#### `GET /partners/me/stats`
-   **Rôles Autorisés :** `PARTNER`
-   **Requête d'Entrée :** Aucune (paramètres `?period=daily`, `?employeeId=...`)
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "total_earnings": 150000,
      "total_orders": 85,
      "earnings_by_employee": [
        { "employee_id": "user_rider_1", "name": "Ali", "earnings": 75000 }
      ]
    }
    ```
---
#### **Module 6 : Fonctionnalités Dashboard Admin**
---
#### `GET /admin/stats`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "total_users": 520,
      "users_by_role": { "CLIENT": 400, "MERCHANT": 80, "DELIVERY_PERSON": 40 },
      "orders_today": 150,
      "revenue_today": 75000
    }
    ```
#### `GET /admin/users`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune (paramètres `?role=...`, `?status=...`, `?search=...`)
-   **Réponse de Succès (200 OK)** : Similaire à `GET /orders` mais pour les utilisateurs.

#### `PUT /admin/users/{userId}/status`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "account_status": "ACTIVE",
      "reason": "Documents vérifiés et approuvés."
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    { "message": "Statut de l'utilisateur mis à jour." }
    ```
#### `GET /admin/documents/pending`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "document_id": "doc_xyz",
          "user_id": "user_abc",
          "user_name": "Nouveau Marchand",
          "document_type": "BUSINESS_REGISTRATION",
          "file_url": "https://storage.way.com/documents/doc_xyz.pdf",
          "upload_date": "2025-08-27T14:00:00Z"
        }
      ]
    }
    ```

#### `PUT /admin/documents/{documentId}/validate`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "status": "REJECTED",
      "rejection_reason": "Le document est illisible."
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    { "message": "Le statut du document a été mis à jour." }
    ```
#### `GET /admin/orders`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune (paramètres de filtre avancés)
-   **Réponse de Succès (200 OK)** : Similaire à `GET /orders` mais avec toutes les commandes.

#### `PUT /admin/orders/{orderId}`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "assigned_delivery_person_id": "new_rider_id",
      "total_delivery_fee": 1800
    }
    ```
-   **Réponse de Succès (200 OK)** : L'objet commande mis à jour.

#### `GET /admin/riders/locations`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "rider_id": "rider_123",
          "name": "Ali",
          "latitude": 6.3650,
          "longitude": 2.4250,
          "last_update": "2025-08-27T15:00:00Z"
        }
      ]
    }
    ```
#### `GET /admin/payouts/pending`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "payout_id": "payout_abc",
          "user_id": "user_rider_1",
          "user_name": "Ali",
          "amount": 25000,
          "request_date": "2025-08-27T09:00:00Z"
        }
      ]
    }
    ```

#### `POST /admin/payouts/{payoutId}/approve`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "transaction_reference": "REF_MOBILE_MONEY_12345"
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    { "message": "La demande de retrait a été approuvée." }
    ```
#### `POST /admin/users/{userId}/wallet/adjust`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
        "adjustment_type": "CREDIT",
        "amount": 5000,
        "reason": "Bonus commercial pour le lancement."
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    {
        "message": "Le portefeuille de l'utilisateur a été ajusté avec succès.",
        "new_balance": 5000
    }
    ```
#### `GET /admin/settings`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        { "setting_key": "price_per_km", "setting_value": "300", "description": "Prix par kilomètre en FCFA." },
        { "setting_key": "commission_rate", "setting_value": "0.15", "description": "Commission de la plateforme (15%)." }
      ]
    }
    ```
#### `PUT /admin/settings`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée (`application/json`)**
    ```json
    {
      "settings": [
        { "setting_key": "price_per_km", "setting_value": "350" }
      ]
    }
    ```
-   **Réponse de Succès (200 OK)**
    ```json
    { "message": "Les paramètres ont été mis à jour." }
    ```
#### `GET /admin/logs`
-   **Rôles Autorisés :** `ADMIN`
-   **Requête d'Entrée :** Aucune (paramètres de filtre `?userId=...`, `?action_type=...`)
-   **Réponse de Succès (200 OK)**
    ```json
    {
      "data": [
        {
          "log_id": "log_123",
          "user_id": "admin_user",
          "action_type": "ADMIN_SUSPEND_USER",
          "details_json": { "target_user_id": "user_to_suspend", "reason": "Fraude" },
          "action_timestamp": "2025-08-27T16:00:00Z"
        }
      ]
    }
    ```
