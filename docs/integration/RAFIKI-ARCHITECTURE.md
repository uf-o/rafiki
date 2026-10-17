# Architecture Rafiki — guide de déploiement et d’intégration

## Documentation officielle

- [Architecture Rafiki](https://rafiki.dev/v1-beta/resources/architecture/)
- [Service backend](https://rafiki.dev/integration/deployment/services/backend-service/)
- [Service auth](https://rafiki.dev/integration/deployment/services/auth-service/)
- [Déploiement Rafiki](https://rafiki.dev/integration/deployment/)
- [Code source Rafiki](https://github.com/interledger/rafiki)
- [TigerBeetle — documentation officielle](https://docs.tigerbeetle.com/)
- [Open Payments specifications](https://github.com/interledger/open-payments-specifications)

## Vue d’ensemble

Rafiki est un monorepo de services conçu pour permettre à un prestataire de services financiers (FSP) de fournir des paiements Interledger et Open Payments. Les services principaux sont :

- **backend** : logique métier, API Open Payments, connecteur ILP/STREAM, suivi des comptes et de la liquidité, API d’administration GraphQL.
- **auth** : serveur d’autorisation de référence conforme au modèle GNAP et gestion des grants, jetons et interactions.
- **frontend / Rafiki Admin** : interface d’administration facultative.

## Dépendances de données

L’architecture documentée utilise typiquement :

- **PostgreSQL pour auth** : grants, jetons et interactions d’autorisation.
- **Redis pour auth** : données de session.
- **PostgreSQL pour backend** : ressources Open Payments et données applicatives.
- **TigerBeetle** : écritures et soldes comptables haute performance.
- **Redis pour backend** : cache/partage de données de connexion STREAM entre processus.

Les bases PostgreSQL de `auth` et `backend` sont des responsabilités distinctes ; ne les fusionnez pas sans examiner les contraintes et la configuration de la version déployée. Rafiki documente aussi un mode comptable PostgreSQL alternatif, tandis que TigerBeetle est sélectionné par configuration dans les versions qui le prennent en charge.

## TigerBeetle et comptabilité

TigerBeetle est une base spécialisée dans les transactions financières et le suivi des comptes. Il ne remplace pas les ressources métier Open Payments ni l’ensemble de PostgreSQL. Dans Rafiki, il sert au moteur de comptabilité lorsque l’option correspondante est activée.

Principes d’intégration :

- Considérer les écritures comptables comme une source de vérité distincte des états UI.
- Ne jamais modifier un solde en écrivant directement dans une table métier ou dans le navigateur.
- Faire passer les mouvements par le chemin comptable supporté par Rafiki.
- Garantir l’unicité/idempotence des opérations de crédit et débit au niveau applicatif.
- Prévoir rapprochement, audit, alertes et procédures de correction explicites.
- Lire les exigences TigerBeetle concernant les identifiants de transfert, comptes, contraintes de débit/crédit et reprise avant d’écrire du code comptable.

## Configuration backend : points à vérifier

La documentation backend liste notamment :

- `OPERATOR_TENANT_ID` : UUID v4 de l’opérateur.
- `ADMIN_API_SECRET` : secret fort utilisé pour l’API d’administration.
- `REDIS_URL` : connexion au Redis du backend.
- `USE_TIGERBEETLE` : active TigerBeetle quand cette option existe dans la version déployée.
- `OPEN_PAYMENTS_URL`, `ILP_ADDRESS`, `ILP_CONNECTOR_URL`, `KEY_ID` et les paramètres associés selon le déploiement.
- URL/identifiants de la base PostgreSQL du backend, et paramètres distincts de la base PostgreSQL du service auth.

Les noms et valeurs exacts doivent être validés dans les fichiers de configuration et exemples de la version du dépôt. Ne pas copier des valeurs de production ou des secrets dans la documentation.

## Sécurité et réseau

- Exposer uniquement les endpoints publics nécessaires ; placer les API d’administration derrière un contrôle d’accès réseau.
- Utiliser TLS en production et des secrets aléatoires, distincts par environnement.
- Les appels Admin GraphQL doivent inclure le contexte de tenant attendu et une signature HMAC-SHA-256 générée à partir du secret API du tenant.
- Accepter les paquets ILP entrants uniquement des pairs configurés, avec le jeton d’authentification attendu.
- Isoler les services et bases de données sur le réseau, restreindre les permissions, sauvegarder les données et tester la restauration.
- Ne pas commettre les fichiers `.env`, clés, jetons, mots de passe ou dumps contenant des données personnelles.
- Mettre en place des contrôles d’accès, des limites de débit, des journaux sans secrets et des alertes opérationnelles.

## Développement local

Le README du dépôt décrit les prérequis Git, Docker et NVM, l’installation de Node.js via `nvm`, l’activation de Corepack, l’installation des dépendances via `pnpm i`, puis les commandes de build et de test. Le dépôt utilise le sous-module `open-payments-specifications/` ; après clonage, exécuter :

```sh
git submodule update --init --recursive
```

Commandes utiles indiquées par le README :

```sh
pnpm -r build
pnpm --filter backend build
pnpm --filter backend test
pnpm -r --workspace-concurrency=1 test
pnpm checks
```

## Parcours d’intégration recommandé

1. Faire démarrer l’environnement local et ses dépendances.
2. Configurer les actifs, le tenant opérateur et les comptes de test.
3. Configurer un pair ILP de test et vérifier le flux Prepare/Fulfill/Reject.
4. Créer une adresse de portefeuille et tester la découverte.
5. Tester les grants GNAP et les scénarios Open Payments avec les spécifications de la version utilisée.
6. Vérifier les écritures comptables, la reprise après erreur et le rapprochement.
7. Effectuer une revue sécurité/réglementaire avant tout usage avec de l’argent réel.

## Limite importante

Rafiki est conçu pour des prestataires de services financiers et ne doit pas être utilisé en production par une plateforme non réglementée. Avant de connecter des fonds réels, confirmer le cadre réglementaire applicable, les obligations KYC/AML/CFT, le filtrage des sanctions, la prévention de la fraude, la gouvernance des clés et les exigences du partenaire de règlement.
