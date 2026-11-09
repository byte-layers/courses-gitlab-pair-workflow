---
title: "07 - Préparer la version 2 avec un jalon, un label et une issue"
description: "Planifier le travail suivant dans GitLab, puis ranger"
---

La première version est livrée. GitLab sert aussi à organiser la suivante. Une **issue** décrit un travail à faire, un **label** la classe, et un **jalon** (*milestone*) regroupe les issues d'une même version.

## 07.1 - 🟦 Dev. A : créer le jalon, le label et l'issue

> 🟦 **Dev. A (manager)**, sur gitlab.com.

Dans le menu du projet, ouvrez **Plan**, puis **Milestones**, et cliquez sur **New milestone**. Saisissez le titre `V2` et validez.

Ouvrez ensuite **Manage**, puis **Labels**, et cliquez sur **New label**. Saisissez le titre `feature`, choisissez une couleur et validez.

Ouvrez enfin **Work items** dans le menu du projet, cliquez sur **New item**, et choisissez le type **Issue**. Créez cette issue, en l'assignant à dev. B :

| Champ | Valeur |
|---|---|
| **Title** | `Ajouter une section Compétences` |
| **Assignee** | dev. B, ou vous-même si vous êtes seul |
| **Labels** | `feature` |
| **Milestone** | `V2` |

Elle porte le numéro `#1`, visible à côté de son titre.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
I=$(curl -fsS "$A/issues?milestone=V2&labels=feature" 2>/dev/null)
if ! echo "$I" | grep -q 'Ajouter une section Comp'; then
  echo "Aucune issue « Ajouter une section Compétences » avec le jalon V2 et le label feature."
  exit 1
fi
echo "L'issue est rattachée au jalon V2, avec le label feature."
```

➡️ **Au tour de 🟧 dev. B** : section 07.2.

## 07.2 - 🟧 Dev. B : créer la branche de l'issue

> 🟧 **Dev. B**, sur son environnement, dans le dossier `dev-b`.
>
> ⏳ **Attendez** que dev. A ait terminé la section 07.1.

Chaque issue peut donner une branche. Mettre le numéro de l'issue dans le nom de la branche permet de retrouver le lien entre les deux. Partez de `develop`, et publiez la branche :

```exec
cd ~/layers-git/binome/dev-b && git switch -c feature/1-competences develop
cd ~/layers-git/binome/dev-b && git push -u origin feature/1-competences
```

La suite vous est maintenant familière : développer, ouvrir une merge request vers `develop`, et la faire fusionner par le manager. Vous n'avez pas à le faire ici.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/repository/branches/feature%2F1-competences" >/dev/null 2>&1; then
  echo "La branche feature/1-competences n'est pas sur le serveur."
  exit 1
fi
echo "La branche de l'issue est publiée."
```

➡️ **À vous deux** : section 07.3.

## 07.3 - 🟪 A et B : ranger le dossier du lab

> 🟪 **A et B**, chacun de son côté. Seul, une seule fois suffit. Une fois que dev. B a terminé la section précédente, dev. A peut aussi supprimer le projet sur GitLab, depuis **Settings**, **General**, **Advanced**, **Delete project**.

```exec
cd ~ && rm -rf ~/layers-git/binome
```

```verify
if [ -e ~/layers-git/binome ]; then
  echo "Le dossier ~/layers-git/binome existe encore."
  exit 1
fi
echo "Le dossier est supprimé."
```

## Récap

- `main` contient la production, `develop` prépare la suite, et chaque travail a sa branche.
- Une branche protégée n'accepte le push que des rôles autorisés, et le serveur refuse les autres.
- Un Developer pousse ses branches et propose une merge request, qu'un Maintainer ou un Owner fusionne.
- Une release passe de `develop` à `main` avec un tag, puis revient dans `develop`.
- Un hotfix part de `main`, et rejoint `main` puis `develop`.

Pour reprendre ces gestes sans guide, faites le challenge [Mener un projet GitLab, sans guide](https://byte-layers.com/fr/challenges/gitlab-team-workflow-challenge).

> 📖 [Milestones](https://docs.gitlab.com/user/project/milestones/) · [Labels](https://docs.gitlab.com/user/project/labels/) · [L'article d'origine de Gitflow](https://nvie.com/posts/a-successful-git-branching-model/)
