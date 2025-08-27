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

## 3. Architecture et Spécification de l'API

### 3.1. Principes Généraux

-   **URL de base :** `https://api.way.com/v1/`
-   **Authentification :** JWT (JSON Web Tokens). Le token contient le `user_id` et le `active_role`.
-   **Gestion des Rôles :** L'API vérifie que l'utilisateur a le rôle requis pour accéder à un endpoint.
-   **Communication Temps Réel :** Les WebSockets sont utilisés pour le suivi GPS, la messagerie instantanée et les notifications de nouvelles courses.

### 3.2. Tableau Récapitulatif des Endpoints

| Méthode | Route | Description | Rôles Autorisés |
| :--- | :--- | :--- | :--- |
| **POST** | `/auth/register` | Inscription de base (numéro + rôle). | Public |
| **POST** | `/auth/verify-otp` | Vérification du code OTP. | Public |
| **POST** | `/auth/login` | Connexion d'un utilisateur. | Public |
| **GET** | `/users/me` | Récupérer son propre profil. | Tous (authentifié) |
| **PUT** | `/users/me` | Mettre à jour son propre profil. | Tous (authentifié) |
| **POST** | `/users/me/documents` | Uploader un document justificatif. | `MERCHANT`, `DELIVERY_PERSON` |
| **POST** | `/orders` | Créer une nouvelle commande. | `CLIENT`, `MERCHANT` |
| **GET** | `/orders` | Lister ses commandes. | `CLIENT`, `MERCHANT` |
| **GET** | `/orders/{orderId}` | Obtenir les détails d'une commande. | `CLIENT`, `MERCHANT` |
| **POST** | `/orders/{orderId}/cancel` | Annuler une commande. | `MERCHANT` |
| **POST** | `/orders/{orderId}/rate` | Noter une livraison. | `CLIENT` |
| **GET** | `/addresses` | Lister ses adresses enregistrées. | `CLIENT` |
| **POST** | `/addresses` | Ajouter une nouvelle adresse. | `CLIENT` |
| **GET** | `/wallet` | Consulter son portefeuille. | `CLIENT`, `DELIVERY_PERSON` |
| **PUT** | `/riders/me/status` | Mettre à jour sa disponibilité. | `DELIVERY_PERSON` |
| **PUT** | `/riders/me/location` | Envoyer sa position GPS. | `DELIVERY_PERSON` |
| **GET** | `/orders/available` | Lister les commandes en attente. | `DELIVERY_PERSON` |
| **POST** | `/orders/{orderId}/accept` | Accepter une course. | `DELIVERY_PERSON` |
| **PUT** | `/orders/{orderId}/status` | Mettre à jour le statut d'une course. | `DELIVERY_PERSON` |
| **POST**| `/orders/street-pickup`| Créer une commande "Street Pickup".| `DELIVERY_PERSON` |
| **GET**| `/admin/stats` | Obtenir les statistiques globales. | `ADMIN` |
| **GET**| `/admin/users` | Lister tous les utilisateurs. | `ADMIN` |
| **PUT** | `/admin/users/{userId}/status` | Valider ou suspendre un utilisateur. | `ADMIN` |
| **GET**| `/admin/orders` | Lister toutes les commandes. | `ADMIN` |
| **PUT** | `/admin/orders/{orderId}`| Modifier une commande. | `ADMIN` |
| **PUT** | `/admin/settings` | Mettre à jour les paramètres système. | `ADMIN` |
| **GET**| `/admin/logs` | Consulter le journal d'activités. | `ADMIN` |

### 3.3. Spécification Détaillée des Endpoints

---
#### **Module 1 : Authentification et Utilisateurs**
---

#### `POST /auth/register`
-   **Description :** Lance le processus d'inscription en envoyant un code OTP au numéro fourni.
-   **Rôles Autorisés :** Public.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "phone_number": "+22912345678",
      "role": "MERCHANT"
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "message": "Un code de vérification a été envoyé au +22912345678."
    }
    ```

#### `POST /auth/verify-otp`
-   **Description :** Vérifie le code OTP pour valider le numéro et permettre de compléter le profil.
-   **Rôles Autorisés :** Public.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "phone_number": "+22912345678",
      "otp_code": "123456"
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "message": "Numéro de téléphone validé. Veuillez compléter votre profil.",
      "registration_token": "UN_TOKEN_TEMPORAIRE_POUR_COMPLETER_LE_PROFIL"
    }
    ```

#### `POST /auth/login`
-   **Description :** Connecte un utilisateur existant.
-   **Rôles Autorisés :** Public.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "phone_number": "+22912345678",
      "password": "a_strong_password_123"
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "user": {
        "user_id": "c3a2b1f0-...",
        "personal_first_name": "Jean",
        "active_role": "MERCHANT",
        "account_status": "ACTIVE"
      }
    }
    ```

---
#### **Module 2 : Marchands et Clients (Gestion des commandes)**
---

#### `POST /orders`
-   **Description :** Crée une nouvelle commande de livraison (initiée par un Client ou un Marchand).
-   **Rôles Autorisés :** `CLIENT`, `MERCHANT`.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "pickup_address_street": "Restaurant 'Chez Fifi', Cadjehoun",
      "pickup_address_latitude": 6.35,
      "pickup_address_longitude": 2.38,
      "delivery_address_id": "addr_1a2b3c",
      "delivery_content_description": "1x Pizza Reine, 1x Coca",
      "client_phone_at_delivery": "+22998765432"
    }
    ```
-   **Réponse de Succès (201 Created) :**
    ```json
    {
      "delivery_order_id": "order_f4b3a2c1-...",
      "current_delivery_status": "SEARCHING_DELIVERY_PERSON",
      "total_delivery_fee": 1500,
      "estimated_delivery_time": "25 minutes"
    }
    ```

#### `GET /orders`
-   **Description :** Récupère la liste des commandes pour l'utilisateur authentifié.
-   **Rôles Autorisés :** `CLIENT`, `MERCHANT`.
-   **Paramètres de Requête (Query Params) :** `?status=in_progress`, `?page=1`, `?limit=10`
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "pagination": { "currentPage": 1, "totalPages": 3, "totalItems": 25 },
      "data": [
        {
          "delivery_order_id": "order_f4b3a2c1-...",
          "current_delivery_status": "IN_DELIVERY",
          "order_date": "2025-08-27T10:30:00Z",
          "total_delivery_fee": 1500
        }
      ]
    }
    ```
    
#### `POST /orders/{orderId}/rate`
-   **Description :** Permet à un client de noter une livraison terminée.
-   **Rôles Autorisés :** `CLIENT`.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "rated_entity_type": "DELIVERY_PERSON",
      "rating_value": 5,
      "comment": "Livreur très rapide et courtois !"
    }
    ```
-   **Réponse de Succès (201 Created) :**
    ```json
    {
      "message": "Votre évaluation a été enregistrée avec succès."
    }
    ```

---
#### **Module 3 : Livreurs (Gestion des courses)**
---

#### `PUT /riders/me/status`
-   **Description :** Met à jour le statut de disponibilité du livreur.
-   **Rôles Autorisés :** `DELIVERY_PERSON`.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "is_available": true
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "message": "Statut mis à jour avec succès.",
      "new_status": "disponible"
    }
    ```

#### `POST /orders/{orderId}/accept`
-   **Description :** Un livreur accepte une course qui lui est proposée.
-   **Rôles Autorisés :** `DELIVERY_PERSON`.
-   **Corps de la Requête :** Aucun.
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "message": "Course acceptée.",
      "order_details": {
        "delivery_order_id": "order_f4b3a2c1-...",
        "pickup_address_street": "Restaurant 'Chez Fifi', Cadjehoun",
        "delivery_address_street": "15 Rue des Flamboyants, Cotonou",
        "client_phone_at_delivery": "+22998765432"
      }
    }
    ```

#### `PUT /orders/{orderId}/status`
-   **Description :** Le livreur met à jour le statut d'une course qu'il gère.
-   **Rôles Autorisés :** `DELIVERY_PERSON`.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "status": "PICKED_UP",
      "latitude": 6.35,
      "longitude": 2.38
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "delivery_order_id": "order_f4b3a2c1-...",
      "new_status": "PICKED_UP"
    }
    ```

---
#### **Module 4 : Administration**
---

#### `GET /admin/users`
-   **Description :** Récupère la liste de tous les utilisateurs avec des filtres.
-   **Rôles Autorisés :** `ADMIN`.
-   **Paramètres de Requête (Query Params) :** `?role=DELIVERY_PERSON&account_status=PENDING_VALIDATION`
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "pagination": { "currentPage": 1, "totalPages": 1, "totalItems": 5 },
      "data": [
        {
          "user_id": "user_d1e2f3-...",
          "personal_first_name": "Paul",
          "phone_number": "+22955667788",
          "account_status": "PENDING_VALIDATION",
          "registration_date": "2025-08-26T15:00:00Z"
        }
      ]
    }
    ```

#### `PUT /admin/users/{userId}/status`
-   **Description :** Valide, suspend ou réintègre un compte utilisateur.
-   **Rôles Autorisés :** `ADMIN`.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
      "account_status": "ACTIVE",
      "reason": "Documents vérifiés et approuvés."
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
      "message": "Le statut de l'utilisateur a été mis à jour.",
      "user_id": "user_d1e2f3-...",
      "new_status": "ACTIVE"
    }
    ```
    
#### `POST /admin/users/{userId}/wallet/adjust`
-   **Description :** Permet à un admin de créditer ou débiter manuellement le portefeuille d'un utilisateur.
-   **Rôles Autorisés :** `ADMIN`.
-   **Corps de la Requête (`application/json`) :**
    ```json
    {
        "adjustment_type": "CREDIT",
        "amount": 5000,
        "reason": "Bonus commercial pour le lancement."
    }
    ```
-   **Réponse de Succès (200 OK) :**
    ```json
    {
        "message": "Le portefeuille de l'utilisateur a été ajusté avec succès.",
        "new_balance": 5000
    }
    ```
