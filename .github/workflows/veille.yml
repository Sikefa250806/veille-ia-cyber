#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Agent de veille quotidienne IA & Cybersécurité -> Telegram.

Fonctionnement :
  1. Demande à Claude (avec recherche web) de produire une synthèse des
     actualités IA + cybersécurité des dernières 24-48h.
  2. Envoie la synthèse sur Telegram.

Variables d'environnement attendues (à mettre dans les "Secrets" GitHub) :
  - ANTHROPIC_API_KEY   : votre clé API Anthropic (console.anthropic.com)
  - TELEGRAM_BOT_TOKEN  : le token donné par @BotFather
  - TELEGRAM_CHAT_ID    : l'identifiant de votre conversation Telegram
"""

import os
import sys
import datetime
import requests
import anthropic

# ---------------------------------------------------------------------------
# Réglages que vous pouvez modifier librement
# ---------------------------------------------------------------------------
MODEL = "claude-sonnet-4-6"      # modèle utilisé (rapide et économique)
MAX_WEB_SEARCHES = 8             # nb max de recherches web par exécution
TIMEZONE_LABEL = "Europe/Paris"  # pour l'affichage de la date

# ---------------------------------------------------------------------------
# Le "cerveau" de l'agent : la consigne donnée à Claude
# ---------------------------------------------------------------------------
def build_prompt() -> str:
    today = datetime.date.today().strftime("%d/%m/%Y")
    return f"""Tu es l'assistant de veille de Mourad, fondateur de l'association IA & Citoyenneté \
(sensibilisation à l'IA et à la cybersécurité, relais ANSSI/MonAideCyber pour la région PACA) \
et de NovIA Secure (intégration IA et cybersécurité pour les entreprises).

Nous sommes le {today}. Réalise sa veille quotidienne sur DEUX thèmes, en t'appuyant sur la \
recherche web pour trouver les actualités les plus importantes des dernières 24 à 48 heures.

1) INTELLIGENCE ARTIFICIELLE : nouveaux modèles et outils marquants, réglementation (AI Act, \
CNIL...), usages en éducation et en entreprise, débats de société utiles pour la sensibilisation.

2) CYBERSÉCURITÉ : en priorité l'officiel français (ANSSI, CERT-FR, CNIL), les alertes et \
vulnérabilités majeures, les incidents et cyberattaques notables, les bonnes pratiques. \
Privilégie ce qui est utile à un public PME et grand public.

SOURCES À PRIVILÉGIER (gratuites et lisibles) :
- Cybersécurité : CERT-FR (cert.ssi.gouv.fr), ANSSI (cyber.gouv.fr), CNIL (cnil.fr), \
LeMagIT, Numerama/Cyberguerre, ZATAZ, Next (next.ink), Silicon.fr, et à l'international \
The Hacker News et BleepingComputer.
- IA : ActuIA, Le Big Data, LeMagIT, Silicon.fr, Next, France 24, Europe 1, ainsi que les \
blogs officiels (Anthropic, OpenAI, Google DeepMind, Mistral) et à l'international TechCrunch \
et Ars Technica.

ÉVITE de t'appuyer sur les médias payants dont les articles sont bloqués (Le Monde, Les Échos, \
Mediapart, Le Figaro...) : tu peux mentionner qu'ils traitent un sujet, mais récupère toujours \
le contenu via une source gratuite équivalente. Quand plusieurs sources couvrent la même actu, \
recoupe-les et garde la plus complète et la plus fiable.

Format de sortie pour Telegram (TEXTE SIMPLE, sans Markdown, sans astérisques ni dièses) :
- Une première ligne titre avec la date.
- Une section "🤖 IA" : 3 à 5 brèves. Chaque brève = un titre court, puis une phrase \
d'explication, puis l'URL de la source sur sa propre ligne.
- Une section "🔒 CYBERSÉCURITÉ" : 3 à 5 brèves, même structure.
- Une dernière ligne "💡 À retenir" : un seul enseignement concret et actionnable, \
réutilisable pour un post LinkedIn ou une formation.

Sois concis, factuel et en français. Mets les URLs en clair (Telegram les rend cliquables). \
Ne rajoute aucun commentaire en dehors de la veille elle-même."""


# ---------------------------------------------------------------------------
# Étape 1 : générer la veille via Claude + recherche web
# ---------------------------------------------------------------------------
def generate_digest() -> str:
    client = anthropic.Anthropic()  # lit automatiquement ANTHROPIC_API_KEY
    response = client.messages.create(
        model=MODEL,
        max_tokens=2500,
        tools=[{
            "type": "web_search_20250305",
            "name": "web_search",
            "max_uses": MAX_WEB_SEARCHES,
        }],
        messages=[{"role": "user", "content": build_prompt()}],
    )

    # On ne garde que les blocs de texte (on ignore les blocs techniques de recherche)
    parts = [block.text for block in response.content if getattr(block, "type", "") == "text"]
    digest = "\n".join(p for p in parts if p).strip()

    if not digest:
        digest = "⚠️ La veille n'a pas pu être générée aujourd'hui (réponse vide)."
    return digest


# ---------------------------------------------------------------------------
# Étape 2 : envoyer sur Telegram (découpe si > 4096 caractères)
# ---------------------------------------------------------------------------
def send_to_telegram(text: str) -> None:
    token = os.environ["TELEGRAM_BOT_TOKEN"]
    chat_id = os.environ["TELEGRAM_CHAT_ID"]
    url = f"https://api.telegram.org/bot{token}/sendMessage"

    # Telegram limite chaque message à 4096 caractères : on découpe proprement.
    chunks = []
    limit = 4000
    while text:
        if len(text) <= limit:
            chunks.append(text)
            break
        cut = text.rfind("\n", 0, limit)
        if cut == -1:
            cut = limit
        chunks.append(text[:cut])
        text = text[cut:].lstrip("\n")

    for chunk in chunks:
        resp = requests.post(
            url,
            json={
                "chat_id": chat_id,
                "text": chunk,
                "disable_web_page_preview": True,
            },
            timeout=30,
        )
        if not resp.ok:
            print("Erreur Telegram:", resp.status_code, resp.text, file=sys.stderr)
            resp.raise_for_status()


# ---------------------------------------------------------------------------
def main() -> None:
    print("Génération de la veille...")
    digest = generate_digest()
    print("Envoi sur Telegram...")
    send_to_telegram(digest)
    print("Terminé ✅")


if __name__ == "__main__":
    main()
