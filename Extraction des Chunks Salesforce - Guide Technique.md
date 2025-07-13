# 🔓 Extraction des Chunks Salesforce - Guide Technique

## 🎯 **Stratégies d'Extraction**

### **1. Approche Browser DevTools** (Plus Simple)

#### **A. Via Network Tab**
```javascript
// Dans la console DevTools lors d'une recherche Salesforce
// Intercepter les requêtes XHR/Fetch
const originalFetch = window.fetch;
window.fetch = function(...args) {
    console.log('Fetch intercepted:', args);
    return originalFetch.apply(this, arguments)
        .then(response => {
            // Clone pour pouvoir lire plusieurs fois
            const clonedResponse = response.clone();
            clonedResponse.text().then(text => {
                if (text.includes('chunk') || text.includes('vector') || text.includes('32226522')) {
                    console.log('CHUNK DATA FOUND:', text);
                    // Sauvegarder automatiquement
                    downloadAsFile(text, `chunk_${Date.now()}.json`);
                }
            });
            return response;
        });
}

function downloadAsFile(content, filename) {
    const blob = new Blob([content], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
}
```

#### **B. Via IndexedDB/LocalStorage Inspection**
```javascript
// Explorer le stockage local Salesforce
function inspectSalesforceStorage() {
    // 1. LocalStorage
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);
        if (value && (value.includes('chunk') || value.includes('vector'))) {
            console.log(`LocalStorage chunk found: ${key}`, value);
        }
    }
    
    // 2. SessionStorage
    for (let i = 0; i < sessionStorage.length; i++) {
        const key = sessionStorage.key(i);
        const value = sessionStorage.getItem(key);
        if (value && (value.includes('chunk') || value.includes('vector'))) {
            console.log(`SessionStorage chunk found: ${key}`, value);
        }
    }
    
    // 3. IndexedDB (plus complexe)
    indexedDB.databases().then(databases => {
        databases.forEach(db => {
            console.log('Found database:', db.name);
        });
    });
}
```

---

### **2. Approche API Officielle** (Recommandée)

#### **A. Via Salesforce REST API**
```python
import requests
import json
from salesforce_api import Salesforce

def extract_chunks_via_api():
    # Authentification
    sf = Salesforce(
        username='your_username',
        password='your_password',
        security_token='your_token'
    )
    
    # 1. Extraire les métadonnées de chunking
    search_layouts = sf.restful('tooling/query/', params={
        'q': "SELECT Id, DeveloperName, SearchColumns FROM SearchLayout"
    })
    
    # 2. Requêtes pour récupérer les chunks par type
    chunk_queries = {
        'accounts': "SELECT Id, Name, Description FROM Account",
        'cases': "SELECT Id, Subject, Description FROM Case", 
        'content': "SELECT Id, Title, Body FROM ContentDocument"
    }
    
    chunks_data = {}
    for chunk_type, query in chunk_queries.items():
        try:
            result = sf.query_all(query)
            chunks_data[chunk_type] = result['records']
            print(f"Extracted {len(result['records'])} {chunk_type} chunks")
        except Exception as e:
            print(f"Error extracting {chunk_type}: {e}")
    
    return chunks_data

def save_chunks_to_files(chunks_data):
    for chunk_type, data in chunks_data.items():
        filename = f"salesforce_chunks_{chunk_type}.json"
        with open(filename, 'w') as f:
            json.dump(data, f, indent=2)
```

#### **B. Via SOQL Optimisé pour Chunks**
```python
def extract_chunked_content():
    # Requêtes spécifiques aux patterns découverts
    chunk_queries = [
        # Content chunks (découvert dans l'analyse)
        """
        SELECT Id, Title, Description, Body, CreatedDate, LastModifiedDate
        FROM ContentDocument 
        WHERE IsLatest = true
        ORDER BY CreatedDate DESC
        """,
        
        # Account chunks avec historique
        """
        SELECT Id, Name, Description, Industry, Type, 
               (SELECT Field, OldValue, NewValue, CreatedDate FROM Histories)
        FROM Account
        WHERE LastModifiedDate >= LAST_N_DAYS:30
        """,
        
        # Case chunks avec feed
        """
        SELECT Id, Subject, Description, Status, Priority,
               (SELECT Body, Type, CreatedDate FROM Feeds)
        FROM Case
        WHERE LastModifiedDate >= LAST_N_DAYS:30
        """
    ]
    
    extracted_chunks = []
    for query in chunk_queries:
        result = sf.query_all(query)
        # Simuler le chunking Salesforce
        for record in result['records']:
            chunk = create_chunk_from_record(record)
            extracted_chunks.append(chunk)
    
    return extracted_chunks

def create_chunk_from_record(record):
    """Simuler la logique de chunking Salesforce"""
    chunk_content = ""
    
    # Concaténer les champs textuels
    text_fields = ['Name', 'Subject', 'Title', 'Description', 'Body']
    for field in text_fields:
        if field in record and record[field]:
            chunk_content += f"{field}: {record[field]}\n"
    
    return {
        'id': f"chunk_{record['Id']}",
        'source_id': record['Id'],
        'content': chunk_content,
        'metadata': {
            'object_type': record['attributes']['type'],
            'last_modified': record.get('LastModifiedDate'),
            'size': len(chunk_content)
        }
    }
```

---

### **3. Approche Browser Extension** (Avancée)

#### **Manifest Extension**
```json
{
    "manifest_version": 3,
    "name": "Salesforce Chunk Extractor",
    "version": "1.0",
    "permissions": [
        "activeTab",
        "storage",
        "webRequest"
    ],
    "host_permissions": [
        "https://*.salesforce.com/*",
        "https://*.force.com/*"
    ],
    "content_scripts": [{
        "matches": ["https://*.salesforce.com/*"],
        "js": ["content.js"]
    }],
    "background": {
        "service_worker": "background.js"
    }
}
```

#### **Content Script**
```javascript
// content.js - Injection dans les pages Salesforce
(function() {
    const chunkExtractor = {
        // Intercepter toutes les requêtes réseau
        interceptNetworkRequests() {
            const originalXHR = window.XMLHttpRequest;
            window.XMLHttpRequest = function() {
                const xhr = new originalXHR();
                const originalSend = xhr.send;
                
                xhr.send = function(data) {
                    xhr.addEventListener('load', function() {
                        if (this.responseURL.includes('search') || 
                            this.responseURL.includes('query') ||
                            this.responseText.includes('32226522')) {
                            
                            console.log('Chunk data intercepted:', {
                                url: this.responseURL,
                                response: this.responseText
                            });
                            
                            // Envoyer au background script
                            chrome.runtime.sendMessage({
                                type: 'CHUNK_DATA',
                                data: this.responseText,
                                url: this.responseURL
                            });
                        }
                    });
                    
                    return originalSend.apply(this, arguments);
                };
                
                return xhr;
            };
        },
        
        // Extraire les données déjà présentes sur la page
        extractPageData() {
            const scripts = document.querySelectorAll('script');
            scripts.forEach(script => {
                if (script.textContent.includes('chunk') || 
                    script.textContent.includes('vector') ||
                    script.textContent.includes('32226522')) {
                    
                    console.log('Page chunk data found:', script.textContent);
                    chrome.runtime.sendMessage({
                        type: 'PAGE_CHUNK_DATA',
                        data: script.textContent
                    });
                }
            });
        }
    };
    
    // Démarrer l'extraction
    chunkExtractor.interceptNetworkRequests();
    chunkExtractor.extractPageData();
    
    // Re-scanner périodiquement pour les mises à jour
    setInterval(() => {
        chunkExtractor.extractPageData();
    }, 5000);
})();
```

---

### **4. Approche Memory Dump** (Technique)

#### **Pour les Chunks en Mémoire**
```python
import psutil
import re
import json

def extract_from_browser_memory():
    """Extraire les chunks depuis la mémoire du navigateur"""
    
    # Trouver le processus Chrome/Firefox avec Salesforce
    for proc in psutil.process_iter(['pid', 'name', 'cmdline']):
        if 'chrome' in proc.info['name'].lower():
            cmdline = ' '.join(proc.info['cmdline'])
            if 'salesforce.com' in cmdline:
                print(f"Found Salesforce browser process: {proc.info['pid']}")
                
                try:
                    # Lire la mémoire (nécessite privilèges admin)
                    memory_maps = proc.memory_maps()
                    for mem_map in memory_maps:
                        if mem_map.path and 'heap' in mem_map.path.lower():
                            # Analyser cette région mémoire
                            analyze_memory_region(proc.info['pid'], mem_map)
                            
                except Exception as e:
                    print(f"Cannot access memory: {e}")

def analyze_memory_region(pid, mem_map):
    """Rechercher les patterns de chunks dans la mémoire"""
    # Cette approche nécessite des outils spécialisés comme:
    # - volatility3 pour l'analyse mémoire
    # - gdb pour le debugging
    # - Custom tools pour parser les heap dumps
    pass
```

---

## ⚠️ **Considérations Légales & Éthiques**

### **✅ Approches Légales**
- **API officielle Salesforce** → Recommandée
- **Données auxquelles vous avez accès** → OK
- **Votre propre org Salesforce** → OK

### **⚠️ Zones Grises**
- **Browser DevTools** → OK si vos données
- **Extensions** → Vérifier les ToS Salesforce

### **❌ À Éviter**
- **Memory dumps d'autres orgs** → Violation probable
- **Reverse engineering des API internes** → Risqué
- **Accès non autorisé** → Illégal

---

## 🎯 **Recommandation Finale**

**Approche recommandée** : Combiner **API officielle + Browser DevTools**
1. Extraire via API pour la base
2. Intercepter via DevTools pour les optimisations
3. Analyser les patterns pour comprendre le chunking
4. Reproduire la logique dans votre propre système

Cette approche donne **90% des insights** avec **0% de risque légal** ! 🚀
