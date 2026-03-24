# twitter-viral-bot2
Twitter viral Content Detector 
twitter-viral-bot/
├── bot.py
├── requirements.txt
├── .github/
│   └── workflows/
│       └── run-bot.yml
└── README.md
snscrape
pandas
import snscrape.modules.twitter as sntwitter
import pandas as pd
from datetime import datetime, timezone
import json

# ---------------------------
# CONFIG
# ---------------------------
MAX_TWEETS_PER_TREND = 50
VIRAL_THRESHOLD = 300

TRENDS = [
    "Breaking News",
    "Ukraine",
    "Tesla",
    "AI",
    "Bitcoin"
]

# ---------------------------
# VIRAL SCORE
# ---------------------------
def viral_score(tweet):
    try:
        now = datetime.now(timezone.utc)
        age_minutes = (now - tweet.date).total_seconds() / 60

        if age_minutes < 1:
            age_minutes = 1

        engagement = (
            tweet.likeCount * 1 +
            tweet.retweetCount * 2 +
            tweet.replyCount * 1.5
        )

        return engagement / age_minutes
    except Exception as e:
        return 0


# ---------------------------
# FETCH TWEETS
# ---------------------------
def get_tweets(query):
    tweets = []
    try:
        for i, tweet in enumerate(sntwitter.TwitterSearchScraper(query).get_items()):
            if i >= MAX_TWEETS_PER_TREND:
                break
            tweets.append(tweet)
        print(f"✓ {len(tweets)} tweets für '{query}' gefunden")
    except Exception as e:
        print(f"✗ Fehler bei '{query}': {e}")
    
    return tweets


# ---------------------------
# MAIN BOT
# ---------------------------
def find_viral_content():
    results = []

    for trend in TRENDS:
        print(f"🔍 Scanne: {trend}")
        tweets = get_tweets(trend)

        for t in tweets:
            score = viral_score(t)

            if score > VIRAL_THRESHOLD and t.likeCount > 100:
                results.append({
                    "trend": trend,
                    "text": t.content[:100] + "..." if len(t.content) > 100 else t.content,
                    "likes": t.likeCount,
                    "retweets": t.retweetCount,
                    "replies": t.replyCount,
                    "date": str(t.date),
                    "score": round(score, 2),
                    "url": t.url
                })

    return sorted(results, key=lambda x: x["score"], reverse=True)


if __name__ == "__main__":
    print("=" * 50)
    print("🚀 Twitter Viral Bot - GESTARTET")
    print("=" * 50)
    
    viral_posts = find_viral_content()

    with open("viral_posts.json", "w", encoding="utf-8") as f:
        json.dump(viral_posts, f, ensure_ascii=False, indent=2)

    print(f"\n✅ {len(viral_posts)} virale Posts gefunden!")
    print(f"📁 Ergebnisse: viral_posts.json")
    name: Run Twitter Bot

on:
  schedule:
    - cron: '0 */6 * * *'  # Alle 6 Stunden ausführen
  workflow_dispatch:       # Manuell ausführbar

jobs:
  run-bot:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Run bot
      run: python bot.py
    
    - name: Commit results
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add viral_posts.json
        git commit -m "🤖 Bot results: $(date)" || echo "No changes to commit"
        git push
        # 🐦 Twitter Viral Bot

Ein automatisierter Bot, der virale Tweets findet und analysiert.

## 🚀 Features
- Scannt Twitter Trends automatisch
- Berechnet Viral-Score basierend auf Engagement
- Speichert Ergebnisse in JSON
- Läuft automatisch auf GitHub Actions

## 📅 Zeitplan
- Läuft automatisch alle 6 Stunden
- Kann manuell über GitHub Actions ausgelöst werden

## 📊 Ergebnisse
Die Ergebnisse werden in `viral_posts.json` gespeichert und automatisch committet.

## 🛠️ Lokal ausführen
```bash
pip install -r requirements.txt
python bot.py
```
