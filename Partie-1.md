# Part 1 : DevOps init

Dans cette première partie, on met en place le **socle d'outils DevOps** pour commencer à bosser sur la suite.

Au menu : on va utiliser **Gitlab** pour **lancer des tâches automatiquement** à chaque push sur un repo.

> On pourrait installer notre propre Gitlab, mais ça bouffe pas mal de ressources, et se taper l'install/conf, c'est chiant sur 2 jours si on peut l'éviter.

Pour bosser, **je vous file un bout de code que j'ai écrit qui lance un ptite appli web**.

![DevOps](./img/devops.jpg)

## Index

- [Part 1 : DevOps init](#part-1--devops-init)
  - [Index](#index)
  - [1. Create code repo](#1-create-code-repo)
  - [2. Grab the code](#2-grab-the-code)
  - [3. Gitlab Runner](#3-gitlab-runner)
    - [A. Intro](#a-intro)
    - [B. Install and conf](#b-install-and-conf)
  - [4. First pipeline](#4-first-pipeline)
  - [Go next](#go-next)

## 1. Create code repo


➜ **Go sur [Gitlab](https://gitlab.com)**

- créer un compte et connectez-vous si c'est pas déjà fait
- créer un nouveau dépôt git vide et **public**
- **appelez le dépôt `broken_webapp`**

➜ **Get the repo**

- clonez le dépôt sur votre machine, votre PC, votre OS (pas une VM ou koa)
- utilisez la ligne de commande `git` depuis votre PC
- poussez un ptit `README.md` tout pourri de votre choix (`add`, `commit`, `push`)

> Vérifiez que le fichier `README.md` est visible sur la WebUI avant de continuer.

🌞 **Balancez l'URL de ce dépôt `broken_webapp` dans le compte-rendu Markdown**

```
REPONSE 

 git clone https://gitlab.com/cauchemar/broken_webapp.git
 cd broken_webapp/
 vim README.md
 git add README.md
git commit -m "Mise à jour du README : ajout de la date de clonage"
git push origin main
  
```


## 2. Grab the code

🌞 **Récupérez le code**

- uniquement avec des commandes shell ~bande de noobs~ que vous me mettez dans le compte-rendu
- le code est dans ce dépôt git, à côté des fichiers markdown pour le TP, dans le sous-dossier [**`app/` ici**](./app)

🌞 **Lancez le code, histoire de voir ce qu'il fait**

- un simple `docker compose up` depuis le dossier qui contient le `docker-compose.yml` devrait suffire
- prouvez avec un `curl` que vous pouvez visiter l'application

🌞 **Connectez-vous sur la WebUI**

- des users ont été créés en base de données au moment de votre `docker compose up`
- cherchez où et comment ils ont été créés (c'est dans le dossier `app/` quelque part hein) automatiquement, pour apprendre un couple de user/password qui existe :)
- prouvez avec un `curl` que vous arrivez à vous connecter

> Si vous vous connectez en navigateur d'abord (recommandé), vous pouvez récupérer la requête qu'a fait votre navigateur sous forme de requête `curl`, c'est idéal pour rejouer une requête de connexion comme celle-ci. Demandez à Google ou Gepetto, ça se fait depuis la console du navigateur, onglet "Réseau".

🌞 **`add`, `commit`, `push`**

- poussez tout le contenu de mon dossier `app/` dans le dépôt `broken_webapp` que vous avez créé
- u know what to do, j'veux les commandes dans le compte-rendu
  - allez on est sur un TP avec `git` au coeur de toutes les opérations, essayez de pas faire des messages de commits trop nazes :d
- vous devez conserver l'architecture du dossier `app/` dans votre dépôt
  - fichier `Dockerfile` à la racine
  - fichier `docker-compose.yml` à la racine
  - dossier `src/` qui contient le code

➜ **A la racine du dépôt, vous devriez donc avoir :**

- votre `README.md`
- fichier `Dockerfile`
- fichier `docker-compose.yml`
- un dossier `src` (qui contient tout le code)
- un fichier `requirements.txt` : la liste des librairies Python nécessaires
- un env file : `.env`
- kom sa koa :

```bash
❯ ls app -a
db  docker-compose.yml  Dockerfile  .env  requirements.txt  src
```

> Cette architecture de dossier est très classique pour un projet dév.

## 3. Gitlab Runner

### A. Intro

Quand on fait de la CI/CD avec GitLab, on a besoin d'utiliser des *Runner*. Un *Runner* est une machine qui va exécuter les tests automatisés.

Concrètement :

- on fait un `git push` pour envoyer du code sur GitLab
- à la réception du `push`, Gitlab va contacter un *Runner* associé au dépôt
- il va demander au *Runner* d'exécuter les tests listés dans le fichier `.gitlab-ci.yml`

Avant de pouvoir lancer nos premiers tests automatisés, il faut donc qu'on installe et configure un *Runner GitLab*.

On va faire ça sur une VM Rocky locale, vous pouvez faire chauffer VirtualBox.

![Gitlab CI/CD](./img/logo_gitlab_cicd.png)

### B. Install and conf

➜ **Allumez une VM Rocky**

- elle doit avoir un accès internet, et vous l'administrez en SSH

🌞 **Installer Docker** sur la VM Rocky

- [suivez la doc officielle](https://docs.docker.com/engine/install/centos/) pour installer Docker SVP :'(
- `start` et `enable` le service `docker.service`

> En effet, le *Runner Gitlab* va utiliser Docker pour lancer les tests ! Pour lancer un test, le *Runner* récupère le code du dépôt Git dans un conteneur éphémère, il effectue les tests, et le conteneur est détruit à la fin des tests. La conteneurisation trouve ici un cas d'utilisation de fou : c'est juste trop pratique de pouvoir lancer des ptits conteneurs à la volée, pour effectuer des tests dedans, et détruire après :)

➜  **Il va vous falloir un token associé à votre repo**

- RDV dans la WebUI de Gitlab, sur la page principale de votre dépôt `broken_webapp`
- dans le menu latéral : `Settings > CI/CD`
- dans la page qui s'ouvre : section `Runners`
- en haut de la liste qui vient d'être déroulée, il y a les `Project runners`
- cliquez sur le bouton `Create project runner`
- dans la page qui s'ouvre, **cochez bien la case `Run untagged jobs`**, puis validez avec `Create runner`
- **vous pourrez alors récupérer votre token sur la dernière page**

🌞 **Installer un Runner Gitlab** sur la VM Rocky

- [suivez la doc officielle](https://docs.gitlab.com/runner/install/)
  - je vous recommande de l'installer directement sur Rocky avec le `.rpm` (pas de l'installer avec Docker)
  - choisissez l'executor `docker` (vous verrez de quoi il s'agit pendant que vous l'installez et le configurez)
  - il faut générer le fichier de conf avec la commande `gitlab-runner register` en fournissant un token lié à votre dépôt
  - `start` et `enable` le service `gitlab-runner.service` une fois la conf en place
- j'vous file la suite de commandes 

```bash
# si vous préférez saisir les infos de manière interactive :
sudo gitlab-runner register

# sinon, pour tout faire en une seule commande sans prompt
# remplacez $RUNNER_TOKEN par votre token
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "$RUNNER_TOKEN" \
  --executor "docker" \
  --docker-image debian:latest \
  --description "docker-runner"
```

⚠️⚠️⚠️ **Avant de continuer, assurez-vous que vous voyez votre *Runner* dans la WebUI de GitLab**

- depuis la page principale de votre repo `broken_webapp`
- dans le menu latéral, allez dans la section `Settings > CI/CD > Runners`
- vous devriez voir votre *Runner* qui s'est enregistré dans les `Project Runners`

## 4. First pipeline

Allez, notre première *pipeline*. C'est un mot générique pour désigner un truc à exécuter automatiquement dans un dépôt git.

On va donc créer notre première *pipeline* avec GitLab : **en créant un fichier `gitlab-ci.yml` à la racine du dépôt git.**

Une *pipeline* Gitlab est composée de plusieurs *stages*. Chaque *stage* consiste en une série de *job*. Chaque *job* est une commande ou une suite de commande à exécuter dans un environnement précis.

> Vous allez voir, ça va prendre du sens au fur et à mesure du TP, on va poncer ces concepts !

➜ **Ajoutez un fichier `gitlab-ci.yml` à la racine de votre dépôt `broken_webapp`**

- il doit contenir ça :

```yml
image: debian:latest

stages:
  - meow

meow-job:
  stage: meow
  before_script:
    - apt-get update -qq
    - apt-get install -y cowsay
  script:
    - /usr/games/cowsay "Meoooow"
```

🌞 **`add`, `commit`, `push`**

- pour ajouter le fichier `gitlab-ci.yml` au dépôt
- un message de commit psa tout pourri encore svp :ddd

![Break the build](./img/dont_always_commit.jpg)

➜ **RDV sur la WebUI de GitLab**

- allez sur la page principale de votre repo `broken_webapp`, et vérifiez que vous voyez bien le fichier `.gitlab-ci.yml`
- toujours depuis la page de votre repo `broken_webapp`, depuis le menu latéral, allez dans la section `Build`
- vous devriez voir votre *job* en cours d'exécution, vous pouvez avoir l'output du test

> Le *job* a été exécuté sur votre machine *Gitlab Runner*, qui a lancé un conteneur éphémère (avec Docker) pour exécuter le code demandé. Vous pouvez consulter l'output console du *job* depuis la WebUI, et vous pourrez voir que la première étape avant d'exécuter notre *job* c'est de cloner le dépôt git à l'intérieur du conteneur éphémère. Beh ui, comme ça il peut faire des tests dessus !

⚠️⚠️⚠️ **Ne continuez pas tant que vous n'avez pas vu votre *job* s'exécuter et vous devriez pouvoir voir l'output du test. Donc tu continues pas tant que t'as pas vu une vache miauler**

```
 _________
< Meoooow >
 ---------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

## Go next

👉 [**Gogogo partie 2 : Test then Build**](./part2.md)
