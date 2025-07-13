# 🔍 Interception Agentforce VS Code - Guide Technique

## 🎯 **Architecture Agentforce for Developers**

```
VS Code Extension → Language Server → Salesforce APIs → Vector Search
      ↓                    ↓              ↓              ↓
   User Code         Code Analysis    Metadata API    Embedding Store
```

---

## 🕵️ **Méthodes d'Interception**

### **1. Analyse de l'Extension VS Code** (Plus Direct)

#### **A. Localiser l'Extension**
```bash
# Chemin typique des extensions VS Code
# Windows
%USERPROFILE%\.vscode\extensions\salesforce.*

# macOS  
~/.vscode/extensions/salesforce.*

# Linux
~/.vscode/extensions/salesforce.*

# Chercher spécifiquement Agentforce
find ~/.vscode/extensions -name "*salesforce*" -o -name "*agentforce*"
```

#### **B. Analyser les Fichiers de l'Extension**
```bash
cd ~/.vscode/extensions/salesforce.agentforce-*

# 1. Manifest de l'extension
cat package.json | jq '.contributes'

# 2. Scripts principaux  
ls -la out/ src/
cat out/extension.js | grep -i "chunk\|vector\|embedding"

# 3. Configuration réseau
grep -r "api\|endpoint\|url" . --include="*.js" --include="*.json"
```

---

### **2. Monitoring Réseau VS Code**

#### **A. Via Process Monitor**
```bash
# 1. Identifier le processus VS Code avec Agentforce
ps aux | grep -i code
ps aux | grep -i electron

# 2. Monitor les connexions réseau
# macOS/Linux
lsof -p <VSCODE_PID> | grep TCP
netstat -p <VSCODE_PID>

# Windows  
netstat -ano | findstr <VSCODE_PID>
```

#### **B. Via Proxy Interception**
```bash
# Configurer un proxy local pour intercepter
# 1. Démarrer mitmproxy
mitmproxy -p 8080 --set confdir=~/.mitmproxy

# 2. Configurer VS Code pour utiliser le proxy
code --proxy-server=http://localhost:8080

# 3. Les requêtes Agentforce apparaîtront dans mitmproxy
```

---

### **3. Debugging de l'Extension**

#### **A. Mode Debug VS Code**
```bash
# 1. Lancer VS Code en mode développeur
code --inspect-extensions=9229

# 2. Ouvrir Chrome DevTools
# Aller à chrome://inspect
# Connecter au port 9229
```

#### **B. Extension Development Host**
```javascript
// Dans VS Code, ouvrir Command Palette (Cmd+Shift+P)
// > Developer: Reload Window With Extensions Disabled
// > Developer: Show Running Extensions

// Puis dans l'Extension Host Debug Console:
console.log = (function(originalLog) {
    return function(...args) {
        if (args.some(arg => 
            typeof arg === 'string' && 
            (arg.includes('chunk') || arg.includes('vector') || arg.includes('agentforce'))
        )) {
            originalLog('🎯 AGENTFORCE DATA:', ...args);
        }
        return originalLog.apply(this, args);
    };
})(console.log);
```

---

### **4. Interception des Communications**

#### **A. Hook des APIs Extension**
```javascript
// Créer une extension VS Code pour intercepter
// package.json
{
    "name": "agentforce-interceptor",
    "main": "./out/extension.js",
    "activationEvents": ["*"],
    "contributes": {
        "commands": [{
            "command": "agentforce.intercept",
            "title": "Intercept Agentforce"
        }]
    }
}

// extension.js
const vscode = require('vscode');

function activate(context) {
    // Intercepter les appels réseau de l'extension Agentforce
    const originalRequire = require;
    require = function(id) {
        const module = originalRequire(id);
        
        if (id.includes('https') || id.includes('http')) {
            // Wrapper pour les requêtes HTTP
            const originalRequest = module.request;
            module.request = function(...args) {
                console.log('🔍 HTTP Request intercepted:', args);
                const req = originalRequest.apply(this, args);
                
                req.on('response', (res) => {
                    let data = '';
                    res.on('data', chunk => data += chunk);
                    res.on('end', () => {
                        if (data.includes('chunk') || data.includes('vector')) {
                            console.log('🎯 Agentforce response:', data);
                            vscode.workspace.fs.writeFile(
                                vscode.Uri.file(`/tmp/agentforce_${Date.now()}.json`),
                                Buffer.from(data)
                            );
                        }
                    });
                });
                
                return req;
            };
        }
        
        return module;
    };
}
```

#### **B. Language Server Protocol Interception**
```javascript
// Intercepter les communications Language Server
const net = require('net');
const fs = require('fs');

// VS Code communique avec Language Servers via JSON-RPC
// Intercepter les messages
function interceptLSP() {
    const originalConnect = net.connect;
    net.connect = function(...args) {
        const socket = originalConnect.apply(this, args);
        
        socket.on('data', (data) => {
            const message = data.toString();
            if (message.includes('agentforce') || 
                message.includes('chunk') || 
                message.includes('vector')) {
                
                console.log('🎯 LSP Message:', message);
                fs.appendFileSync('/tmp/agentforce_lsp.log', message + '\n');
            }
        });
        
        return socket;
    };
}
```

---

### **5. Analyse des Logs VS Code**

#### **A. Localiser les Logs**
```bash
# Logs VS Code standard
# Windows
%APPDATA%\Code\logs\

# macOS
~/Library/Application Support/Code/logs/

# Linux  
~/.config/Code/logs/

# Chercher logs Agentforce spécifiques
find . -name "*.log" -exec grep -l "agentforce\|chunk\|vector" {} \;
```

#### **B. Monitoring en Temps Réel**
```bash
# Surveiller les logs en temps réel
tail -f ~/.config/Code/logs/*/exthost/output_logging_* | grep -i agentforce

# Ou avec filtrage plus spécifique
tail -f ~/.config/Code/logs/*/exthost/* | grep -E "(chunk|vector|embedding|32226522)"
```

---

### **6. Memory Dump de VS Code**

#### **A. Process Dump**
```bash
# 1. Identifier le PID du processus Extension Host
ps aux | grep "extensionHost"

# 2. Créer un dump mémoire
# Linux
gcore <EXTENSION_HOST_PID>

# macOS
sample <EXTENSION_HOST_PID> 10 -file agentforce_dump.txt

# 3. Analyser le dump pour les chunks
strings core.<PID> | grep -E "(chunk|vector|32226522|agentforce)"
```

#### **B. Heap Snapshot**
```bash
# Si VS Code utilise Node.js (ce qui est le cas)
kill -USR1 <EXTENSION_HOST_PID>  # Génère heap snapshot
# Le fichier apparaît dans le répertoire de travail
```

---

## 🛠️ **Script Automation Complet**

```bash
#!/bin/bash
# agentforce_interceptor.sh

echo "🎯 Starting Agentforce Interception..."

# 1. Setup proxy
mitmproxy -p 8080 --set confdir=~/.mitmproxy &
PROXY_PID=$!

# 2. Monitor VS Code processes
ps aux | grep -i code > vscode_processes.txt

# 3. Start network monitoring
lsof -i | grep -i code > network_connections.txt &

# 4. Launch VS Code with proxy
code --proxy-server=http://localhost:8080 &

# 5. Monitor logs
tail -f ~/.config/Code/logs/*/exthost/* | grep -E "(chunk|vector|agentforce)" > agentforce_logs.txt &

echo "✅ Monitoring active. Check:"
echo "   - mitmproxy UI: http://localhost:8081"  
echo "   - Logs: agentforce_logs.txt"
echo "   - Network: network_connections.txt"

# Cleanup on exit
trap "kill $PROXY_PID; pkill -f tail" EXIT
wait
```

---

## 🎯 **Cibles Prioritaires**

### **Fichiers à Analyser**
1. **Extension manifest**: `package.json`
2. **Code principal**: `out/extension.js`
3. **Language Server**: `out/languageServer.js`  
4. **Configuration**: `contributes.configuration`

### **Network Endpoints à Surveiller**
- `*.salesforce.com/services/data/*`
- `*.force.com/services/*`
- `api.agentforce.com/*` (hypothétique)
- WebSocket connections pour real-time

### **Patterns de Données**
- JSON avec `chunk`, `vector`, `embedding`
- IDs pattern `32226522-*`
- Apex class metadata
- Code completion responses

---

## ⚠️ **Considérations**

### **✅ Legal & Safe**
- Analyser **votre propre** installation VS Code
- Extensions que **vous avez installées**
- **Vos propres** projets Salesforce

### **⚠️ À Noter**
- Extensions souvent **obfusquées/minifiées**
- Communications peuvent être **chiffrées**
- Rate limiting sur les APIs

---

## 🚀 **Quick Start**

```bash
# Méthode la plus simple
cd ~/.vscode/extensions/salesforce*
grep -r "chunk\|vector\|embedding" . --include="*.js"
```

Cette approche devrait révéler comment Agentforce chunke et vectorise votre code Salesforce ! 🎯
