# ============ CONFIGURATION =============
# Documentation complète pour l'intégration de l'API Gemini (Gemma 4) sur InfinityFree
# CONFIG: Cette documentation utilise des variables d'environnement pour les clés API

# ============ IMPORTS =============
# (N/A - Fichier Markdown)

# ============ CONSTANTES =============
# Version de la documentation: 1.0.0
# Date de création: 2025-06-13
# Modèle ciblé: Gemma 4 (Google AI)

# ============ DOCUMENTATION =============

# 📘 Documentation Complète - API Gemini (Gemma 4)
## Intégration sur InfinityFree

---

## Table des Matières

1. [Introduction](#1-introduction)
2. [Prérequis](#2-prérequis)
3. [Configuration de l'API Google AI](#3-configuration-de-lapi-google-ai)
4. [Architecture pour InfinityFree](#4-architecture-pour-infinityfree)
5. [Implémentation PHP](#5-implémentation-php)
6. [Implémentation JavaScript](#6-implémentation-javascript)
7. [Exemples d'Utilisation](#7-exemples-dutilisation)
8. [Gestion des Erreurs](#8-gestion-des-erreurs)
9. [Optimisation et Bonnes Pratiques](#9-optimisation-et-bonnes-pratiques)
10. [Sécurité](#10-sécurité)
11. [Dépannage](#11-dépannage)
12. [Ressources Complémentaires](#12-ressources-complémentaires)

---

## 1. Introduction

### 1.1 Qu'est-ce que Gemma 4 ?

**Gemma 4** est la dernière génération de modèles de langage développés par Google, faisant partie de la famille Gemma. Ce modèle offre :

- **Performances améliorées** : Meilleure compréhension contextuelle et génération de texte
- **Multilingue** : Support natif de nombreuses langues dont le français
- **Flexible** : Différentes tailles disponibles selon vos besoins
- **Optimisé** : Conçu pour être efficace en production

### 1.2 Pourquoi l'API Gemini ?

L'API Gemini de Google Cloud permet d'accéder aux modèles Gemma sans avoir à les héberger vous-même, ce qui est idéal pour InfinityFree qui a des limitations de ressources.

**Avantages pour InfinityFree :**
- ✅ Aucun besoin de serveur puissant
- ✅ Pas de gestion de modèles ML localement
- ✅ Facturation à l'usage (quota gratuit disponible)
- ✅ Mise à jour automatique des modèles

### 1.3 Limitations d'InfinityFree à Considérer

| Contrainte | Impact | Solution |
|------------|--------|----------|
| Pas de Node.js en backend | Impossible d'exécuter du JS serveur | Utiliser PHP ou appels API côté client |
| Limites de mémoire (PHP) | Scripts limités en mémoire | Optimiser les requêtes, paginer |
| Timeout max ~30s | Requêtes longues interrompues | Utiliser async/timeout appropriés |
| Pas de cron jobs natifs | Tâches planifiées limitées | Utiliser des appels déclenchés par visite |

---

## 2. Prérequis

### 2.1 Compte Google Cloud

1. Rendez-vous sur [Google Cloud Console](https://console.cloud.google.com/)
2. Créez un nouveau projet ou sélectionnez-en un existant
3. Activez l'API Gemini

### 2.2 Clé API

1. Accédez à [Google AI Studio](https://aistudio.google.com/)
2. Cliquez sur **"Get API Key"**
3. Créez une nouvelle clé API
4. **⚠Important** : Stockez cette clé securely (voir section Sécurité)

### 2.2 Quotas et Tarification

**Quota Gratuit (au moment de la rédaction) :**
- 60 requêtes par minute
- 1 500 000 tokens par mois (modèle Gemma 4)

**Tarification payante :**
- Consultez [la page tarifaire officielle](https://ai.google.dev/pricing) pour les tarifs à jour

### 2.3 Configuration InfinityFree

Vérifiez que votre hébergement InfinityFree supporte :
- ✅ PHP 7.4+ (recommandé : 8.0+)
- ✅ cURL activé
- ✅ JSON activé
- ✅ HTTPS sortant autorisé

---

## 3. Configuration de l'API Google AI

### 3.1 Activer l'API Gemini

```bash
# Via Google Cloud Console
1. Menu "APIs & Services" > "Library"
2. Recherchez "Gemini API"
3. Cliquez sur "Enable"
```

### 3.2 Créer une Clé API

```bash
# Via Google AI Studio
1. Visitez https://aistudio.google.com/app/apikey
2. Cliquez sur "Create API Key"
3. Sélectionnez votre projet Google Cloud
4. Copiez la clé générée (format: AIzaSy...)
```

### 3.3 Restreindre la Clé API (Recommandé)

```bash
# Dans Google Cloud Console :
1. Allez dans "APIs & Services" > "Credentials"
2. Cliquez sur votre clé API
3. Section "Application restrictions" :
   - Sélectionnez "HTTP referrers"
   - Ajoutez : votre-domaine.infinityfreeapp.com
   - Ajoutez : *.votredomaine.com
4. Section "API restrictions" :
   - Sélectionnez "Restrict key"
   - Cochez uniquement "Gemini API"
```

---

## 4. Architecture pour InfinityFree

### 4.1 Structure de Fichiers Recommandée

```
htdocs/
├── api/
│   ├── gemini/
│   │   ├── config.php           # Configuration et constantes
│   │   ├── GeminiClient.php     # Client API principal
│   │   ├── models/
│   │   │   ├── Request.php      # Classe de requête
│   │   │   └── Response.php     # Classe de réponse
│   │   └── exceptions/
│   │       └── GeminiException.php
│   └── index.php                # Point d'entrée API
├── assets/
│   ├── js/
│   │   └── gemini-app.js        # Frontend JavaScript
│   └── css/
│       └── styles.css
├── includes/
│   └── functions.php            # Fonctions utilitaires
├── .env.example                 # Template de variables d'environnement
└── index.php                    # Page principale
```

### 4.2 Diagramme de Flux

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│  Navigateur │ ───► │ InfinityFree │ ───► │  API Gemini │
│   (Client)  │ ◄─── │    (PHP)     │ ◄─── │   (Google)  │
└─────────────┘      └──────────────┘      └─────────────┘
       │                      │                      │
       │  1. Requête HTTP     │                      │
       │─────────────────────►│                      │
       │                      │  2. Appel API        │
       │                      │─────────────────────►│
       │                      │  3. Réponse JSON     │
       │                      │◄─────────────────────│
       │  4. Réponse HTML/JSON│                      │
       │◄─────────────────────│                      │
```

### 4.3 Modèles de Communication

**Option A : Proxy PHP (Recommandé)**
- Le frontend appelle votre script PHP
- PHP appelle l'API Gemini
- PHP retourne la réponse au frontend

**Avantages :**
- ✅ Clé API cachée côté serveur
- ✅ Contrôle des quotas
- ✅ Logging et monitoring
- ✅ Validation des entrées

**Option B : Appel Direct (Déconseillé)**
- Le frontend appelle directement l'API Gemini
- Nécessite d'exposer la clé API (risque de sécurité)

---

## 5. Implémentation PHP

### 5.1 Fichier de Configuration (`api/gemini/config.php`)

```php
<?php
// ============ CONFIGURATION =============
// CONFIG: Variables d'environnement pour clés API et paramètres

// ============ IMPORTS =============
// (Aucun import externe nécessaire)

// ============ CONSTANTES =============

/**
 * Version du client Gemini
 */
define('GEMINI_CLIENT_VERSION', '1.0.0');

/**
 * URL de base de l'API Gemini
 * [NON VÉRIFIÉ] Peut évoluer selon les mises à jour Google
 */
define('GEMINI_API_BASE_URL', 'https://generativelanguage.googleapis.com/v1beta');

/**
 * Modèle par défaut (Gemma 4)
 * [HYPOTHÈSE] Vérifiez le nom exact dans la documentation Google
 */
define('GEMINI_DEFAULT_MODEL', 'gemma-4-latest');

/**
 * Timeout des requêtes en secondes
 */
define('GEMINI_TIMEOUT', 30);

/**
 * Nombre maximum de tentatives de retry
 */
define('GEMINI_MAX_RETRIES', 3);

/**
 * Délai entre les retries en millisecondes
 */
define('GEMINI_RETRY_DELAY_MS', 1000);

// ============ FONCTIONS =============

/**
 * Charge la clé API depuis les variables d'environnement ou fichier .env
 *
 * @return string|null La clé API ou null si non trouvée
 * @throws RuntimeException Si la clé API est manquante
 */
function getGeminiApiKey() {
    // Essayer d'abord les variables serveur
    $apiKey = getenv('GEMINI_API_KEY');
    
    if ($apiKey === false) {
        $apiKey = $_ENV['GEMINI_API_KEY'] ?? null;
    }
    
    // Si non trouvé, essayer de charger depuis .env
    if ($apiKey === null && file_exists(__DIR__ . '/../../.env')) {
        $envContent = file_get_contents(__DIR__ . '/../../.env');
        if (preg_match('/GEMINI_API_KEY=(.+)/', $envContent, $matches)) {
            $apiKey = trim($matches[1]);
        }
    }
    
    if (empty($apiKey)) {
        throw new RuntimeException(
            'Clé API Gemini manquante. Définissez GEMINI_API_KEY dans vos variables d\'environnement.'
        );
    }
    
    return $apiKey;
}

/**
 * Valide le format d'une clé API Gemini
 *
 * @param string $key La clé à valider
 * @return bool True si valide, false sinon
 */
function validateGeminiApiKey($key) {
    // Les clés API Google commencent généralement par "AIza"
    return preg_match('/^AIza[A-Za-z0-9_-]{35}$/', $key) === 1;
}

// ============ EXPORTS =============
// Les fonctions sont accessibles via require/include
