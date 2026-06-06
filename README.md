# Agent de veille quotidienne IA & Cybersécurité (sur Telegram)

Cet agent vous envoie chaque matin sur Telegram une synthèse des actualités IA et
cybersécurité. Une fois installé, **vous n'avez plus rien à faire** : il tourne tout
seul, gratuitement, grâce à GitHub Actions.

---

## Installation unique (~10 minutes, que du copier-coller)

### Étape 1 — Créer votre bot Telegram (2 min)
1. Dans Telegram, ouvrez une conversation avec **@BotFather**.
2. Envoyez `/newbot`, puis suivez les instructions (nom + identifiant du bot).
3. BotFather vous donne un **token** (du type `123456:ABC-DEF...`). **Gardez-le**, c'est votre `TELEGRAM_BOT_TOKEN`.
4. Cherchez votre nouveau bot dans Telegram et envoyez-lui n'importe quel message (ex : « bonjour »). C'est important pour la suite.

### Étape 2 — Trouver votre identifiant de conversation (1 min)
1. Dans Telegram, ouvrez une conversation avec **@userinfobot**.
2. Il vous renvoie votre **Id** (un nombre, ex : `987654321`). C'est votre `TELEGRAM_CHAT_ID`.

### Étape 3 — Récupérer votre clé API Anthropic (2 min)
1. Allez sur **console.anthropic.com** → *API Keys* → *Create Key*.
2. Copiez la clé (du type `sk-ant-...`). C'est votre `ANTHROPIC_API_KEY`.

### Étape 4 — Mettre le projet sur GitHub (3 min)
1. Créez un compte gratuit sur **github.com** si vous n'en avez pas.
2. Cliquez sur **New repository**, nommez-le par ex. `veille-ia-cyber`, laissez-le **Private**, puis *Create*.
3. Sur la page du dépôt, cliquez **uploading an existing file** et glissez-y les fichiers de ce dossier :
   - `veille.py`
   - `requirements.txt`
   - le dossier `.github` (avec `workflows/veille.yml` dedans)
4. Cliquez **Commit changes**.

### Étape 5 — Enregistrer vos 3 secrets (2 min)
Dans votre dépôt GitHub : **Settings** → *Secrets and variables* → **Actions** → *New repository secret*.
Créez ces trois secrets (un par un) :

| Nom du secret        | Valeur                          |
|----------------------|---------------------------------|
| `ANTHROPIC_API_KEY`  | votre clé `sk-ant-...`          |
| `TELEGRAM_BOT_TOKEN` | le token de BotFather           |
| `TELEGRAM_CHAT_ID`   | votre Id (le nombre)            |

### Étape 6 — Tester tout de suite
Onglet **Actions** → *Veille IA & Cyber* → bouton **Run workflow**.
En 1-2 minutes, vous devriez recevoir votre première veille sur Telegram. 🎉

---

## C'est tout
À partir de là, l'agent s'exécute **automatiquement chaque jour** vers 7h-8h (heure de Paris).
Vous ne touchez plus à rien.

## Réglages éventuels
- **Changer l'heure d'envoi** : ouvrez `.github/workflows/veille.yml` et modifiez la ligne `cron`.
  L'heure est en UTC (Paris = UTC+1 en hiver, UTC+2 en été). Ex : `'0 16 * * *'` ≈ 18h Paris.
- **Changer les thèmes ou le ton** : ouvrez `veille.py`, modifiez le texte dans `build_prompt()`.

## Coût
Très faible : une exécution par jour avec recherche web revient à quelques centimes,
soit de l'ordre de 1 à 3 € par mois sur votre compte Anthropic. GitHub Actions est gratuit
pour cet usage.

## Bon à savoir
- GitHub désactive les tâches planifiées si le dépôt reste **inactif 60 jours**. Un simple
  passage tous les deux mois (ou un petit commit) suffit à le réactiver.
- Les horaires planifiés peuvent avoir quelques minutes de retard en cas de forte charge GitHub : c'est normal.
