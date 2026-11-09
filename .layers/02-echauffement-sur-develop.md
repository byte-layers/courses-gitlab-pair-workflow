---
title: "02 - Créer develop et s'échauffer avec push et pull"
description: "Une branche develop partagée, puis chacun son tour : modifier, pousser, récupérer"
---

Avant de travailler en parallèle, vous prenez vos marques : une seule branche, et chacun son tour. C'est un échauffement. La branche `develop` n'est pas encore protégée, et chacun peut y pousser directement.

## 02.1 - 🟦 Dev. A : créer et publier la branche develop

> 🟦 **Dev. A (manager)**, sur son environnement, dans le dossier `dev-a`.

Le projet a déjà `main`, qui contiendra uniquement ce qui part en production. Créez `develop`, la branche où s'assemble la prochaine version, et publiez-la pour que dev. B la voie :

```exec
cd ~/layers-git/binome/dev-a && git switch -c develop
cd ~/layers-git/binome/dev-a && git push -u origin develop
```

> 🔑 **Identifiants.** Git les demande au premier push. Si GitLab refuse votre mot de passe, ce qui arrive avec la double authentification ou une connexion par Google, créez un **personal access token**, un jeton qui remplace le mot de passe. Il se crée depuis **Edit profile**, puis **Access tokens**, avec le droit `write_repository`.

> 💡 **Remarque.** Beaucoup de tutoriels écrivent `git checkout -b develop`, qui fait la même chose. `git switch`, plus récent, est dédié aux branches, alors que `git checkout` sert aussi à restaurer des fichiers.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/repository/branches/develop" >/dev/null 2>&1; then
  echo "La branche develop n'est pas sur le serveur."
  exit 1
fi
echo "La branche develop est publiée."
```

## 02.2 - 🟦 Dev. A : créer la page du CV et la pousser

> 🟦 **Dev. A (manager)**, sur son environnement, dans le dossier `dev-a`.

Créez le dossier `cv` et sa page `index.html`, celle d'un CV fictif :

```exec
mkdir -p ~/layers-git/binome/dev-a/cv
cat > ~/layers-git/binome/dev-a/cv/index.html <<'EOF'
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CV</title>
</head>
<body>
  <header>
    <img src="./images/photo.png" alt="Photo de John Doe">
    <h1>John Doe<br>Ingénieur DevOps</h1>
  </header>
  <main>
    <section>
      <h2>Profil</h2>
      <p>Ingénieur DevOps depuis dix ans, John Doe automatise la livraison d'applications web.</p>
    </section>
    <aside>
      <h2>Contact</h2>
      <p>john.doe@example.com</p>
    </aside>
  </main>
</body>
</html>
EOF
```

Enregistrez cette page dans un commit, et poussez-le sur `develop` :

```exec
cd ~/layers-git/binome/dev-a && git add cv && git commit -m "Ajoute la page du CV"
cd ~/layers-git/binome/dev-a && git push
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
if ! curl -fsS "$A/repository/files/cv%2Findex.html?ref=develop" >/dev/null 2>&1; then
  echo "cv/index.html n'est pas sur la branche develop du serveur."
  exit 1
fi
echo "La page est sur develop. À dev. B de la récupérer."
```

➡️ **Au tour de 🟧 dev. B** : section 02.3.

## 02.3 - 🟧 Dev. B : récupérer develop

> 🟧 **Dev. B**, sur son environnement, dans le dossier `dev-b`.
>
> ⏳ **Attendez** que dev. A ait terminé la section 02.2.

Votre clone date d'avant la création de `develop`. `git fetch` récupère ce qui est nouveau sur le serveur, puis `git switch develop` crée votre branche locale à partir de celle du serveur :

```exec
cd ~/layers-git/binome/dev-b && git fetch
cd ~/layers-git/binome/dev-b && git switch develop
cd ~/layers-git/binome/dev-b && ls cv
```

Le fichier créé par dev. A est maintenant chez vous :

```text
index.html
```

Confirmez que vous avez bien récupéré le fichier.

```acknowledge
Je vois cv/index.html dans mon dossier
```

## 02.4 - 🟧 Dev. B : ajouter l'image et la pousser

> 🟧 **Dev. B**, sur son environnement, dans le dossier `dev-b`.

La page affiche une image qui n'existe pas encore. Cette commande télécharge une photo fictive, un simple carré portant les initiales JD, dans le dossier `cv/images` :

```exec
mkdir -p ~/layers-git/binome/dev-b/cv/images
curl -fsSL -o ~/layers-git/binome/dev-b/cv/images/photo.png https://raw.githubusercontent.com/byte-layers/courses-gitlab-pair-workflow/main/.layers/assets/photo.png
```

Une image se versionne comme n'importe quel fichier. Git la traite comme un fichier binaire : il en garde chaque version, mais ne sait pas montrer ce qui a changé à l'intérieur. Enregistrez-la, puis poussez à votre tour :

```exec
cd ~/layers-git/binome/dev-b && git add cv/images && git commit -m "Ajoute la photo du CV"
cd ~/layers-git/binome/dev-b && git push
```

> 🔑 **Identifiants.** Git les demande au premier push. Si GitLab refuse votre mot de passe, ce qui arrive avec la double authentification ou une connexion par Google, créez un **personal access token**, un jeton qui remplace le mot de passe. Il se crée depuis **Edit profile**, puis **Access tokens**, avec le droit `write_repository`.

Si le push répond `403`, vérifiez que vous avez bien accepté l'invitation de dev. A.

```verify
C=$(cat ~/layers-git/binome/url 2>/dev/null)
if [ -z "$C" ]; then
  echo "L'adresse du projet n'est pas enregistrée dans ~/layers-git/binome/url."
  exit 1
fi
H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#')
P=$(echo "$C" | sed -E 's#^https?://[^/]+/##; s#\.git$##; s#/#%2F#g')
A="$H/api/v4/projects/$P"
if ! curl -fsS "$A/repository/files/cv%2Fimages%2Fphoto.png?ref=develop" >/dev/null 2>&1; then
  echo "cv/images/photo.png n'est pas sur la branche develop du serveur."
  exit 1
fi
echo "L'image est sur develop. Dev. A peut la récupérer."
```

➡️ **Au tour de 🟦 dev. A** : section 02.5.

## 02.5 - 🟦 Dev. A : récupérer l'image de dev. B

> 🟦 **Dev. A (manager)**, sur son environnement, dans le dossier `dev-a`.
>
> ⏳ **Attendez** que dev. B ait terminé la section 02.4.

Avant de commencer un travail, on part d'un dépôt à jour. Récupérez le commit de dev. B, puis regardez l'état de votre dépôt :

```exec
cd ~/layers-git/binome/dev-a && git pull
cd ~/layers-git/binome/dev-a && git status
```

La sortie doit se terminer ainsi :

```text
On branch develop
Your branch is up to date with 'origin/develop'.

nothing to commit, working tree clean
```

Pour voir le résultat, ouvrez `~/layers-git/binome/dev-a/cv/index.html` dans votre navigateur. La page affiche la photo.

Confirmez lorsque votre dépôt est dans cet état.

```acknowledge
Le dépôt de dev. A est à jour, avec working tree clean
```

➡️ **Dev. A, continuez** à l'étape 03 : section 03.1.

## 02.6 - 🟧 Dev. B : vérifier que son dépôt est à jour

> 🟧 **Dev. B**, sur son environnement, dans le dossier `dev-b`.

Vous avez poussé le dernier commit, donc rien de nouveau ne vous attend. Vérifiez-le :

```exec
cd ~/layers-git/binome/dev-b && git pull
cd ~/layers-git/binome/dev-b && git status
```

La sortie est la même que chez dev. A, avec `working tree clean` à la fin.

Confirmez lorsque votre dépôt est dans cet état.

```acknowledge
Le dépôt de dev. B est à jour, avec working tree clean
```

➡️ **Dev. B, patientez** : dev. A protège les branches, et vous reprenez à la section 04.2.

## Question

```quiz
prompt: "Dev. A vient de pousser sur develop. Que doit faire dev. B pour obtenir ce commit dans sa branche develop ?"
answers:
  - "git push"
  - "git pull"
  - "git commit"
correct: 2
explanation: "`git pull` récupère les commits du serveur et les intègre à la branche courante. `git push` fait le trajet inverse."
```
