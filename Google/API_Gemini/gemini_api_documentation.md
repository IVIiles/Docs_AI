# Google Gemini API - Documentation Technique Complète

*Document mis à jour : Décembre 2025*  
*Source officielle : https://ai.google.dev/gemini-api/docs*

---

## Table des Matières

1. [Quickstart](#1-quickstart)
2. [Architecture](#2-architecture)
3. [Implémentation](#3-implémentation)
4. [Optimisation](#4-optimisation)
5. [Maintenance](#5-maintenance)

---

## 1. Quickstart

### 1.1 Installation du SDK Google GenAI

#### Python (Python 3.9+)
```bash
pip install -q -U google-genai
```

#### JavaScript/Node.js (Node.js v18+)
```bash
npm install @google/genai
```

#### Java (Maven)
```xml
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>google-cloud-vertexai</artifactId>
  <version>1.7.0</version>
</dependency>
```

### 1.2 Authentification

1. **Obtenir une clé API** : Rendez-vous sur [Google AI Studio](https://aistudio.google.com/apikey)
2. **Configurer la variable d'environnement** :
   ```bash
   export GEMINI_API_KEY="votre-clé-api"
   ```

### 1.3 Premier Appel API

#### Python
```python
from google import genai

# Le client récupère automatiquement la clé API depuis GEMINI_API_KEY
client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Explique l'API Gemini en une phrase"
)
print(response.text)
```

#### JavaScript
```javascript
import { GoogleGenAI } from "@google/genai";

const client = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const response = await client.models.generateContent({
  model: "gemini-2.5-flash",
  contents: "Explique l'API Gemini en une phrase"
});
console.log(response.text);
```

### 1.4 Modèles Disponibles (Décembre 2025)

| Modèle | Type | Contexte | Sortie Max | Statut |
|--------|------|----------|------------|--------|
| `gemini-2.5-pro` | Polyvalent (code, raisonnement) | 1,048,576 tokens | 65,536 tokens | Stable |
| `gemini-2.5-flash` | Rapide, haute performance | 1,048,576 tokens | 65,536 tokens | Stable |
| `gemini-2.5-flash-lite` | Optimisé coût/volume | 1,048,576 tokens | 65,536 tokens | Stable |
| `gemini-2.5-flash-image` | Génération d'images (Nano Banana) | Variable | Variable | Stable |
| `gemini-2.5-flash-native-audio-preview-12-2025` | Audio natif (Live API) | Variable | Variable | Preview |
| `gemini-2.5-flash-preview-tts` | Text-to-Speech | Variable | Variable | Preview |
| `gemini-2.5-pro-preview-tts` | TTS Pro | Variable | Variable | Preview |
| `gemini-2.5-computer-use-preview-10-2025` | Agent informatique | Variable | Variable | Preview |

**Modèles Dépréciés :**
- `gemini-2.0-flash` → Déprécié
- `gemini-2.0-flash-lite` → Déprécié

---

## 2. Architecture

### 2.1 Composants Principaux

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Client                        │
│  (Python / Node.js / Java / Go / REST API directe)          │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Google GenAI SDK (Client Library)               │
│  • Gestion authentification                                  │
│  • Sérialisation requêtes                                    │
│  • Retry logic & rate limiting                               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ HTTPS
┌─────────────────────────────────────────────────────────────┐
│                  Gemini API Gateway                          │
│  Endpoint: https://generativelanguage.googleapis.com         │
│  • Routage vers modèles appropriés                           │
│  • Quotas & Rate Limiting (RPM/TPM)                          │
│  • Logging & Monitoring                                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                Model Serving Infrastructure                  │
│  • TPU v4/v5 pods                                            │
│  • GPU clusters (A100/H100)                                  │
│  • Context caching layer                                     │
│  • Token streaming engine                                    │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Flux de Données

1. **Requête** : Le client envoie un prompt avec configuration
2. **Tokenization** : Conversion texte → tokens (~4 caractères/token)
3. **Inférence** : Exécution du modèle sur TPU/GPU
4. **Streaming** : Renvoi token par token (optionnel)
5. **Post-processing** : Formatage réponse JSON/texte

### 2.3 Formats de Données Supportés

| Type | Formats Acceptés |
|------|------------------|
| **Texte** | UTF-8, Markdown, Code |
| **Images** | PNG, JPEG, WebP, GIF |
| **Vidéo** | MP4, MOV, AVI (extraction frames) |
| **Audio** | WAV, MP3, FLAC, AAC |
| **Documents** | PDF, TXT, HTML, MD |
| **Code** | Tous langages majeurs |

### 2.4 Endpoints API

| Endpoint | Méthode | Description |
|----------|---------|-------------|
| `/v1beta/models/{model}:generateContent` | POST | Génération contenu standard |
| `/v1beta/models/{model}:streamGenerateContent` | POST | Génération avec streaming |
| `/v1beta/models/{model}:countTokens` | POST | Compter tokens avant envoi |
| `/v1beta/models/{model}:embedContent` | POST | Générer embeddings |
| `/v1beta/cachedContents` | POST/GET/DELETE | Gestion cache contexte |

---

## 3. Implémentation

### 3.1 Configuration Avancée

#### Python - Avec Options Complètes
```python
from google import genai
from google.genai.types import GenerateContentConfig, SafetySetting, HarmCategory, HarmBlockThreshold

client = genai.Client()

response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents=[
        {"type": "text", "text": "Analyse ce code Python"},
        {"type": "image", "image_url": "https://example.com/code.png"}
    ],
    config=GenerateContentConfig(
        temperature=0.7,
        top_p=0.95,
        top_k=40,
        max_output_tokens=8192,
        safety_settings=[
            SafetySetting(
                category=HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT,
                threshold=HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE
            )
        ],
        system_instruction="Tu es un expert en développement Python.",
        response_mime_type="application/json",
    )
)
```

#### JavaScript - Streaming
```javascript
import { GoogleGenAI } from "@google/genai";

const client = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const stream = await client.models.generateContentStream({
  model: "gemini-2.5-flash",
  contents: "Écris une histoire interactive chapitre par chapitre"
});

for await (const chunk of stream) {
  process.stdout.write(chunk.text);
}
```

### 3.2 Gestion d'Erreurs

```python
from google.genai.errors import APIError, ResourceExhausted, InvalidArgument
import time

def generate_with_retry(client, model, prompt, max_retries=3):
    for attempt in range(max_retries):
        try:
            return client.models.generate_content(
                model=model,
                contents=prompt
            )
        except ResourceExhausted as e:
            wait_time = 2 ** attempt
            print(f"Rate limit. Attente {wait_time}s...")
            time.sleep(wait_time)
        except InvalidArgument as e:
            print(f"Erreur prompt: {e}")
            raise
        except APIError as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(1)
    
    raise Exception("Max retries exceeded")
```

### 3.3 Utilisation du Context Caching

```python
# Créer un contenu caché
cached = client.cached_contents.create(
    model="gemini-2.5-flash",
    contents={"type": "text", "text": long_document},
    ttl="2h"
)

# Utiliser le cache
response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents=[
        {"cached_content": cached.name},
        {"type": "text", "text": "Question spécifique"}
    ]
)
```

**Avantage** : Réduction de 90% du coût pour les tokens cachés.

### 3.4 Batch API (Réduction 50% des coûts)

```python
from google.genai.types import BatchJobConfig

job = client.batches.create(
    model="gemini-2.5-flash",
    requests=[{"contents": f"Traite le document {i}"} for i in range(1000)],
    config=BatchJobConfig(display_name="traitement-lot")
)

while job.state == "RUNNING":
    time.sleep(60)
    job = client.batches.get(job.name)

results = client.batches.results.get(job.name)
```

---

## 4. Optimisation

### 4.1 Optimisation des Tokens

- **1 token ≈ 4 caractères** (60-80 mots = 100 tokens)
- Utilisez `countTokens` avant gros payloads
- Compressez images avant upload

```python
token_count = client.models.count_tokens(
    model="gemini-2.5-flash",
    contents=long_prompt
)
print(f"Tokens: {token_count.total_tokens}")
```

### 4.2 Stratégies de Cache

| Scénario | Stratégie | Économie |
|----------|-----------|----------|
| Documents > 128K tokens | Context Caching | 90% |
| Prompts récurrents | System Instructions | Variable |
| Embeddings identiques | Cache embeddings | 100% |

### 4.3 Choix du Modèle

| Cas d'Usage | Modèle | Coût | Latence |
|-------------|--------|------|---------|
| Chat temps réel | gemini-2.5-flash | $ | Très faible |
| Code complexe | gemini-2.5-pro | $$$ | Moyenne |
| Volume élevé | gemini-2.5-flash-lite | ¢ | Faible |
| Documents 1M tokens | gemini-2.5-pro/flash | $$ | Moyenne |
| Audio conversationnel | gemini-2.5-flash-native-audio | $$ | Très faible |

### 4.4 Parallelisation

```python
import asyncio
from google import genai

async def process_batch(prompts):
    client = genai.Client()
    tasks = [
        client.models.generate_content_async(
            model="gemini-2.5-flash",
            contents=prompt
        )
        for prompt in prompts
    ]
    return await asyncio.gather(*tasks)

results = asyncio.run(process_batch(100_prompts))
```

---

## 5. Maintenance

### 5.1 Monitoring

| Métrique | Seuil Alerte | Action |
|----------|--------------|--------|
| Taux erreur 429 | > 5% | Augmenter tier |
| Latence P95 | > 5s | Switch Flash |
| Coût/jour | > Budget | Alertes billing |

### 5.2 Rotation Clés API

**Fréquence** : Tous les 90 jours

```bash
export GEMINI_API_KEY_NEW="nouvelle-clé"
export GEMINI_API_KEY="$GEMINI_API_KEY_NEW"
```

### 5.3 Versions de Modèles

| Version | Support | Migration |
|---------|---------|-----------|
| Stable | Illimité | Non |
| Preview | 6 mois | Oui avant EOL |
| Deprecated | 90 jours | **Urgent** |

### 5.4 Ressources Officielles

| Ressource | URL |
|-----------|-----|
| Documentation | https://ai.google.dev/gemini-api/docs |
| Pricing | https://ai.google.dev/gemini-api/docs/pricing |
| Rate Limits | https://ai.google.dev/gemini-api/docs/rate-limits |
| Cookbook | https://github.com/google-gemini/cookbook |
| Forum | https://discuss.ai.google.dev/c/gemini-api/ |

---

## Annexe A : Tarifs (Décembre 2025)

### Gemini 2.5 Pro
| Type | Prix / 1M tokens |
|------|------------------|
| Input ≤ 200K | $2.00 |
| Input > 200K | $4.00 |
| Output | $12.00 - $18.00 |

### Gemini 2.5 Flash
| Type | Prix / 1M tokens |
|------|------------------|
| Input (text/image) | $0.25 |
| Input (audio) | $0.50 |
| Output | $1.50 |

### Gemini 2.5 Flash-Lite
| Type | Prix / 1M tokens |
|------|------------------|
| Input (text/image) | $0.125 |
| Input (audio) | $0.25 |
| Output | $0.75 |

**Notes** : Batch API -50% | Free Tier limité

---

## Annexe B : Rate Limits (Tier 1 Gratuit)

| Modèle | RPM | TPM |
|--------|-----|-----|
| gemini-2.5-pro | 15 | 30,000 |
| gemini-2.5-flash | 60 | 1,000,000 |
| gemini-2.5-flash-lite | 60 | 1,000,000 |
