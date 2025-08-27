# Documentation Complète du Projet et de l'API WAY (Version Corrigée et Complétée)

Ce document sert de référence centrale pour l'application WAY. Il contient le cahier des charges fonctionnel, le modèle logique de données (MLD) et la spécification technique détaillée de l'API RESTful.

## 1. Cahier des Charges Fonctionnel

### 1.1. Introduction de l'Application WAY

WAY est une plateforme de mise en relation pour la livraison de repas et de colis, connectant des clients, des marchands (restaurants/commerçants) et des livreurs. Le système se compose de plusieurs applications :
-   Application Client
-   Application Marchand/Restaurant
-   Application Livreur
-   Back Office (Administration)

### 1.2. Fonctionnalités Principales (Résumé)

-   **Clients :** Gestion de compte, commande (immédiate ou programmée), suivi en temps réel, paiement multi-options (Cash, Mobile Money, CB, Portefeuille), historique et support.
-   **Marchands :** Inscription validée, gestion des commandes, lancement de demandes de livreurs, consultation de la grille tarifaire par zone, suivi des livraisons.
-   **Livreurs (Indépendants & Employés) :** Inscription validée, gestion de la disponibilité, acceptation/refus des courses, mise à jour des statuts de livraison, portefeuille de gains.
-   **Entreprises Partenaires :** Gestion de leur flotte de livreurs employés, suivi des performances et des gains consolidés.
-   **Administration :** Tableau de bord global, gestion complète des utilisateurs, des commandes et des finances, configuration des paramètres système (prix, commissions), journal d'activités pour une traçabilité totale.

---

## 2. Modèle Logique de Données (MLD)

Le MLD est conçu pour supporter toutes les fonctionnalités décrites avec une attention particulière à la flexibilité et la traçabilité.

-   **USER :** Entité centrale contenant les informations communes (téléphone, email, mot de passe, rôle actif, statut du compte).
-   **PROFILES (CLIENT, MERCHANT, DELIVERY_PERSON, PARTNER, ADMIN) :** Tables spécifiques pour chaque rôle, contenant les informations qui leur sont propres.
-   **DOCUMENT :** Stocke les fichiers justificatifs (CNI, IFU, etc.) et leur statut de validation.
-   **DELIVERY_ORDER :** Le cœur du système, contient tous les détails d'une commande de livraison, de l'initiateur au client final, en passant par les adresses et les frais.
-   **DELIVERY_STATUS_HISTORY :** Historique de chaque changement de statut d'une commande pour un suivi précis.
-   **PAYMENT :** Enregistre toutes les transactions financières (paiement de course, dépôt/retrait portefeuille).
-   **SYSTEM_SETTING :** Paramètres globaux de l'application (prix/km, prix de base, majoration nuit/pluie, commission).
-   **ACTIVITY_LOG :** Journal d'audit de toutes les actions sensibles effectuées sur la plateforme.

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
