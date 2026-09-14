---
description: Enregistrer le contexte de la mission en cours, une fois pour toutes
---

L'utilisateur démarre une mission. Recueille et structure son contexte, puis produis un fichier réutilisable.

Informations fournies : $ARGUMENTS

## Étape 1 — compléter

Compare ce que l'utilisateur a donné à la liste ci-dessous. Demande uniquement ce qui manque, groupé en une seule question, maximum 8 lignes.

| Élément | Pourquoi il compte |
|---|---|
| Banque / client | Vocabulaire, contraintes internes |
| Core banking | Faisabilité en paramétrage vs développement |
| Plateforme monétique (si applicable) | Format des messages et des fichiers |
| Processus concerné | Périmètre |
| Volumétrie | Dimensionnement, exigences non fonctionnelles |
| Méthodologie (Agile / Cycle en V) | Format des exigences attendu |
| Échéance | Arbitrage exhaustivité / exploitabilité |
| Contraintes réglementaires | Priorité 1 des arbitrages |
| Décisions déjà actées | Ce qu'il ne faut pas rouvrir |
| Interlocuteurs clés et leur rôle | RACI, destinataires des questions |
| Livrables déjà produits | Cohérence, numérotation à poursuivre |
| Risques perçus | Ce qui doit remonter dans chaque livrable |

## Étape 2 — produire

Génère un fichier `contexte-mission.md` contenant le tableau rempli, plus une section « Conventions du projet » (numérotation des exigences, format de livraison attendu, langue).

Dis à l'utilisateur de le déposer dans la base de connaissances de son Projet Claude, pour que chaque nouvelle conversation démarre avec ce contexte sans qu'il ait à le redonner.
