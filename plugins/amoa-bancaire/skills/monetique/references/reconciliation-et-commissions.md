# Réconciliation et commissions

## Les trois niveaux de réconciliation

La réconciliation monétique n'est jamais un rapprochement unique. C'est une cascade de trois, chacune avec sa propre population de suspens.

| Niveau | On rapproche | Clé de rapprochement | Écart typique |
|---|---|---|---|
| **N1 — Autorisation / Capture** | Journal des autorisations vs remises des accepteurs | RRN + STAN + date + terminal | Autorisation sans capture, capture sans autorisation |
| **N2 — Capture / Compensation** | Remises acceptées vs fichiers de compensation du scheme | RRN + montant + devise | Transaction non présentée, présentée en double, montant modifié |
| **N3 — Compensation / Trésorerie** | Positions de compensation vs mouvements du compte de règlement | Date de règlement + devise + montant net | Décalage de valeur, frais scheme non anticipés, écart de change |

**Erreur fréquente en projet** : spécifier une réconciliation « transactions vs comptabilité » en un seul rapprochement. Les écarts deviennent alors ininterprétables, parce qu'un suspens N1 et un suspens N3 n'ont ni la même cause, ni le même responsable, ni le même délai de traitement.

## Typologie des suspens

| Suspens | Cause probable | Traitement |
|---|---|---|
| Autorisation sans capture | Vente abandonnée, ou télécollecte non remontée | Expiration automatique après le délai contractuel ; alerte si volume anormal |
| Capture sans autorisation | Transaction forcée, secours hors ligne, fraude accepteur | Contrôle manuel ; risque porté par l'acquéreur |
| Montant capturé ≠ montant autorisé | Pourboire, ajustement hôtellerie/location, carburant | Règle de tolérance à spécifier, en valeur absolue **et** en pourcentage |
| Transaction compensée en double | Rejeu de fichier, incident de remise | Détection par RRN ; annulation de la seconde présentation |
| Écart de date | Heure GMT du message vs date métier locale | Définir la date de référence dès la spec — c'est une décision, pas un détail technique |
| Écart de change | Devise de transaction ≠ devise de règlement | Identifier le taux appliqué et le moment de sa fixation |

## Règle de conception : la date de référence

Une transaction porte au moins quatre dates : transmission (DE7, GMT), locale porteur (DE12/13), de remise par l'accepteur, de compensation. Une spécification de réconciliation doit **désigner explicitement laquelle fait foi**, et pour quel usage : arrêté comptable, relevé porteur, calcul de commission, décompte des délais de contestation.

Ne pas trancher cette question produit des écarts de rapprochement en fin de mois qui ne se résolvent jamais proprement.

## Chaîne de commissions

Sur une transaction acquéreur, la valeur se décompose en cascade :

```
Montant payé par le porteur
  − Commission commerçant (MSC), prélevée par l'acquéreur
      dont Interchange, reversé à l'émetteur
      dont Frais scheme, facturés par le réseau
      dont Marge acquéreur
  = Montant net versé à l'accepteur
```

**Les trois composantes ne se calculent pas au même moment** : l'interchange est déterminé à la compensation selon le MCC, le canal, le type de carte et la géographie ; les frais scheme arrivent souvent en facturation périodique séparée ; la marge est contractuelle. Une spec de calcul de commission doit préciser le fait générateur et le moment de calcul de chaque composante.

## Facteurs déterminant l'interchange

L'interchange n'est jamais un taux unique. Il dépend au minimum de :

| Facteur | Effet |
|---|---|
| Type de carte | Débit, crédit, prépayée, commerciale — les cartes commerciales sont les plus chères |
| Géographie | Domestique, intra-régional, international — trois grilles différentes |
| MCC de l'accepteur | Certains secteurs bénéficient de taux réduits |
| Canal et mode de saisie | Carte présente, e-commerce authentifié, e-commerce non authentifié |
| Programme scheme | Adhésion à des programmes de qualité de données |

**Ne cite jamais un taux d'interchange de mémoire dans un livrable.** Les grilles sont publiées périodiquement par chaque scheme et encadrées réglementairement dans certaines zones. Renvoie à la grille en vigueur et date-la.

## Ce qu'un plan de recette réconciliation doit contenir

1. **Un jeu de données par type de suspens** — pas un jeu global. Un fichier avec une transaction non présentée, un avec un doublon, un avec un écart de montant dans la tolérance, un avec un écart hors tolérance.
2. **Un test de rejeu** — le même fichier de compensation injecté deux fois ne doit pas doubler les écritures.
3. **Un test de fichier partiel** — coupure en cours de traitement, reprise sans perte ni doublon.
4. **Un test d'arrêté** — que se passe-t-il pour une transaction à cheval sur deux jours comptables, ou sur deux mois.
5. **Un test de devise** — transaction en devise étrangère, avec vérification du taux appliqué et de sa date de fixation.
6. **Un contrôle de bouclage** — somme des transactions individuelles = montant net du règlement, aux commissions près. Si ce contrôle n'est pas dans le plan de recette, la réconciliation n'est pas testée.

## KPI d'un projet de réconciliation

| KPI | Définition | Cible usuelle |
|---|---|---|
| Taux de rapprochement automatique | Transactions rapprochées sans intervention / total | À définir avec le métier, mesuré avant le projet |
| Volume de suspens à J+1 | Nombre et montant restant non rapprochés le lendemain | — |
| Âge moyen des suspens | Ancienneté des écarts ouverts | Un suspens qui vieillit devient une perte |
| Délai de bouclage de l'arrêté | Du dernier flux reçu à la validation comptable | — |
| Taux de reprise manuelle | Écritures corrigées à la main / total | Indicateur réel de la qualité du paramétrage |

Chacun de ces KPI a besoin d'une **baseline mesurée avant le projet**. Sans elle, aucun gain ne sera démontrable en fin de mission.
