# Lab 4 : Installation et exploitation de Git et Forgejo

**Auteur :** Montassar Hannachi


## 1\. Objectif

Installer Git, déployer Forgejo dans un conteneur Docker, puis utiliser Git avec un dépôt distant hébergé sur Forgejo (création du dépôt, 3 commits, publication, clonage et synchronisation).

## 2\. Environnement

|Élément|Valeur|
|-|-|
|Système|Windows 11, terminal PowerShell|
|Git|2.56.0 (Git for Windows)|
|Docker|Docker Desktop 29.8.1 avec WSL 2|
|Forgejo|Version 15, image `codeberg.org/forgejo/forgejo:15`|
|Base de données|SQLite3|
|Accès|`http://localhost:3000`|

## 3\. Installation de Git

```powershell
winget install --id Git.Git -e
git --version
```

Résultat :

```text
git version 2.56.0.windows.1
```

## 4\. Configuration de Git

```powershell
git config --global user.name "Montassar"
git config --global user.email "<montassarmh04@gmail.com>"
git config --global init.defaultBranch main
git config --list
```

Extrait du résultat :

```text
user.name=Montassar
user.email=<montassarmh04@gmail.com>
init.defaultbranch=main
```

L'option `init.defaultBranch main` évite que la branche initiale s'appelle `master`, afin de rester cohérent avec la branche `main` utilisée plus tard pour le `push`.

## 5\. Création du dépôt local et premier commit

```powershell
cd $HOME
mkdir ProjetForgejo
cd ProjetForgejo
git init
New-Item file1.txt, file2.txt
git add .
git commit -m "Initialisation du projet"
```

Résultat :

```text
Initialized empty Git repository in C:/Users/Infoshop/ProjetForgejo/.git/
\[main (root-commit) 4ee3a30] Initialisation du projet
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file1.txt
 create mode 100644 file2.txt
```

## 6\. Installation de Docker

L'étape « VM Linux » est remplacée par Docker Desktop sur Windows.

```powershell
wsl --install
winget install --id Docker.DockerDesktop -e
docker --version
docker run hello-world
```

Résultat :

```text
Docker version 29.8.1, build 4a63305

```

## 7\. Déploiement de Forgejo

```powershell
docker volume create forgejo\_data

docker run -d --name forgejo `
  -p 3000:3000 -p 2222:22 `
  -v forgejo\_data:/data `
  codeberg.org/forgejo/forgejo:15

docker ps
```

* Le volume `forgejo\_data` conserve les données de Forgejo même si le conteneur est supprimé.
* Le port `3000` donne accès à l'interface web, le port `2222` redirige vers le SSH du conteneur.

Résultat de `docker ps` :

```text
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS          PORTS                                                                                  NAMES
2bde2a3a3be5   codeberg.org/forgejo/forgejo:15   "/usr/bin/entrypoint…"   26 seconds ago   Up 24 seconds   0.0.0.0:3000->3000/tcp, \[::]:3000->3000/tcp, 0.0.0.0:2222->22/tcp, \[::]:2222->22/tcp   forgejo
```

Le conteneur `forgejo` est au statut `Up`.

## 8\. Accès à Forgejo et assistant d'installation

Ouverture de `http://localhost:3000`, puis configuration de l'assistant :

* Base de données : SQLite3
* Domaine du serveur : `localhost`
* URL de base : `http://localhost:3000/`
* Compte administrateur créé dans les paramètres facultatifs

## 9\. Création du dépôt Forgejo

Création du dépôt `tp-forgejo` depuis l'interface web, **vide** (sans README, sans .gitignore, sans licence) pour éviter un conflit au premier `push`.

## 10\. Liaison, 3 commits et publication

```powershell
git remote add origin http://localhost:3000/Montassar/tp-forgejo.git
git remote -v
```

```text
origin  http://localhost:3000/Montassar/tp-forgejo.git (fetch)
origin  http://localhost:3000/Montassar/tp-forgejo.git (push)
```

Deux commits supplémentaires :

```powershell
"# Projet Forgejo" | Set-Content README.md
git add README.md
git commit -m "Ajout du README"

Add-Content file1.txt "Contenu du fichier 1"
git add file1.txt
git commit -m "Modification de file1.txt"

git log --oneline
```

Résultat :

```text
7bc8211 (HEAD -> main) Modification de file1.txt
72d8eb1 Ajout du README
4ee3a30 Initialisation du projet
```

Publication :

```powershell
git push -u origin main
```

Résultat :

```text
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 16 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 806 bytes | 403.00 KiB/s, done.
Total 9 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To http://localhost:3000/Montassar/tp-forgejo.git
 \* \[new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

## 11\. Clonage

```powershell
cd $HOME
git clone http://localhost:3000/Montassar/tp-forgejo.git tp-forgejo-clone
cd tp-forgejo-clone
git log --oneline
```

Résultat :

```text
Cloning into 'tp-forgejo-clone'...
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (9/9), done.

7bc8211 (HEAD -> main, origin/main, origin/HEAD) Modification de file1.txt
72d8eb1 Ajout du README
4ee3a30 Initialisation du projet
```

Le clone contient exactement les 3 mêmes commits.

## 12\. Synchronisation

Modification et publication depuis le clone :

```powershell
Add-Content file2.txt "Ligne ajoutée depuis le clone"
git add file2.txt
git commit -m "Modification depuis le clone"
git push
```

```text
\[main 34141e2] Modification depuis le clone
 1 file changed, 1 insertion(+)
To http://localhost:3000/Montassar/tp-forgejo.git
   7bc8211..34141e2  main -> main
```

Récupération dans le dossier d'origine :

```powershell
cd $HOME\\ProjetForgejo
git pull
git log --oneline
```

```text
From http://localhost:3000/Montassar/tp-forgejo
   7bc8211..34141e2  main       -> origin/main
Updating 7bc8211..34141e2
Fast-forward
 file2.txt | 1 +
 1 file changed, 1 insertion(+)

34141e2 (HEAD -> main, origin/main, origin/HEAD) Modification depuis le clone
7bc8211 Modification de file1.txt
72d8eb1 Ajout du README
4ee3a30 Initialisation du projet
```

Le `pull` s'effectue en mode *Fast-forward* (sans conflit) et le 4e commit apparaît : la synchronisation fonctionne dans les deux sens.

## 13\. Conclusion

Ce lab a permis d'installer et de configurer Git, de déployer un serveur Git auto-hébergé (Forgejo) dans un conteneur Docker, puis de réaliser un cycle de travail complet : création d'un dépôt distant, commits, `push`, `clone` et synchronisation avec `pull`. L'utilisation d'un volume Docker garantit la persistance des données de Forgejo. Les difficultés rencontrées (tag d'image, dossier protégé, confusion entre terminaux) ont été résolues et documentées ci-dessus.

