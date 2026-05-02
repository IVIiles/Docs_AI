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

### 1.1 Installation du SDK Google Generative AI

#### Python (Python 3.9+)
```bash
pip install -q -U google-generativeai
```

> **Note** : Le package officiel est `google-generativeai` (importé comme `genai`).

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

| Modèle | Type | Contexte | Sortie Max | Knowledge Cutoff | Statut |
|--------|------|----------|------------|------------------|--------|
| `gemini-2.5-pro` | Polyvalent (code, raisonnement) | 2,097,152 tokens | 65,536 tokens | Août 2025 | Stable |
| `gemini-2.5-flash` | Rapide, haute performance | 1,048,576 tokens | 65,536 tokens | Août 2025 | Stable |
| `gemini-2.5-flash-lite` | Optimisé coût/volume | 1,048,576 tokens | 65,536 tokens | Août 2025 | Stable |
| `gemini-2.5-flash-image` | Génération d'images (Nano Banana) | Variable | Variable | Août 2025 | Stable |
| `gemini-2.5-flash-native-audio-preview-12-2025` | Audio natif (Live API) | Variable | Variable | Décembre 2025 | Preview |
| `gemini-2.5-flash-preview-tts` | Text-to-Speech | Variable | Variable | Décembre 2025 | Preview |
| `gemini-2.5-pro-preview-tts` | TTS Pro | Variable | Variable | Décembre 2025 | Preview |
| `gemini-2.5-computer-use-preview-10-2025` | Agent informatique | Variable | Variable | Août 2025 | Preview |

**Notes importantes :**
- **gemini-2.5-pro** supporte désormais jusqu'à **2,097,152 tokens** de contexte (2M+), un argument majeur pour les documents très longs.
- **Knowledge Cutoff** : Août 2025 pour la série 2.5 standard, Décembre 2025 pour les modèles audio preview.

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

### 3.5 Function Calling (Appel de Fonctions)

Le **Function Calling** permet au modèle d'interagir avec des outils externes (bases de données, APIs tierces) pour créer de véritables agents opérationnels.

#### Python - Déclaration et Appel de Fonctions

```python
from google import genai
from google.genai.types import GenerateContentConfig, FunctionDeclaration, Tool

client = genai.Client()

# Déclarer les fonctions disponibles
get_weather = FunctionDeclaration(
    name="get_weather",
    description="Obtenir la météo actuelle pour une ville donnée",
    parameters={
        "type": "object",
        "properties": {
            "city": {"type": "string", "description": "Nom de la ville"},
            "unit": {"type": "string", "enum": ["celsius", "fahrenheit"], "description": "Unité de température"}
        },
        "required": ["city"]
    }
)

search_database = FunctionDeclaration(
    name="search_database",
    description="Rechercher des informations dans la base de données",
    parameters={
        "type": "object",
        "properties": {
            "query": {"type": "string", "description": "Requête de recherche"}
        },
        "required": ["query"]
    }
)

# Créer un outil avec les fonctions
tools = [Tool(function_declarations=[get_weather, search_database])]

# Appel avec function calling
response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="Quelle est la météo à Paris aujourd'hui ?",
    config=GenerateContentConfig(tools=tools)
)

# Vérifier si le modèle demande d'appeler une fonction
if response.function_calls:
    for call in response.function_calls:
        print(f"Fonction appelée: {call.name}")
        print(f"Arguments: {call.args}")
        # Exécuter la fonction réelle ici
        # result = execute_function(call.name, call.args)
        # Envoyer le résultat au modèle pour continuer
```

#### JavaScript - Function Calling

```javascript
import { GoogleGenAI } from "@google/genai";

const client = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const getWeatherFunction = {
  name: "get_weather",
  description: "Obtenir la météo actuelle pour une ville donnée",
  parameters: {
    type: "object",
    properties: {
      city: { type: "string", description: "Nom de la ville" },
      unit: { type: "string", enum: ["celsius", "fahrenheit"], description: "Unité de température" }
    },
    required: ["city"]
  }
};

const response = await client.models.generateContent({
  model: "gemini-2.5-pro",
  contents: "Quelle est la météo à Paris ?",
  config: {
    tools: [{ functionDeclarations: [getWeatherFunction] }]
  }
});

if (response.functionCalls) {
  for (const call of response.functionCalls) {
    console.log(`Fonction: ${call.name}`);
    console.log(`Args: ${JSON.stringify(call.args)}`);
  }
}
```

**Cas d'usage typiques :**
- Recherche en base de données
- Appels APIs tierces (météo, stocks, actualités)
- Exécution de commandes système
- Interactions avec services externes

### 3.6 Grounding (Ancrage via Google Search)

Le **Grounding** permet au modèle de vérifier ses réponses en temps réel sur le web via l'outil `google_search_retrieval`, réduisant les hallucinations.

#### Python - Activer Google Search

```python
from google import genai
from google.genai.types import GenerateContentConfig, GoogleSearchRetrieval

client = genai.Client()

# Activer la recherche Google
search_tool = GoogleSearchRetrieval()

response = client.models.generate_content(
    model="gemini-2.5-pro",
    contents="Quelles sont les dernières nouvelles sur l'IA en décembre 2025 ?",
    config=GenerateContentConfig(
        tools=[search_tool],
        temperature=0.3  # Température basse recommandée pour grounding
    )
)

print(response.text)

# Accéder aux sources utilisées
if response.grounding_metadata and response.grounding_metadata.search_entry_point:
    print("Sources consultées:")
    for source in response.grounding_metadata.grounding_chunks:
        print(f"- {source.title}: {source.uri}")
```

#### JavaScript - Grounding avec Search

```javascript
import { GoogleGenAI } from "@google/genai";

const client = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const response = await client.models.generateContent({
  model: "gemini-2.5-pro",
  contents: "Qui a gagné le prix Nobel de physique 2025 ?",
  config: {
    tools: [{ googleSearchRetrieval: {} }]
  }
});

console.log(response.text);

// Afficher les sources
if (response.groundingMetadata?.groundingChunks) {
  console.log("Sources:");
  response.groundingMetadata.groundingChunks.forEach(chunk => {
    console.log(`- ${chunk.web?.title}: ${chunk.web?.uri}`);
  });
}
```

**Avantages du Grounding :**
- Réponses basées sur des informations à jour
- Réduction significative des hallucinations
- Citations automatiques des sources
- Idéal pour : actualités, faits récents, données vérifiées

### 3.7 Response Schema (JSON Structuré)

Pour garantir un format JSON valide, utilisez un **Response Schema** qui définit la structure attendue.

```python
from google import genai
from google.genai.types import GenerateContentConfig, ResponseSchema

client = genai.Client()

# Définir le schéma de réponse attendu
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"},
        "email": {"type": "string"},
        "skills": {"type": "array", "items": {"type": "string"}}
    },
    "required": ["name", "email"]
}

response = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Génère un profil utilisateur fictif",
    config=GenerateContentConfig(
        response_mime_type="application/json",
        response_schema=schema  # Schéma structuré requis
    )
)

# Le JSON retourné respecte toujours le schéma
import json
data = json.loads(response.text)
print(data["name"])
```

**Note** : Depuis fin 2025, le mode JSON (`application/json`) nécessite souvent un `response_schema` pour garantir la validité du format.

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
| Documents 2M tokens | gemini-2.5-pro (2M context) | $$ | Moyenne |
| Audio conversationnel | gemini-2.5-flash-native-audio | $$ | Très faible |
| Agent avec outils | gemini-2.5-pro + Function Calling | $$$ | Moyenne |
| Infos à jour | gemini-2.5-pro + Grounding | $$$ | Moyenne |

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

**Note** : gemini-2.5-pro supporte jusqu'à **2,097,152 tokens** de contexte.

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

**Notes** : 
- Batch API -50% sur tous les modèles
- Free Tier limité disponible
- Function Calling et Grounding inclus sans surcoût (comptés comme tokens normaux)

---

## Annexe B : Rate Limits (Tier 1 Gratuit)

| Modèle | RPM | TPM |
|--------|-----|-----|
| gemini-2.5-pro | 15 | 30,000 |
| gemini-2.5-flash | 60 | 1,000,000 |
| gemini-2.5-flash-lite | 60 | 1,000,000 |
