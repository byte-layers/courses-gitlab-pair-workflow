---
title: "04 - Livrer une fonctionnalité chacun : push refusé, merge request et conflit"
description: "Dev. A fusionne directement, dev. B passe par une merge request"
---

Chacun développe maintenant sa **fonctionnalité** (une *feature*) sur sa propre branche, partie de `develop`. Dev. A ajoute le style de la page, et dev. B en modifie le contenu. Comme vous allez toucher tous les deux à la même ligne, un conflit vous attend à la fin. Deux commandes vont échouer, et c'est voulu : le push refusé, puis la fusion en conflit.

## 04.1 - 🟦 Dev. A : développer la feature css (le style)

> 🟦 **Dev. A (manager)**, dans `dev-a`.

Créez votre branche à partir de `develop` :

```exec
cd ~/layers-git/binome/dev-a && git switch -c feature/css
```

Ouvrez `~/layers-git/binome/dev-a/cv/index.html` dans votre éditeur. Ajoutez cette ligne juste après la ligne `<meta charset="UTF-8">`, pour charger la feuille de style de Bootstrap :

```html
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
```

Dans le même fichier, remplacez la ligne du titre `<h1>` par celle-ci, qui agrandit le titre grâce à une classe de Bootstrap :

```html
    <h1 class="display-5">John Doe<br>Ingénieur DevOps</h1>
```

Enregistrez le fichier, puis faites un commit. L'option `-a` y inclut tous les fichiers déjà suivis par Git, sans `git add` :

```exec
cd ~/layers-git/binome/dev-a && git commit -am "Ajoute le style Bootstrap"
```

Confirmez lorsque votre commit est fait.

```acknowledge
Mon commit est fait sur feature/css
```

➡️ **Dev. A, patientez** : reprenez à la section 04.5 quand dev. B aura ouvert sa merge request.

## 04.2 - 🟧 Dev. B : développer la feature content (le contenu)

> 🟧 **Dev. B**, dans `dev-b`.
>
> ⏳ **Attendez** que dev. A ait protégé `develop` (section 03.1).

Pendant ce temps, créez votre propre branche à partir de `develop` :

```exec
cd ~/layers-git/binome/dev-b && git switch -c feature/content
```

Ouvrez `~/layers-git/binome/dev-b/cv/index.html` dans votre éditeur. Remplacez la ligne `<title>` par la première ligne ci-dessous, et la ligne `<h1>` par la seconde :

```html
  <title>Mon CV</title>
    <h1>Jane Doe<br>Développeuse web</h1>
```

Enregistrez le fichier, puis faites un commit :

```exec
cd ~/layers-git/binome/dev-b && git commit -am "Personnalise le contenu du CV"
```

Confirmez lorsque votre commit est fait.

```acknowledge
Mon commit est fait sur feature/content
```

## 04.3 - 🟧 Dev. B : tenter de pousser sur develop, et se faire refuser

> 🟧 **Dev. B**, dans `dev-b`.

Votre travail est prêt. Le réflexe de l'échauffement serait de l'envoyer directement sur `develop` : fusionnez votre branche dans `develop`, puis poussez. Le push va être refusé, c'est le but de cette section.

> 🧍 **Seul.** Votre compte est Owner, et le push passerait. Avant de lancer la commande, réglez **Allowed to push and merge** sur **No one** pour `develop`, dans **Settings**, **Repository**, **Protected branches**.

```exec
cd ~/layers-git/binome/dev-b && git switch develop
cd ~/layers-git/binome/dev-b && git merge feature/content
cd ~/layers-git/binome/dev-b && git push
```

Votre fusion locale a réussi, mais le serveur refuse le push :

```text
remote: GitLab: You are not allowed to push code to protected branches on this project.
To https://gitlab.com/nom-de-dev-a/cv-binome.git
 ! [remote rejected] develop -> develop (pre-receive hook declined)
error: failed to push some refs to 'https://gitlab.com/nom-de-dev-a/cv-binome.git'
```

Le message dit que vous n'avez pas le droit de pousser sur une branche protégée. `pre-receive hook declined` signifie que le contrôle du serveur a rejeté le push avant de l'enregistrer. Le serveur n'a donc rien reçu, et personne n'est gêné : c'est exactement le rôle de la protection.

> 🧍 **Seul.** Remettez maintenant **Allowed to push and merge** sur **Maintainers**. Sans cela, le push de dev. A serait refusé plus loin.

Votre branche `develop` locale contient maintenant un commit que le serveur n'aura jamais. Remettez-la au même point qu'`origin/develop`, la copie locale de la branche du serveur. `git reset --hard` efface ce qui n'est pas enregistré dans un commit, mais votre travail ne risque rien, car il reste sur `feature/content` :

```exec
cd ~/layers-git/binome/dev-b && git reset --hard origin/develop
cd ~/layers-git/binome/dev-b && git switch feature/content
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
if curl -fsS "$A/repository/files/cv%2Findex.html/raw?ref=develop" 2>/dev/null | grep -q 'Jane Doe'; then
  echo "Le serveur a accepté votre push, car develop contient déjà votre modification. Il manquait la protection de l'étape 03, ou seul, le réglage No one. Le conflit ne pourra plus se produire : recommencez sur un nouveau projet."
  exit 1
fi
echo "develop n'a pas bougé sur le serveur."
```

## 04.4 - 🟧 Dev. B : publier sa branche et ouvrir une merge request

> 🟧 **Dev. B**, dans `dev-b` puis sur gitlab.com.

Une branche de travail, elle, n'est pas protégée. Publiez-la :

```exec
cd ~/layers-git/binome/dev-b && git push -u origin feature/content
```

Une **merge request** est une demande de fusion : vous proposez votre branche, et une personne autorisée la relit puis la fusionne. Sur GitLab, ouvrez **Merge requests** dans le menu du projet, et cliquez sur **New merge request**. Choisissez ces deux branches :

- **Source branch** : `feature/content` ;
- **Target branch** : `develop`, à la place de `main` proposée par défaut.

Cliquez sur **Compare branches and continue**, vérifiez que le titre vous convient, désignez dev. A dans **Reviewer**, la personne chargée de relire, puis cliquez sur **Create merge request**. Seul, désignez-vous, ou laissez ce champ vide.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/merge_requests?state=opened&source_branch=feature/content&target_branch=develop" 2>/dev/null | grep -q '"iid"'; then
  echo "Aucune merge request ouverte de feature/content vers develop."
  exit 1
fi
echo "La merge request est ouverte, de feature/content vers develop."
```

➡️ **Au tour de 🟦 dev. A** : section 04.5.

## 04.5 - 🟦 Dev. A : fusionner sa feature directement dans develop

> 🟦 **Dev. A (manager)**, dans `dev-a`.
>
> ⏳ **Attendez** que dev. B ait ouvert sa merge request (section 04.4).

Votre rôle vous autorise à pousser sur `develop`. Revenez sur cette branche, récupérez les commits du serveur, fusionnez votre feature, et poussez :

```exec
cd ~/layers-git/binome/dev-a && git switch develop
cd ~/layers-git/binome/dev-a && git pull
cd ~/layers-git/binome/dev-a && git merge feature/css
cd ~/layers-git/binome/dev-a && git push
```

Cette fois, le serveur accepte. Seul, si ce push est refusé, c'est que `develop` est restée sur **No one** : revoyez la section 04.3.

> 💡 **Remarque.** Le manager passe ici par un push direct pour que vous voyiez la différence entre les deux rôles. Dans une vraie équipe, il vaut mieux que tout le monde passe par une merge request, manager compris. La relecture est utile pour chacun.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/repository/files/cv%2Findex.html/raw?ref=develop" 2>/dev/null | grep -q 'bootstrap'; then
  echo "cv/index.html sur develop ne charge pas Bootstrap."
  exit 1
fi
echo "La feature css est sur develop."
```

➡️ **Au tour de 🟧 dev. B** : section 04.6.

## 04.6 - 🟧 Dev. B : constater le conflit et le résoudre

> 🟧 **Dev. B**, sur gitlab.com puis dans `dev-b`.
>
> ⏳ **Attendez** que dev. A ait terminé la section 04.5.

Retournez sur votre merge request, sur GitLab. Elle ne peut plus être fusionnée, car GitLab y signale un conflit : dev. A et vous avez modifié la même ligne `<h1>`. La résolution se fait sur votre branche. Récupérez `develop`, et fusionnez-la dans `feature/content`. Cette fusion va s'arrêter sur le conflit :

```exec
cd ~/layers-git/binome/dev-b && git fetch
cd ~/layers-git/binome/dev-b && git merge origin/develop
```

Git s'arrête, et c'est attendu :

```text
CONFLICT (content): Merge conflict in cv/index.html
Automatic merge failed; fix conflicts and then commit the result.
```

Ouvrez `~/layers-git/binome/dev-b/cv/index.html` dans votre éditeur. Git y a écrit les deux versions de la ligne, entre des marqueurs :

```text
<<<<<<< HEAD
    <h1>Jane Doe<br>Développeuse web</h1>
=======
    <h1 class="display-5">John Doe<br>Ingénieur DevOps</h1>
>>>>>>> origin/develop
```

Gardez votre texte et la classe ajoutée par dev. A. Remplacez ces cinq lignes par cette seule ligne, sans aucun marqueur, puis enregistrez :

```html
    <h1 class="display-5">Jane Doe<br>Développeuse web</h1>
```

Terminez la fusion, puis poussez votre branche. `--no-edit` garde le message de commit proposé par Git, et la merge request se met à jour toute seule :

```exec
cd ~/layers-git/binome/dev-b && git add cv/index.html && git commit --no-edit
cd ~/layers-git/binome/dev-b && git push
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
if ! curl -fsS "$A/repository/files/cv%2Findex.html/raw?ref=feature/content" 2>/dev/null | grep -q '<h1 class="display-5">Jane Doe'; then
  echo "Sur feature/content, le titre devrait avoir la classe display-5 et le texte Jane Doe."
  exit 1
fi
if curl -fsS "$A/repository/files/cv%2Findex.html/raw?ref=feature/content" 2>/dev/null | grep -qE '^(<<<<<<<|=======|>>>>>>>)'; then
  echo "Il reste des marqueurs de conflit dans cv/index.html."
  exit 1
fi
echo "Le conflit est résolu sur feature/content."
```

➡️ **Au tour de 🟦 dev. A** : section 04.7.

## 04.7 - 🟦 Dev. A : relire et fusionner la merge request

> 🟦 **Dev. A (manager)**, sur gitlab.com.
>
> ⏳ **Attendez** que dev. B ait résolu le conflit (section 04.6).

Dev. B ne peut pas fusionner sa propre merge request, car son rôle ne l'autorise pas à modifier `develop`. C'est à vous de le faire. Ouvrez la merge request, relisez le travail dans l'onglet **Changes**, puis cliquez sur **Merge**. La case **Delete source branch** supprime la branche sur le serveur après la fusion : vous pouvez la laisser cochée, car le travail est maintenant dans `develop`.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/merge_requests?state=merged&target_branch=develop" 2>/dev/null | grep -q '"iid"'; then
  echo "La merge request vers develop n'est pas encore fusionnée."
  exit 1
fi
F=$(curl -fsS "$A/repository/files/cv%2Findex.html/raw?ref=develop" 2>/dev/null)
if ! echo "$F" | grep -q 'bootstrap' || ! echo "$F" | grep -q 'Jane Doe'; then
  echo "develop devrait contenir à la fois Bootstrap et le contenu de dev. B."
  exit 1
fi
echo "Les deux features sont sur develop."
```

➡️ **Dev. A, continuez** à l'étape 05. Dev. B reprendra à la section 05.4.

## Question

```quiz
prompt: "Pourquoi le push de dev. B sur develop est-il refusé alors que sa fusion locale a réussi ?"
answers:
  - "Parce que sa fusion locale contient un conflit"
  - "Parce que develop est protégée, et que son rôle Developer ne permet pas d'y pousser"
  - "Parce que son compte n'a pas accès au projet"
correct: 2
explanation: "La protection est vérifiée par le serveur, au moment du push. Dev. B garde le droit de pousser ses propres branches."
```

> 📖 [Merge requests](https://docs.gitlab.com/user/project/merge_requests/)
