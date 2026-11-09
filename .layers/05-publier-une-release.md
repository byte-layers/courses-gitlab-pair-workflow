---
title: "05 - Publier la version 1.0.0 avec une branche de release"
description: "Une branche release, une fusion dans main, un tag"
---

`develop` contient tout ce qui doit partir en production. Une **branche de release** isole la préparation de la version : pendant qu'on la finalise, l'équipe peut continuer sur `develop`. Une fois prête, elle est fusionnée dans `main`, et aussi dans `develop` pour que rien ne se perde.

Une seule personne s'en charge. Comme `main` et `develop` sont protégées, c'est le manager. Ce lab ne déploie rien : la production, c'est simplement le contenu de `main`.

## 05.1 - 🟦 Dev. A : préparer la release

> 🟦 **Dev. A (manager)**, dans `dev-a`.

Récupérez d'abord le travail de dev. B, puis créez la branche de release à partir de `develop` :

```exec
cd ~/layers-git/binome/dev-a && git switch develop
cd ~/layers-git/binome/dev-a && git pull
cd ~/layers-git/binome/dev-a && git switch -c release/1.0.0
```

Sur cette branche, on ne fait que finaliser la version : ici, enregistrer son numéro dans un fichier `VERSION`.

```exec
cd ~/layers-git/binome/dev-a && echo '1.0.0' > VERSION
cd ~/layers-git/binome/dev-a && git add VERSION && git commit -m "Prépare la version 1.0.0"
```

Confirmez lorsque ce commit est fait.

```acknowledge
Le commit de préparation est fait sur release/1.0.0
```

## 05.2 - 🟦 Dev. A : fusionner dans main et poser le tag v1.0.0

> 🟦 **Dev. A (manager)**, dans `dev-a`.

Fusionnez la release dans `main`. Sans option, Git ferait une fusion **fast-forward** : il avancerait simplement la branche, sans trace de la fusion. L'option `--no-ff` force la création d'un commit de fusion, pour que l'historique montre l'arrivée de la version. Un **tag** marque ensuite ce commit avec le numéro de version. L'option `-a` en fait un tag annoté, qui garde son auteur, sa date et un message :

```exec
cd ~/layers-git/binome/dev-a && git switch main
cd ~/layers-git/binome/dev-a && git pull
cd ~/layers-git/binome/dev-a && git merge --no-ff release/1.0.0 -m "Publie la version 1.0.0"
cd ~/layers-git/binome/dev-a && git tag -a v1.0.0 -m "Version 1.0.0"
```

> 💡 **Remarque.** Le manager fusionne ici dans `main` depuis son poste, pour garder le lab court. Dans une vraie équipe, il vaut mieux passer aussi par une merge request vers `main`, comme le fera le hotfix.

Rien n'est encore publié sur le serveur. Confirmez pour passer à la suite.

```acknowledge
main contient la release, et le tag v1.0.0 est posé
```

## 05.3 - 🟦 Dev. A : reporter la release dans develop et tout publier

> 🟦 **Dev. A (manager)**, dans `dev-a`.

Reportez la release dans `develop`, qui ne connaît pas encore le fichier `VERSION`. Sans argument, `git push` n'envoie que la branche courante, et jamais les tags : cette commande nomme donc tout ce qu'il faut publier. Supprimez enfin les branches locales devenues inutiles :

```exec
cd ~/layers-git/binome/dev-a && git switch develop
cd ~/layers-git/binome/dev-a && git merge --no-ff release/1.0.0 -m "Reporte la version 1.0.0 dans develop"
cd ~/layers-git/binome/dev-a && git push origin main develop v1.0.0
cd ~/layers-git/binome/dev-a && git branch -d release/1.0.0 feature/css
```

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/repository/tags" 2>/dev/null | grep -q '"name":"v1.0.0"'; then
  echo "Le tag v1.0.0 n'est pas sur le serveur."
  exit 1
fi
for B in main develop; do
  if ! curl -fsS "$A/repository/files/VERSION/raw?ref=$B" 2>/dev/null | grep -q '1.0.0'; then
    echo "La branche $B du serveur n'a pas le fichier VERSION à 1.0.0."
    exit 1
  fi
done
echo "La version 1.0.0 est publiée sur main, et reportée dans develop."
```

➡️ **Au tour de 🟧 dev. B** : section 05.4.

## 05.4 - 🟧 Dev. B : récupérer la version et ranger sa branche

> 🟧 **Dev. B**, dans `dev-b`.
>
> ⏳ **Attendez** que dev. A ait terminé la section 05.3.

Récupérez les deux branches permanentes et le tag, puis supprimez votre copie locale de la branche de travail, désormais fusionnée :

```exec
cd ~/layers-git/binome/dev-b && git switch main
cd ~/layers-git/binome/dev-b && git pull
cd ~/layers-git/binome/dev-b && git switch develop
cd ~/layers-git/binome/dev-b && git pull
cd ~/layers-git/binome/dev-b && git branch -d feature/content
cd ~/layers-git/binome/dev-b && git tag
```

La dernière commande affiche le tag de la version :

```text
v1.0.0
```

Confirmez lorsque vous voyez ce tag.

```acknowledge
Je vois le tag v1.0.0
```

➡️ **Dev. B, continuez** à l'étape 06 : section 06.1.

> 📖 [Tags](https://docs.gitlab.com/user/project/repository/tags/)
