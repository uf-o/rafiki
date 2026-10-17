# Rafiki — documentation d’intégration

Ce dossier rassemble des guides de travail synthétiques et des liens vers les spécifications officielles. Les documents externes ne sont pas recopiés intégralement : consultez toujours les sources normatives pour les schémas, règles et versions à jour.

## Guides locaux

- [Interledger Protocol (ILP)](./ILP-PROTOCOL.md) — paquets Prepare/Fulfill/Reject, routage multi-sauts, connecteur et tests.
- [Open Payments, Payment Pointer et GNAP](./OPEN-PAYMENTS.md) — découverte, quotes, autorisation, paiements et idempotence.
- [Architecture Rafiki et TigerBeetle](./RAFIKI-ARCHITECTURE.md) — services, bases de données, configuration, réseau et déploiement.

## Sources primaires

- [Documentation Rafiki](https://rafiki.dev/)
- [Références API Rafiki](https://docs.rafiki.com/reference/welcome)
- [Dépôt Rafiki](https://github.com/interledger/rafiki)
- [Spécifications Open Payments](https://github.com/interledger/open-payments-specifications)
- [ILP v4 — RFC 27](https://interledger.github.io/rfcs/0027-interledger-protocol-4/)
- [Architecture Interledger — RFC 1](https://interledger.github.io/rfcs/0001-interledger-architecture/)
- [Payment Pointers](https://paymentpointers.org/)
- [GNAP — spécification IETF](https://datatracker.ietf.org/doc/draft-ietf-gnap-core-protocol/)
- [TigerBeetle](https://docs.tigerbeetle.com/)

## Pour utiliser Rafiki

1. Lire d’abord le guide d’architecture.
2. Préparer l’environnement local selon le README racine et initialiser le sous-module des spécifications Open Payments.
3. Étudier ILP et Open Payments avant de toucher aux flux de paiement.
4. Développer et tester localement avec des comptes et fonds de test.
5. Ne connecter des fonds réels qu’après revue technique, sécurité et conformité réglementaire.

**Production :** Rafiki est destiné aux prestataires de services financiers réglementés. Ces guides ne constituent pas une validation réglementaire ou un audit de sécurité.
