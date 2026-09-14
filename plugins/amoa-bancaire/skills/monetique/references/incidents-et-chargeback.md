# Incidents, contestations et fraude

## Ne pas confondre quatre opérations distinctes

| Opération | Quand | Effet | Qui déclenche |
|---|---|---|---|
| **Annulation** (reversal) | Avant compensation | Libère la provision, la transaction n'existera jamais | Acquéreur / accepteur |
| **Remboursement** (refund) | Après compensation | Nouvelle transaction de sens inverse | Accepteur |
| **Chargeback** | Après compensation | Rejet de la créance vers l'acquéreur | Émetteur, à la demande du porteur |
| **Extourne comptable** | À tout moment | Correction d'écriture, sans flux scheme | Back office |

Une spec qui utilise « annulation » pour désigner un remboursement produit des règles de gestion fausses et des tests inexploitables. Impose le vocabulaire dès le glossaire du projet.

## Cycle de contestation

```
Présentation (compensation initiale)
      ↓
Chargeback — l'émetteur rejette la créance
      ↓
Représentation — l'acquéreur conteste le rejet avec preuves
      ↓
Pré-arbitrage — tentative de résolution bilatérale
      ↓
Arbitrage — le scheme tranche et facture la partie perdante
```

Chaque étape a un délai impératif propre au scheme et au motif. Un délai dépassé vaut renoncement, quelle que soit la valeur du dossier. **Les délais et les codes motifs changent à chaque publication des règles du scheme : ne les cite jamais de mémoire dans un livrable, va les chercher dans la documentation en vigueur.**

## Familles de motifs de contestation

Les schemes classent les motifs en grandes familles, stables même quand les codes changent :

| Famille | Contenu type | Défense de l'acquéreur |
|---|---|---|
| **Fraude** | Transaction non reconnue par le porteur | Preuve d'authentification (3DS), de présence carte, historique |
| **Autorisation** | Transaction sans autorisation valide, ou autorisation expirée | Trace de l'autorisation, code d'autorisation, RRN |
| **Erreur de traitement** | Double débit, montant erroné, devise erronée, transaction tardive | Ticket, journal du terminal, preuve de remise unique |
| **Litige commercial** | Marchandise non reçue, non conforme, service annulé, abonnement non résilié | Preuve de livraison, CGV acceptées, trace de résiliation |

## Ce qu'une spec ou un plan de recette doit couvrir

1. **Constitution du dossier** — quelles pièces, sous quel format, conservées combien de temps, récupérables par qui.
2. **Délais internes** — ils doivent être strictement inférieurs aux délais scheme, marge de sécurité incluse. Une alerte à J-5 du délai scheme n'est pas un confort, c'est un contrôle.
3. **Écritures comptables à chaque étape** — le montant contesté quitte-t-il le compte de l'accepteur immédiatement, ou reste-t-il provisionné ? Qui porte la trésorerie pendant l'instruction ?
4. **Récupération auprès de l'accepteur** — sur quel compte, avec quel préavis, et que se passe-t-il si le compte est insuffisant ou le commerçant défaillant. C'est le risque acquéreur pur, systématiquement sous-spécifié.
5. **Cas du commerçant en faillite** — l'acquéreur porte la perte. La spec doit prévoir la détection en amont (réserve, garantie, surveillance du taux de contestation).

## Indicateurs de surveillance fraude

| Indicateur | Définition | Pourquoi |
|---|---|---|
| Taux de fraude | Montant fraudé / montant total, par canal | Les schemes imposent des seuils au-delà desquels des programmes de remédiation se déclenchent |
| Taux de chargeback | Nombre de contestations / nombre de transactions, par accepteur | Seuil de mise sous surveillance du commerçant |
| Taux de refus | Refus / demandes d'autorisation | Un taux qui grimpe signale un paramétrage trop strict ou une attaque en cours |
| Taux d'authentification aboutie | Authentifications réussies / tentées, en e-commerce | Mesure la friction imposée au porteur |

**Piège classique** : un taux de fraude qui baisse alors que le taux de refus explose n'est pas un succès. C'est du chiffre d'affaires perdu. Ces deux indicateurs se lisent toujours ensemble.

## Scénarios de test à ne jamais oublier

| Scénario | Ce qu'il révèle |
|---|---|
| Autorisation acceptée, capture jamais reçue | Gestion des autorisations orphelines et leur expiration |
| Capture reçue sans autorisation préalable | Risque acquéreur, règle d'acceptation ou de rejet |
| Annulation reçue après compensation | Le système doit refuser et rediriger vers un remboursement |
| Double présentation du même RRN | Détection de doublon, sinon double débit porteur |
| Chargeback sur transaction déjà remboursée | Risque de double restitution au porteur |
| Transaction en devise étrangère contestée | Quel taux de change s'applique à la restitution, celui du jour J ou du jour de la contestation |
| Coupure réseau entre `0100` et `0110` | Le porteur est débité sans réponse — mécanisme de reprise et d'annulation automatique |
| Transaction hors ligne au-delà du plafond | Contrôle de risque terminal et responsabilité en cas d'impayé |
