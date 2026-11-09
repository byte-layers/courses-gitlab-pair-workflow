---
title: "06 - Corriger la production avec un hotfix"
description: "Un correctif parti de main, puis reporté dans develop"
---

En relisant le site, dev. B s'aperçoit que le titre de la page en production, « Mon CV », ne nomme personne. Il faut le corriger tout de suite, sans embarquer ce qui se prépare sur `develop`. Un **hotfix** est un correctif urgent : il part de `main`, et rejoint ensuite `main` et `develop`.

## 06.1 - 🟧 Dev. B : développer le correctif depuis main

> 🟧 **Dev. B**, dans `dev-b` puis sur gitlab.com.

Créez la branche du correctif à partir de `main` :

```exec
cd ~/layers-git/binome/dev-b && git switch -c hotfix/1.0.1 main
```

Ouvrez `~/layers-git/binome/dev-b/cv/index.html` dans votre éditeur, et remplacez la ligne `<title>` par celle-ci :

```html
  <title>CV de Jane Doe</title>
```

Mettez aussi à jour le numéro de version, faites un commit, et publiez la branche :

```exec
cd ~/layers-git/binome/dev-b && echo '1.0.1' > VERSION
cd ~/layers-git/binome/dev-b && git commit -am "Corrige le titre de la page"
cd ~/layers-git/binome/dev-b && git push -u origin hotfix/1.0.1
```

Sur GitLab, ouvrez une merge request de `hotfix/1.0.1` vers `main`, comme vous l'avez fait pour la feature content, et désignez dev. A dans **Reviewer**. Cette fois, la cible proposée par défaut, `main`, est la bonne. Seul, désignez-vous, ou laissez ce champ vide.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/merge_requests?state=all&source_branch=hotfix/1.0.1&target_branch=main" 2>/dev/null | grep -q '"iid"'; then
  echo "Aucune merge request de hotfix/1.0.1 vers main."
  exit 1
fi
echo "La merge request du correctif est ouverte."
```

➡️ **Au tour de 🟦 dev. A** : section 06.2.

## 06.2 - 🟦 Dev. A : fusionner le correctif et poser le tag v1.0.1

> 🟦 **Dev. A (manager)**, sur gitlab.com puis dans `dev-a`.
>
> ⏳ **Attendez** que dev. B ait ouvert sa merge request (section 06.1).

Sur GitLab, ouvrez la merge request et cliquez sur **Merge**. Récupérez ensuite `main`, et marquez cette version :

```exec
cd ~/layers-git/binome/dev-a && git switch main
cd ~/layers-git/binome/dev-a && git pull
cd ~/layers-git/binome/dev-a && git tag -a v1.0.1 -m "Version 1.0.1"
```

Si `develop` ne recevait pas ce correctif, le bug reviendrait à la prochaine version. Fusionnez donc `main` dans `develop`, et publiez `develop` et le tag :

```exec
cd ~/layers-git/binome/dev-a && git switch develop
cd ~/layers-git/binome/dev-a && git pull
cd ~/layers-git/binome/dev-a && git merge --no-ff main -m "Reporte le correctif 1.0.1 dans develop"
cd ~/layers-git/binome/dev-a && git push origin develop v1.0.1
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
if ! curl -fsS "$A/repository/tags" 2>/dev/null | grep -q '"name":"v1.0.1"'; then
  echo "Le tag v1.0.1 n'est pas sur le serveur."
  exit 1
fi
for B in main develop; do
  if ! curl -fsS "$A/repository/files/cv%2Findex.html/raw?ref=$B" 2>/dev/null | grep -q 'CV de Jane Doe'; then
    echo "La branche $B du serveur ne contient pas le correctif."
    exit 1
  fi
done
echo "Le correctif est sur main et sur develop, et la version 1.0.1 est taguée."
```

➡️ **Au tour de 🟧 dev. B** : section 06.3. Dev. A, continuez avec l'étape 07.

## 06.3 - 🟧 Dev. B : récupérer le correctif et ranger sa branche

> 🟧 **Dev. B**, dans `dev-b`.
>
> ⏳ **Attendez** que dev. A ait terminé la section 06.2.

Mettez à jour vos deux branches permanentes, et supprimez votre copie locale de la branche du correctif :

```exec
cd ~/layers-git/binome/dev-b && git switch main
cd ~/layers-git/binome/dev-b && git pull
cd ~/layers-git/binome/dev-b && git switch develop
cd ~/layers-git/binome/dev-b && git pull
cd ~/layers-git/binome/dev-b && git branch -d hotfix/1.0.1
```

Confirmez lorsque vos branches sont à jour.

```acknowledge
Mes branches main et develop sont à jour
```

➡️ **Dev. B, patientez** jusqu'à la section 07.2.
