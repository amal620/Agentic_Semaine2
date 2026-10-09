# Car Rental RAG Assistant

## Présentation

**Car Rental RAG Assistant** est un prototype d’assistant intelligent destiné au support client d’une agence de location de voitures.

Il utilise une architecture **Retrieval-Augmented Generation (RAG)** pour rechercher des informations dans des documents PDF, puis générer des réponses fondées sur les passages récupérés.

Lorsque l’information est absente, contradictoire ou insuffisante, le système doit demander une vérification humaine au lieu d’inventer une réponse.

> **Statut : Prototype — des fonctionnalités de sécurité et d’escalade restent à implémenter et à tester.**

## Objectifs

* Répondre aux questions des clients à partir des documents de référence.
* Retrouver les informations pertinentes grâce à la recherche vectorielle.
* Générer des réponses avec un modèle de langage via OpenRouter.
* Citer les documents sources lorsque les informations sont disponibles.
* Limiter la divulgation des données personnelles et des informations internes.
* Prévoir une escalade humaine lorsque la réponse ne peut pas être établie.

## Architecture

Le workflow repose sur les étapes suivantes :

1. Chargement et extraction du texte des documents PDF.
2. Découpage du texte en segments (*chunks*).
3. Génération des embeddings avec Google Gemini.
4. Stockage des segments et recherche vectorielle dans MongoDB Atlas.
5. Recherche et classement des passages pertinents.
6. Transmission de la question et du contexte au modèle via OpenRouter.
7. Génération d’une réponse avec ses sources, ou émission d’un message d’escalade.

## Technologies utilisées

| Technologie              | Rôle                                           |
| ------------------------ | ---------------------------------------------- |
| Langflow                 | Conception et orchestration du workflow        |
| Python                   | Traitement et classement des résultats         |
| MongoDB Atlas            | Stockage des passages et recherche vectorielle |
| Google Gemini Embeddings | Transformation du texte en vecteurs            |
| OpenRouter               | Accès au modèle de langage                     |
| PDF                      | Documents de référence                         |

## Corpus documentaire

| Fichier                   | Contenu                                                                        | Visibilité prévue  |
| ------------------------- | ------------------------------------------------------------------------------ | ------------------ |
| `01_rental_agreement.pdf` | Conditions de location, âge minimum, permis, retards et carburant              | Client             |
| `02_insurance_terms.pdf`  | Couverture, franchises, exclusions et options d’assurance                      | Client             |
| `03_pricing_table.pdf`    | Tarifs journaliers et hebdomadaires, suppléments et kilomètres supplémentaires | Client             |
| `04_fleet_specs.pdf`      | Caractéristiques des véhicules disponibles                                     | Client             |
| `05_faq_procedures.pdf`   | FAQ client et procédures internes                                              | À séparer          |
| `06_internal_margins.pdf` | Coûts, marges et codes de remise                                               | Interne uniquement |

Les données tarifaires utilisées pour la démonstration sont synthétiques et ne représentent pas des prix commerciaux réels.

## Configuration actuelle

Les paramètres documentés du workflow exporté sont :

* **Base MongoDB :** `Agentic_AI`
* **Collection :** `DataBase`
* **Index vectoriel :** `location`
* **Modèle d’embeddings :** `models/gemini-embedding-001`
* **Dimension configurée :** `768`
* **Nombre de résultats récupérés :** `10`
* **Taille des chunks :** environ `1 000` caractères
* **Chevauchement :** environ `200` caractères
* **Modèle de langage :** `openrouter/auto`
* **Limite de sortie :** `768` tokens
* **Limite de contexte prévue :** `14 000` caractères

Ces paramètres doivent être vérifiés avec le workflow exporté et la configuration réelle de MongoDB Atlas avant l’exécution.

## Exemple d’utilisation

**Question :**

`What is the daily rental price for the Clio 2024?`

**Réponse attendue :**

`The Clio 2024 costs 45 EUR per day. [03_pricing_table.pdf, PRICING TABLE]`

Cette réponse est un exemple de démonstration basé sur le corpus synthétique.

## Sécurité

Le prototype prévoit des règles de sécurité visant à :

* Empêcher la divulgation des informations personnelles d’autres clients.
* Limiter l’accès aux documents internes.
* Éviter les réponses non justifiées par les documents.
* Prévoir une vérification humaine lorsque l’information est absente ou contradictoire.

Les prompts ne remplacent pas les mécanismes réels de contrôle d’accès.

Les documents internes doivent être séparés ou protégés par des filtres d’accès au niveau de la base de données. Le filtrage par nom de fichier seul n’est pas une garantie de sécurité.

## Limites et améliorations prévues

Les fonctionnalités suivantes restent à réaliser ou à valider :

* Métadonnées structurées et filtres stricts par véhicule, année, catégorie et visibilité.
* Recherche hybride BM25 et vectorielle.
* Intégration d’un véritable reranker.
* Détection et masquage des données personnelles (PII).
* Gestion des rôles client, agent et administrateur à partir d’une identité authentifiée.
* Calcul d’un score de confiance et définition d’un seuil d’escalade à `0.75`.
* Création automatique d’un ticket ou transfert effectif vers une équipe humaine.
* Séparation des documents clients et internes.
* Tests des citations, des injections de prompt, des données personnelles et des accès non autorisés.
* Complétion du corpus pour les sujets non documentés, notamment les annulations et les pannes.

## Installation et lancement

1. Ouvrir le fichier JSON du workflow dans Langflow.
2. Configurer les identifiants MongoDB Atlas, Gemini et OpenRouter dans les composants correspondants.
3. Vérifier l’URI MongoDB, la base, la collection, l’index vectoriel et la dimension des embeddings.
4. Charger et indexer les documents PDF.
5. Exécuter le workflow avec les questions de test.
6. Vérifier les réponses, les citations et les conditions d’escalade.

Les étapes exactes dépendent de la configuration du workflow exporté.

## Sécurité des identifiants

Ne jamais enregistrer les clés API, les mots de passe ou les URI contenant des identifiants dans le dépôt Git.

Utiliser des variables d’environnement ou un mécanisme de gestion de secrets adapté.

## Tests recommandés

* Prix journalier de la Clio 2024.
* Prix hebdomadaire de la Clio 2024.
* Franchise d’assurance pour un SUV.
* Comparaison entre la Clio 2024 et la Tesla Model 3 2025.
* Demande d’informations personnelles concernant un autre client.
* Demande d’une politique d’annulation absente des documents.
* Demande de marges internes ou de codes de remise.

Chaque test doit vérifier la réponse, la source documentaire, le respect des règles de sécurité et le comportement d’escalade.

## Avertissement

Ce projet est un prototype technique. Il ne doit pas être utilisé avec de vraies données clients avant la mise en place et la validation des contrôles d’accès, de la protection des données personnelles et des mécanismes de sécurité.

## Licence

À définir selon les conditions de distribution du projet.
