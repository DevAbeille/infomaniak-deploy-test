# Banc d'essai — Déploiement Infomaniak

Ce dépôt n'est pas un produit. C'est un test jetable dont le seul but est de PROUVER
la chaîne de déploiement continu :

> **GitHub → GitHub Actions → Infomaniak (FTPS)**

À chaque `push` sur `main`, GitHub Actions envoie le contenu de `site/` par FTPS vers
l'hébergement mutualisé Infomaniak, sur le sous-domaine `test.houseofcodesign.com`.
Aucun build, aucune dépendance : on synchronise directement le HTML pour isoler la
seule variable qui nous intéresse, le transport.

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `site/index.html` | La page publiée. Contient le **marqueur de version** à incrémenter. |
| `.github/workflows/deploy.yml` | L'automate GitHub Actions (dépôt FTPS). |
| `.gitignore` | Fichiers à ignorer. |

## Les 4 Secrets GitHub à créer

Dans le dépôt GitHub : **Settings → Secrets and variables → Actions → New repository secret**.
Les valeurs se lisent dans le **Manager Infomaniak** (Hébergement → ton site → FTP/SSH).
Ne jamais mettre ces valeurs en clair dans le code : elles restent en Secrets.

| Secret | Ce que c'est | Où le trouver |
|---|---|---|
| `FTP_SERVER` | L'hôte FTP, de la forme `xxxxx.ftp.infomaniak.com`. | Manager Infomaniak, section FTP du site. |
| `FTP_USERNAME` | L'utilisateur FTP dédié au site. | Le compte FTP créé pour ce site dans le Manager. |
| `FTP_PASSWORD` | Le mot de passe de cet utilisateur FTP. | Défini à la création du compte FTP (regénérable dans le Manager). |
| `FTP_SERVER_DIR` | Le chemin racine web du sous-domaine, ex. `/sites/test.houseofcodesign.com/`. | Manager Infomaniak, au moment de créer le site. Si le compte FTP est **chrooté** sur ce dossier, mettre simplement `./`. |

> La barre finale de `FTP_SERVER_DIR` est importante (ex. `.../test.houseofcodesign.com/`).
> Elle est déjà présente dans le workflow pour `local-dir` et `server-dir` : ne pas la retirer.

## Comment tester la chaîne

1. Ouvre `site/index.html`.
2. Change le **marqueur de version** : c'est le nombre juste sous le commentaire
   `<!-- MARQUEUR DE VERSION ... -->` (par défaut `1`). Passe-le à `2`, puis `3`, etc.
3. `git add`, `git commit`, puis `git push` sur la branche `main`.
4. Sur GitHub, onglet **Actions** : le workflow « Deploy to Infomaniak » doit passer au **vert**.
5. Ouvre `https://test.houseofcodesign.com` : le numéro affiché doit correspondre à
   celui que tu viens de pousser. S'il a changé, la chaîne continue fonctionne.

## Ordre de mise en place côté Infomaniak (à faire une seule fois)

L'ordre compte, sinon le certificat HTTPS échoue faute de DNS résolu :

1. **Créer le site / sous-domaine** `test.houseofcodesign.com` dans le Manager.
2. **Créer l'utilisateur FTP** dédié à ce site (note bien l'hôte, l'utilisateur, le mot de passe et le chemin racine).
3. **Laisser le DNS résoudre** (le sous-domaine doit pointer et se propager — quelques minutes à quelques heures).
4. **Activer le certificat HTTPS Let's Encrypt** une fois le DNS en place.
5. Renseigner les 4 Secrets sur GitHub, puis pousser pour déclencher le premier déploiement.

## Notes techniques

- Transport : **FTPS explicite sur le port 21** (pas SFTP).
- Action utilisée : `SamKirkland/FTP-Deploy-Action@v4.3.5`.
- Volontairement **sans** `dangerous-clean-slate` : on n'efface pas le distant, on synchronise.
