# Instructions pour Claude

Ce dépôt est le projet d'un élève. Il est mis en ligne automatiquement sur le
serveur de la classe (Dokku).

## Déploiement

- Chaque push sur la branche `main` de GitHub déploie l'app. Il n'y a rien
  d'autre à faire. N'essaie pas de te connecter au serveur en SSH : ça ne
  marche pas depuis une session cloud, et ce n'est pas nécessaire.
- Le nom de l'app est dans le fichier `.lab-app` (une ligne, en minuscules).
  Le prof le donne à l'élève. Si le fichier est vide, demande le nom à
  l'élève, écris-le dedans, commit et push sur `main`.
- Le site est à l'adresse `https://<nom>.lab.herbiserver.eu/`.
- Le workflow `.github/workflows/deploy.yml` fait le travail. Ne le modifie
  pas, sauf si le prof le demande.
- Tu peux travailler sur une branche, mais seul un push (ou un merge) sur
  `main` met en ligne. Quand l'élève demande de « mettre en ligne », push sur
  `main`.

## Après chaque push sur `main` : vérifier

Vérifie toujours le résultat, et corrige si ça a échoué. `{owner}/{repo}`
vient de `git remote get-url origin`.

1. Dernier run :
   `gh api "repos/{owner}/{repo}/actions/runs?branch=main&per_page=1" --jq '.workflow_runs[0] | {id, status, conclusion, html_url}'`
   Attends que `status` soit `completed` (réessaie toutes les 20 secondes).
2. Jobs du run :
   `gh api repos/{owner}/{repo}/actions/runs/<id>/jobs --jq '.jobs[] | {id, conclusion}'`
3. Journal complet du job :
   `gh api repos/{owner}/{repo}/actions/jobs/<job_id>/logs`
   - l'étape « Déployer » montre la construction (erreurs d'installation, de build)
   - l'étape « Journaux de l'app » montre les 80 dernières lignes du programme (erreurs au démarrage)
   - l'étape « Vérifier le site » montre le code HTTP du site

Dans une session locale où `gh` est connecté, `gh run watch` marche aussi.

Si le run se termine sans rien déployer avec une « notice », le message dit ce
qui manque : `.lab-app` vide, ou le secret `DOKKU_SSH_KEY` absent. Seul l'élève
peut ajouter le secret, sur GitHub : Settings > Secrets and variables >
Actions > New repository secret, nom `DOKKU_SSH_KEY`, valeur = la clé donnée
par le prof. Explique-lui où cliquer.

## Ce que le serveur sait faire tourner

Dokku reconnaît le langage tout seul :

- site statique : un fichier `.static` à la racine et un `index.html` (c'est
  le cas au départ) ;
- Node.js : un `package.json` avec un script `start` ; enlève alors le fichier
  `.static` ;
- Python : un `requirements.txt` et un `Procfile`, par exemple
  `web: gunicorn app:app --bind 0.0.0.0:$PORT` (avec `gunicorn` dans
  `requirements.txt`) ; enlève alors le fichier `.static` ;
- PHP : un `composer.json` (même `{}` suffit) et les fichiers `.php` ; enlève
  alors le fichier `.static` ;
- ou un `Dockerfile`.

Règles :

- Le programme écoute sur le port donné par la variable `$PORT`, jamais sur un
  port fixe.
- 512 Mo de mémoire au maximum : reste simple.
- Un seul processus web.
- Base de données : seulement si le prof l'a créée. Elle arrive par la
  variable `DATABASE_URL`. Ne mets jamais d'identifiants dans le code.
- Les fichiers écrits sur le disque de l'app disparaissent à chaque
  déploiement. Pour garder des données, il faut la base de données.

## Sécurité

- `main` est en ligne sur Internet, visible par tout le monde.
- Jamais de mot de passe, de clé ou de jeton dans le code ni dans un commit.
  Les variables secrètes passent par le prof.
- Pas de données personnelles d'autres personnes.
