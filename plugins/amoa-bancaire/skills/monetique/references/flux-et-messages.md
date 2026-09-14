# Flux et messages

## Messages ISO 8583 — MTI

| MTI | Nom | Sens | Usage |
|---|---|---|---|
| `0100` / `0110` | Demande / réponse d'autorisation | Acquéreur → Émetteur | Temps réel, sans mouvement de fonds |
| `0120` / `0130` | Avis d'autorisation | Acquéreur → Émetteur | Autorisation réalisée hors ligne ou en secours, notifiée après coup |
| `0200` / `0210` | Demande / réponse financière | Acquéreur → Émetteur | Autorisation **et** capture en un seul message — typique GAB |
| `0220` / `0230` | Avis financier | Acquéreur → Émetteur | Transaction financière notifiée après coup |
| `0400` / `0410` | Annulation (reversal) | Acquéreur → Émetteur | Libère la provision réservée |
| `0420` / `0430` | Avis d'annulation | Acquéreur → Émetteur | Annulation notifiée, réponse non attendue en temps réel |
| `0800` / `0810` | Gestion de réseau | Bidirectionnel | Echo test, sign-on/sign-off, changement de clé |

Le message `0420` est le principal générateur de suspens : émis sans attente de réponse, il peut se perdre sans que personne ne s'en aperçoive avant la réconciliation.

## Data elements les plus manipulés

| DE | Contenu | Pourquoi il compte |
|---|---|---|
| 2 | PAN | Jamais en clair dans un livrable |
| 3 | Processing code | Distingue achat, retrait, remboursement, consultation de solde |
| 4 | Montant de la transaction | Dans la devise de la transaction (DE49) |
| 7 | Date/heure de transmission | Heure GMT — source classique d'écart de date métier |
| 11 | STAN | Numéro de séquence, clé de rapprochement avec l'annulation |
| 12 / 13 | Heure et date locales | Ce que voit le porteur sur son relevé |
| 18 | MCC | Détermine l'interchange et les restrictions par type de commerce |
| 22 | Mode de saisie POS | Piste, puce, sans contact, manuel, token — détermine le transfert de responsabilité |
| 25 | POS condition code | Présence du porteur, MOTO, récurrent |
| 32 | Identifiant institution acquéreur | Routage |
| 37 | RRN | Référence de bout en bout, clé de réconciliation avec la compensation |
| 38 | Code d'autorisation | Preuve de l'accord émetteur |
| 39 | Code réponse | Voir tableau ci-dessous |
| 41 / 42 | Terminal ID / Merchant ID | Rattachement contractuel de l'accepteur |
| 49 | Devise de la transaction | À ne pas confondre avec la devise de facturation |
| 52 | PIN block | Chiffré, traité en HSM |
| 55 | Données EMV (ICC) | Cryptogramme, résultats de contrôle carte |
| 90 | Données du message d'origine | Indispensable pour rattacher une annulation |

**Clé de rapprochement recommandée** dans une spec de réconciliation : RRN (DE37) en clé primaire, complété par STAN (DE11) + date (DE13) + terminal (DE41) en clé de secours. Le RRN seul n'est pas garanti unique dans le temps chez tous les acquéreurs.

## Codes réponse (DE39) les plus fréquents

| Code | Signification | Nature |
|---|---|---|
| `00` | Transaction acceptée | — |
| `01` | Consulter l'émetteur | Décision humaine |
| `03` | Accepteur invalide | Paramétrage |
| `04` / `07` | Capturer la carte | Sécurité |
| `05` | Ne pas honorer | Refus émetteur générique — souvent scoring fraude |
| `12` | Transaction invalide | Format ou opération non supportée |
| `13` | Montant invalide | Format |
| `14` | Numéro de carte invalide | Donnée |
| `41` / `43` | Carte perdue / volée | Sécurité |
| `51` | Provision insuffisante | Solde |
| `54` | Carte expirée | Donnée |
| `55` | Code PIN erroné | Sécurité |
| `57` / `58` | Opération non autorisée au porteur / au terminal | Paramétrage |
| `61` / `65` | Plafond de montant / de fréquence dépassé | Limites |
| `75` | Nombre d'essais PIN dépassé | Sécurité |
| `91` | Émetteur ou switch indisponible | Technique |
| `96` | Dysfonctionnement système | Technique |

Distinction structurante pour un plan de recette : un refus `51` ou `61` est un **refus métier** attendu et testable ; un refus `91` ou `96` est un **incident technique** qui doit déclencher un mécanisme de secours (stand-in) et une alerte production. Ne les traite jamais dans le même cas de test.

## EMV

Le terminal et la carte négocient une décision, puis l'émetteur tranche :

| Cryptogramme | Sens |
|---|---|
| **ARQC** | La carte demande une autorisation en ligne |
| **TC** | Transaction acceptée hors ligne |
| **AAC** | Transaction refusée |

Le **CVM** (méthode de vérification du porteur) est la question à toujours poser dans une spec : PIN en ligne, PIN hors ligne, signature, CDCVM (biométrie du téléphone), ou aucune vérification sous un plafond sans contact. Le CVM appliqué détermine qui supporte la perte en cas de fraude.

Le **TVR** et le résultat des contrôles carte sont dans DE55. En analyse d'incident, c'est là qu'on trouve pourquoi une transaction est partie en ligne alors qu'elle aurait dû passer hors ligne.

## 3-D Secure et authentification forte

**EMV 3-D Secure (3DS2)** est un protocole d'authentification pour les transactions à distance. Il transporte des données contextuelles permettant à l'émetteur de décider entre authentification transparente (frictionless) et authentification explicite (challenge).

**L'authentification forte (SCA)** est une obligation réglementaire européenne issue de la DSP2. Ce n'est pas la même chose : 3DS est un moyen de la satisfaire, pas l'obligation elle-même. Des exemptions existent (faible montant, analyse de risque, opérations récurrentes, bénéficiaires de confiance), chacune avec ses conditions et son impact sur le transfert de responsabilité.

**Le point à spécifier systématiquement** : qui porte la perte en cas de fraude selon que l'authentification a eu lieu, a été tentée, ou a été exemptée. C'est la règle de gestion la plus souvent oubliée dans les specs e-commerce.

## Sécurité et données sensibles

| Donnée | Règle |
|---|---|
| PAN | Masqué en restitution, chiffré ou tokenisé au stockage |
| Données de piste | Interdiction de stockage après autorisation |
| CVV / CVC | Interdiction absolue de stockage |
| PIN | Jamais en clair hors HSM, jamais stocké |

Le PIN block est traduit d'une clé à l'autre par un HSM à chaque changement de zone de sécurité. Toute spec impliquant le PIN doit identifier les zones et les points de traduction.

La **tokenisation** remplace le PAN par un jeton limité à un domaine d'usage. Dans une spec, précise toujours s'il s'agit d'un token réseau (émis par le scheme, utilisé en paiement) ou d'un token de stockage propriétaire (usage interne, non transportable) — les deux n'ont ni le même cycle de vie ni les mêmes contraintes.
