# Interledger Protocol (ILP) — guide d’intégration

> Guide de travail pour le dépôt Rafiki. Ce document synthétise les concepts et renvoie aux spécifications normatives officielles ; il ne remplace pas les RFC.

## Sources normatives

- [ILP v4 — RFC 27](https://interledger.github.io/rfcs/0027-interledger-protocol-4/)
- [Interledger Architecture — RFC 1](https://interledger.github.io/rfcs/0001-interledger-architecture/)
- [Relationship Between Protocols — IL-RFC-33](https://interledger.github.io/rfcs/0033-relationship-between-protocols/)
- [Vue d’ensemble de la pile Interledger](https://interledger.org/tech/overview/)
- [Documentation du connecteur Rafiki](https://rafiki.dev/integration/deployment/services/backend-service/)

## Modèle mental

ILP transporte de la valeur entre des comptes et des systèmes de paiement qui ne partagent pas nécessairement le même registre ou la même devise. Les nœuds peuvent être expéditeurs, destinataires ou connecteurs. Un connecteur relaie les paquets vers le prochain pair, applique les règles de routage et de liquidité et, selon la configuration, les frais et la conversion de devise.

ILP n’est ni une blockchain ni un registre central. Le règlement des obligations entre participants se fait séparément sur leurs systèmes de règlement convenus.

## Cycle de vie d’un paquet

1. **Prepare** — demande conditionnelle de transfert, avec montant, destination, expiration et condition cryptographique. Les connecteurs transmettent le paquet et peuvent ajuster le montant pour les frais ou le change, selon leurs règles.
2. **Fulfill** — le destinataire satisfait la condition avant expiration. La confirmation remonte le chemin inverse ; les participants finalisent les changements de solde préparés.
3. **Reject** — un nœud refuse le paquet (par exemple route indisponible, montant trop élevé, liquidité insuffisante ou expiration). Le refus remonte vers l’expéditeur avec des informations d’erreur. Une préparation non satisfaite ne doit pas être traitée comme un paiement réussi.

Un Prepare expiré ou rejeté ne doit pas être comptabilisé comme un transfert réussi. Les systèmes applicatifs doivent distinguer une tentative, un paiement en cours, une réussite confirmée et un échec.

## Routage multi-sauts

- Un expéditeur envoie le paquet au premier connecteur configuré.
- Chaque connecteur valide le pair et les limites, sélectionne le prochain saut et relaie le paquet.
- Les connecteurs peuvent appliquer les règles de frais, de change, de liquidité, de limitation de débit et de montant maximal convenu.
- Le Fulfill ou le Reject revient le long du chemin de retour.
- Les informations de routage et de pair doivent être configurées et surveillées ; ne jamais accepter des paquets de pairs inconnus.

## Comment cela se matérialise dans Rafiki

Le service `backend` héberge le connecteur ILP et les API Open Payments. Il authentifie les paquets à l’aide des identifiants du pair, relaie les paquets vers des pairs configurés et peut terminer certains paiements sur son serveur STREAM. Les paiements entrants associés à une adresse ou à un incoming payment géré localement peuvent alors être crédités et suivis.

Pour les appels HTTP entre pairs, configurez les jetons entrants et sortants échangés lors de l’établissement de la relation de peering. Protégez-les comme des secrets.

## Liste de vérification d’intégration

- Définir les pairs, adresses ILP et actifs pris en charge.
- Définir les limites de montant, délais d’expiration, règles de liquidité et éventuels frais.
- Garder les jetons de peering et secrets hors du code et des journaux.
- Traiter uniquement un Fulfill validé comme confirmation de succès.
- Rendre les opérations applicatives idempotentes et gérer les notifications répétées.
- Tester succès, Reject, expiration, absence de route, manque de liquidité, conversion et redémarrage.
- Surveiller les paquets, erreurs, soldes de liquidité et écarts de rapprochement sans journaliser de secrets.

## Note de sécurité

Les détails exacts des champs et règles de validation sont définis par la RFC ILP v4. Toujours consulter la RFC avant d’implémenter ou de modifier un encodeur, décodeur ou connecteur.
