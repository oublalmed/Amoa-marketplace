# Glossaire monétique

## Acteurs et rôles

| Terme | Définition |
|---|---|
| **Porteur** (cardholder) | Titulaire de la carte |
| **Accepteur** (merchant) | Commerçant acceptant la carte en paiement |
| **Émetteur** (issuer) | Banque qui émet la carte et porte le risque porteur |
| **Acquéreur** (acquirer) | Banque qui contractualise avec l'accepteur et porte le risque commerçant |
| **Scheme** | Réseau carte définissant les règles : Visa, Mastercard, CB, réseaux domestiques |
| **Switch** | Plateforme de routage des messages entre acquéreurs et émetteurs |
| **Processeur** | Prestataire opérant tout ou partie de la chaîne pour le compte d'une banque |
| **PSP** | Prestataire de services de paiement, intermédiaire entre l'accepteur et l'acquéreur |
| **Facilitateur de paiement** | Agrège plusieurs accepteurs sous un contrat acquéreur unique |

## Flux et opérations

| Terme | Définition |
|---|---|
| **Autorisation** | Contrôle temps réel et réservation de provision, sans mouvement de fonds |
| **Pré-autorisation** | Autorisation d'un montant estimé, ajusté à la capture (hôtellerie, location, carburant) |
| **Capture / remise** | Confirmation de la vente par l'accepteur, généralement en fin de journée |
| **Télécollecte** | Remontée des transactions du terminal vers l'acquéreur |
| **Compensation** (clearing) | Échange des créances entre banques via le scheme |
| **Règlement** (settlement) | Mouvement effectif des fonds |
| **Annulation** (reversal) | Suppression d'une autorisation avant compensation |
| **Remboursement** (refund) | Transaction de sens inverse après compensation |
| **Chargeback** | Rejet de créance par l'émetteur à la demande du porteur |
| **Représentation** | Contestation du chargeback par l'acquéreur, preuves à l'appui |
| **Stand-in** | Autorisation prise par le scheme ou le switch quand l'émetteur est indisponible |
| **Transaction forcée** | Saisie manuelle d'un code d'autorisation obtenu hors système |
| **MIT** | Transaction initiée par le commerçant sans présence du porteur (abonnement, ajustement) |
| **MOTO** | Vente à distance par courrier ou téléphone |

## Technique

| Sigle | Signification |
|---|---|
| **PAN** | Numéro de carte |
| **BIN / IIN** | Préfixe du PAN identifiant l'émetteur et le type de carte |
| **STAN** | Numéro de séquence du message |
| **RRN** | Référence de la transaction, clé de rapprochement de bout en bout |
| **MCC** | Code d'activité de l'accepteur |
| **MTI** | Type de message ISO 8583 |
| **DE** | Data element, champ d'un message ISO 8583 |
| **EMV** | Standard de la carte à puce |
| **ARQC / TC / AAC** | Cryptogrammes EMV : demande en ligne / accord hors ligne / refus |
| **TVR** | Résultat des contrôles effectués par le terminal |
| **CVM** | Méthode de vérification du porteur |
| **CDCVM** | Vérification du porteur par l'appareil (biométrie du téléphone) |
| **HSM** | Module matériel de sécurité, seul lieu de manipulation des clés et du PIN |
| **PIN block** | Code confidentiel chiffré |
| **Token** | Substitut du PAN limité à un domaine d'usage |
| **3DS** | Protocole d'authentification pour les paiements à distance |
| **SCA** | Authentification forte, obligation réglementaire européenne |
| **P2PE** | Chiffrement de bout en bout entre le terminal et le point de déchiffrement |
| **PCI DSS** | Norme de sécurité des données de l'industrie des cartes |

## Économique

| Terme | Définition |
|---|---|
| **Interchange** | Part reversée par l'acquéreur à l'émetteur sur chaque transaction |
| **Frais scheme** | Facturation du réseau aux deux banques |
| **MSC** | Commission prélevée par l'acquéreur sur l'accepteur |
| **Tarification Interchange++** | Facturation décomposée : interchange + frais scheme + marge |
| **Tarification blended** | Taux unique tous types de cartes confondus |
| **Réserve** | Montant retenu par l'acquéreur en garantie du risque de chargeback |

## Progiciels courants

| Progiciel | Éditeur |
|---|---|
| PowerCARD | HPS |
| Way4 | OpenWay |
| Base24 | ACI |
| Postilion | BPC |
| Cardlink | Diverses implémentations régionales |

## Pièges de vocabulaire à corriger en atelier

| Confusion fréquente | Correction |
|---|---|
| « Annuler » pour un remboursement | Deux opérations distinctes, deux moments, deux écritures |
| « Transaction refusée » sans code | Toujours donner le code DE39 : la cause change le traitement |
| « Commission bancaire » | Préciser : interchange, frais scheme, ou MSC |
| « La carte a été rejetée » | Préciser qui rejette : le terminal, le switch, ou l'émetteur |
| « Paiement sécurisé » | Préciser : chiffré, authentifié, ou avec transfert de responsabilité |
