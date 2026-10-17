# Open Payments, Payment Pointer et GNAP — guide d’intégration

> Synthèse pratique et liens vers les spécifications officielles. Les exigences normatives évoluent ; valider les versions en vigueur avant toute mise en production.

## Sources officielles

- [Open Payments](https://openpayments.dev/)
- [Dépôt des spécifications Open Payments (OpenAPI)](https://github.com/interledger/open-payments-specifications)
- [Documentation Rafiki — concepts Open Payments](https://rafiki.dev/overview/concepts/open-payments)
- [Documentation Rafiki — API Admin GraphQL](https://rafiki.dev/apis/graphql/admin-api-overview)
- [GNAP — spécification IETF](https://datatracker.ietf.org/doc/draft-ietf-gnap-core-protocol/)
- [Payment Pointers](https://paymentpointers.org/)

## Composants Open Payments

Open Payments est un standard d’API qui permet à des clients autorisés de découvrir des ressources de paiement, demander un devis, initier des paiements et consulter les ressources associées. Les spécifications couvrent notamment :

- **Wallet Address Server** : expose les informations publiques d’une adresse de portefeuille.
- **Resource Server** : expose les ressources et opérations de paiement associées à un compte.
- **Authorization Server** : délivre des autorisations limitées, via le modèle GNAP, pour accéder aux ressources.

Rafiki fournit une implémentation de référence de ces fonctions, notamment via ses services `backend` et `auth`.

## Payment Pointer

Un Payment Pointer est un identifiant lisible commençant généralement par `$` et résolu en une adresse HTTPS. Il sert de point de découverte ; il ne constitue pas à lui seul une preuve d’autorisation ni un transfert de fonds. Après résolution, le client suit le profil et les métadonnées publiés par le service, puis effectue les étapes d’autorisation et de paiement prévues.

## Flux de paiement typique

1. Découvrir la ressource du portefeuille ou l’adresse cible.
2. Lire les informations nécessaires pour déterminer les opérations prises en charge.
3. Créer ou obtenir un **quote** (devis) lorsque le flux le requiert. Le devis fixe les paramètres de paiement, notamment les montants et les frais applicables selon la réponse de l’API.
4. Obtenir le consentement de l’utilisateur et un grant GNAP limité aux ressources et actions nécessaires.
5. Créer ou poursuivre l’incoming/outgoing payment selon le scénario.
6. Vérifier l’état de la ressource et traiter les webhooks/événements lorsque configurés. Ne pas déduire la réussite du simple fait que la requête HTTP initiale a répondu.
7. Enregistrer les identifiants de corrélation nécessaires au rapprochement et gérer les retries sans double débit.

Les endpoints et schémas exacts doivent être tirés de la version des fichiers OpenAPI et de la documentation Rafiki correspondant au déploiement.

## GNAP : principes de sécurité

GNAP (Grant Negotiation and Authorization Protocol) permet au client de demander un grant et au serveur d’évaluer l’autorisation. Les jetons d’accès doivent être limités à la portée, aux ressources et à la durée nécessaires. L’application doit :

- obtenir un consentement explicite et compréhensible de l’utilisateur ;
- ne jamais demander des droits plus larges que nécessaire ;
- stocker les jetons uniquement côté serveur ou dans un stockage sécurisé adapté ;
- ne jamais exposer les secrets de client ou jetons dans le navigateur, les URL ou les logs ;
- gérer l’expiration, la révocation, les erreurs et les interactions d’autorisation ;
- vérifier les signatures et les exigences de liaison de clé imposées par la configuration ;
- demander une nouvelle autorisation lorsque le grant existant ne couvre pas l’opération.

## Quotes et idempotence

Un quote est une ressource de tarification/paramétrage, pas une preuve que les fonds ont été transférés. Les quotes peuvent expirer et être soumis à des conditions de montant, de frais ou de routage. Respecter leur durée de validité et les champs renvoyés par l’API.

Les requêtes de paiement doivent être conçues pour résister aux répétitions, aux délais réseau et aux réponses perdues. Utiliser les mécanismes d’idempotence ou les identifiants de ressources définis par l’API ; ne jamais créditer deux fois sur la seule base de deux requêtes entrantes.

## Implémentation dans Rafiki

- Le service `backend` expose les ressources Open Payments et s’appuie sur le connecteur ILP pour les paiements.
- Le service `auth` fournit l’autorisation GNAP.
- Les API d’administration servent à configurer les actifs, pairs, adresses et paramètres de l’instance.
- L’application métier doit conserver une séparation claire entre authentification de l’utilisateur, consentement, grant GNAP, état du paiement et écriture comptable.

## Tests minimums

Tester les grants insuffisants/expirés/révoqués, consentement refusé, quote expiré, montant ou actif non pris en charge, paiement en attente, paiement rejeté, webhook répété, retry réseau et accès à une ressource appartenant à un autre compte.
