---
title: "Travailler en binôme sur GitLab avec un workflow Git simple"
description: "Lab complet à deux : créer un projet GitLab, protéger ses branches, livrer des fonctionnalités, publier une version et corriger la production."
tags:
  - lab
  - gitlab
difficulty: beginner
duration: 120
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: true
  autoCompleteOnNext: unverifiable
---

Dans ce lab, deux développeurs font vivre le même projet GitLab, un petit CV en HTML, de sa création à sa première version corrigée. Ils suivent un **workflow Git**, c'est-à-dire les règles d'une équipe pour ses branches : `main` contient ce qui est en production, `develop` prépare la suite, et chaque travail se fait sur une branche temporaire.

**Niveau** : vous savez faire un commit, créer et fusionner une branche, et utiliser `git push` et `git pull`. Si ce n'est pas le cas, commencez par [Enregistrer son travail avec Git](https://byte-layers.com/fr/skills/git-first-commits), [Travailler en parallèle avec les branches](https://byte-layers.com/fr/skills/git-branches-merge) et [Partager son travail avec un dépôt distant](https://byte-layers.com/fr/skills/git-remotes).

**Ce qu'il faut** : Git 2.28 ou plus récent avec votre identité réglée, un compte gratuit sur [gitlab.com](https://gitlab.com/) pour chaque personne, un éditeur de texte et un terminal (Git Bash sous Windows). Les vérifications utilisent `curl`, livré avec Git Bash sous Windows.

## Qui fait quoi

Le lab alterne sans cesse entre deux personnes. Chaque section indique dans son titre qui agit, avec ces repères :

| Repère | Qui | Son dépôt local | Son rôle sur GitLab |
|---|---|---|---|
| 🟦 | **Dev. A**, le manager de l'équipe | `dev-a` | il crée le projet, et en est donc **Owner** |
| 🟧 | **Dev. B** | `dev-b` | il est invité avec le rôle **Developer** |
| 🟪 | **A et B**, chacun de son côté | le sien | |

`dev-a` et `dev-b` sont deux **dépôts Git distincts**, deux clones du même projet GitLab. Ce ne sont pas des sous-dossiers du projet, et ils n'échangent des commits qu'en passant par le serveur.

« Manager » désigne ici la responsabilité dans l'équipe. Ce n'est pas un rôle GitLab.

**À deux**, chaque personne a son environnement de travail, par exemple son poste, et son compte GitLab, et choisit un rôle. Vous ne faites que les sections qui portent votre repère, et vous pouvez passer celles de l'autre en cliquant sur leur rond de progression. L'ordre compte : une ligne ⏳ vous dit qui attendre au début d'une section, et une ligne ➡️ dit à qui c'est le tour à la fin.

**Seul**, vous jouez les deux rôles dans le même environnement et avec le même compte, en passant d'un dépôt local à l'autre. Votre compte a tous les droits : la section 04.3 vous dit comment obtenir quand même le refus de push.

**Vos fichiers** vivent dans `~/layers-git/binome`, et le projet doit rester **public** pour que les vérifications puissent le lire. La dernière étape supprime le dossier.

> 💡 **Remarque.** Ce workflow est une version simplifiée de **Gitflow**, un modèle publié en 2010. Il reste adapté à un logiciel livré par versions successives (v1.x, v2.x). L'auteur de Gitflow précise depuis 2020 que, pour une application livrée en continu, un workflow plus simple convient mieux. Le bit [Workflows Git](https://byte-layers.com/fr/bits/git-workflows) compare ces modèles.
