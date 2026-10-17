# Documentation externe complète — sources de référence

Ce dossier sert de manifeste pour conserver et retrouver les sources complètes utilisées pour construire la solution. Les documents synthétiques sous `docs/integration/` ne remplacent pas les sources primaires listées ci-dessous.

## 1. Documentation complète de Rafiki — déjà incluse dans ce dépôt

Les sources qui alimentent https://rafiki.dev/ sont présentes dans le dépôt Rafiki :

- `packages/documentation/src/content/docs/` — pages Markdown/MDX de la documentation officielle.
- `packages/documentation/public/` — ressources statiques et images de documentation.
- `packages/documentation/README.md` — procédure de développement du site documentaire.

Pour servir le site localement, depuis la racine du dépôt :

```sh
pnpm i
pnpm --filter documentation start
```

Vérifier le nom exact du filtre dans le `package.json` du workspace avant exécution si la version du dépôt a changé.

## 2. Spécifications Open Payments — sous-module existant

Le dépôt déclare déjà le sous-module `open-payments-specifications` dans `.gitmodules`. Il contient les spécifications OpenAPI (formats YAML/JSON) qui définissent les contrats d’API.

Après un clonage local du dépôt, initialiser les sous-modules :

```sh
git submodule update --init --recursive
```

Sources : https://github.com/interledger/open-payments-specifications et https://openpayments.dev/

## 3. Spécifications Interledger Protocol (ILP)

La source normative est publiée dans le dépôt RFC Interledger et sur le site des RFC :

- ILP v4 (RFC 27) : https://interledger.github.io/rfcs/0027-interledger-protocol-4/
- Interledger Architecture (RFC 1) : https://interledger.github.io/rfcs/0001-interledger-architecture/
- Répertoire des spécifications : https://interledger.github.io/rfcs/

Le présent dépôt ne contient pas une copie vendue/recopiée intégralement de l’ensemble du site des RFC. Pour une archive hors ligne complète, il faut cloner ou mirrorer le dépôt source des RFC en respectant sa licence et conserver le commit utilisé.

## 4. GNAP et Payment Pointers

- GNAP — spécification IETF : https://datatracker.ietf.org/doc/draft-ietf-gnap-core-protocol/
- Payment Pointers : https://paymentpointers.org/
- Open Payments : https://github.com/interledger/open-payments

La documentation publique peut être mise à jour indépendamment du code de Rafiki. Lors de chaque intégration, enregistrer les versions ou commits de ces sources afin que les implémentations et tests restent reproductibles.

## 5. Reproductibilité et attribution

Pour chaque copie/mirror documentaire future, conserver :
- l’URL source et le commit/tag exact ;
- la date de synchronisation ;
- la licence et les notices de copyright originales ;
- les modifications locales clairement séparées de la source upstream.

Ne pas écraser les sources originales par des notes locales. Garder les guides propres à notre solution dans `docs/integration/` et les sources upstream identifiables séparément.

## 6. Avertissement d’usage

Rafiki indique qu’il est destiné aux prestataires de services financiers et ne doit pas être utilisé en production par des entités non réglementées. L’architecture de la solution doit être examinée côté sécurité, conformité et intégrité comptable avant tout raccordement à des fonds réels.
