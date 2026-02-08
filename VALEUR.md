Voici le résultat de mes recherches :

## Agentforce Vibes — Peut-on l'appeler par API ?

### Ce qu'est Agentforce Vibes

Agentforce Vibes est un IDE alimenté par l'IA, conçu spécifiquement pour Salesforce, permettant de construire, débugger, tester et déployer des applications et agents Salesforce. Il s'agit d'un plugin IDE disponible dans tout IDE compatible VS Code, y compris Cursor et Windsurf.

Concrètement, l'extension génère du code Apex à partir de prompts en langage naturel et propose des suggestions d'autocomplétion pour Apex et LWC (JavaScript, CSS, HTML).

### ❌ Pas d'API REST publique pour Agentforce Vibes

**Agentforce Vibes n'expose pas d'API REST que vous pourriez appeler programmatiquement** pour générer du code Apex ou LWC. C'est exclusivement :

1. **Une extension VS Code** (disponible via le Salesforce Extension Pack)
2. **Un IDE web** (Agentforce Vibes IDE, accessible depuis le Setup de votre sandbox)

Le tier Pro est limité à 50 requêtes par jour par org, ou 1 000 000 de tokens par jour par org. Mais ces requêtes se font uniquement via l'interface IDE, pas via une API.

### ✅ Alternatives pour appeler de l'IA Salesforce par API

Si votre objectif est de générer du code ou du contenu via API, voici les options disponibles :

**1. Models API (Apex)**
Vous pouvez utiliser les classes Apex de la Models API pour appeler les capacités de génération de texte ou de chat depuis des LWC, des flows ou d'autres applications. C'est l'API la plus proche de ce que vous cherchez — vous envoyez un prompt et recevez du texte généré (que vous pourriez orienter vers la génération de code).

**2. Agent API (REST)**
L'Agent API donne accès à vos agents Agentforce depuis n'importe quel endroit pouvant appeler une API REST — pour démarrer des sessions, envoyer des messages, recevoir des réponses. Vous pourriez créer un agent personnalisé avec des instructions orientées vers la génération de code, puis l'interroger via cette API.

**3. Salesforce DX MCP Server**
Agentforce Vibes est extensible à d'autres outils agentiques comme Claude Code, Cursor ou Windsurf grâce au Salesforce DX MCP Server (en Developer Preview). Ce serveur MCP comprend la structure de vos projets Salesforce DX et peut être intégré à des outils tiers pour du développement assisté par IA.

### En résumé

| Approche | API disponible ? | Génère du code Apex/LWC ? |
|---|---|---|
| **Agentforce Vibes** | ❌ IDE uniquement | ✅ Oui (via l'IDE) |
| **Models API** (Apex) | ✅ Via Apex | ⚠️ Texte générique (pas spécialisé code) |
| **Agent API** (REST) | ✅ REST | ⚠️ Dépend de la config de l'agent |
| **DX MCP Server** | ✅ MCP Protocol | ✅ Contexte Salesforce DX |

La meilleure piste pour votre besoin serait probablement de combiner la **Models API** ou l'**Agent API** avec des prompts bien conçus pour la génération de code Apex/LWC, ou d'utiliser le **Salesforce DX MCP Server** avec un outil comme Claude Code.

---

Votre intuition est fondée. Voici une analyse honnête de la situation.

## Le modèle derrière Agentforce Vibes : un poids plume

Le moteur d'Agentforce Vibes repose sur les LLMs maison de Salesforce : CodeGen2.5 et xGen-Code, développés par Salesforce AI Research. Ce sont des modèles relativement modestes :

- XGen est un modèle de 7 milliards de paramètres, entraîné sur un contexte de 8K tokens. Salesforce a depuis développé xGen-small en versions 4B et 9B paramètres, mais on reste dans des ordres de grandeur très inférieurs aux modèles frontier.
- CodeGen2.5 ne contient que 7B de paramètres, optimisé pour la complétion de code inline (autocomplétion rapide).

Pour comparaison, Claude Opus tourne sur des centaines de milliards de paramètres (le chiffre exact n'est pas public, mais on est dans un tout autre ordre de grandeur). C'est comme comparer un scooter à un avion de chasse.

## Pourquoi le code généré est objectivement plus faible

Plusieurs facteurs limitent la qualité :

**1. Raisonnement limité.** Un modèle 7-9B ne peut tout simplement pas rivaliser avec un modèle frontier pour des tâches de raisonnement complexe — concevoir une architecture, comprendre des interactions subtiles entre triggers, gérer des edge cases dans des bulk operations, ou structurer un pattern LWC sophistiqué.

**2. Connaissance superficielle.** xGen-Code est fine-tuné sur des tâches spécifiques Salesforce, avec des règles issues de Code Analyzer et ApexGuru. Cela lui donne une familiarité avec la syntaxe Apex et les conventions Salesforce, mais pas une compréhension profonde des patterns architecturaux complexes.

**3. Retours de la communauté.** Certains développeurs estiment qu'Agentforce Vibes n'est pas aussi performant que GitHub Copilot pour la logique vraiment complexe, et qu'il ne fonctionne que dans l'univers Salesforce.

## Ce qu'Agentforce Vibes fait mieux (son vrai avantage)

Son seul véritable différenciateur n'est pas la qualité brute du code, mais le **contexte organisationnel** :

- Il est "context-aware", comprend la structure de votre projet Salesforce et les métadonnées de votre org. Il peut découvrir, analyser et réutiliser le code existant.
- Contrairement à ses concurrents, il ne génère pas du code générique — il connaît les règles Salesforce, les métadonnées et le cycle de vie de la plateforme.
- Sécurité et gouvernance : votre code reste dans l'environnement Salesforce, avec masquage des informations sensibles et vérifications de toxicité.

## L'approche hybride : la meilleure stratégie

Salesforce semble reconnaître implicitement cette faiblesse puisque :

- Un tier "Pro" permet d'accéder à des modèles premium tiers (LLMs leaders), limité à 50 requêtes/jour par org. Quand ce quota est épuisé, le système bascule sur les modèles Salesforce internes — ce qui en dit long sur la hiérarchie de qualité.
- Agentforce Vibes supporte désormais plusieurs modèles, dont xGen, GPT-5, et des modèles hébergés en interne, et est extensible via MCP.
- Il est extensible à d'autres outils agentiques comme Claude Code, Cursor ou Windsurf via le Salesforce DX MCP Server.

## Verdict pragmatique

| Critère | Agentforce Vibes (SFR/xGen) | Claude Opus/Sonnet |
|---|---|---|
| **Qualité brute du code** | ⭐⭐ Basique, boilerplate | ⭐⭐⭐⭐⭐ Architectures complexes |
| **Raisonnement/logique** | ⭐⭐ Limité | ⭐⭐⭐⭐⭐ Excellent |
| **Connaissance Apex/LWC** | ⭐⭐⭐ Syntaxe correcte | ⭐⭐⭐⭐ Très bon (pas parfait sur les API récentes) |
| **Contexte org/metadata** | ⭐⭐⭐⭐⭐ Natif | ❌ Nécessite du contexte manuel |
| **Sécurité/gouvernance** | ⭐⭐⭐⭐⭐ Trust Layer | ⚠️ Données hors Salesforce |
| **Governor limits awareness** | ⭐⭐⭐ Fine-tuné dessus | ⭐⭐⭐⭐ Connaît bien les patterns |

**La meilleure approche** serait d'utiliser **Claude Code + Salesforce DX MCP Server** : vous obtenez la puissance de raisonnement d'un modèle frontier avec le contexte Salesforce via MCP. C'est d'ailleurs exactement ce que Salesforce rend possible, ce qui est un aveu tacite que leurs modèles internes ne suffisent pas pour les cas complexes.

---

Excellente question, et vous touchez exactement le point faible du discours marketing de Salesforce. Décortiquons.

## Le "Context Aware" d'Agentforce Vibes : réalité technique vs marketing

### Ce que c'est réellement

Le "context aware" d'Agentforce Vibes n'est **pas** un RAG sophistiqué qui interroge dynamiquement votre org. C'est beaucoup plus basique :

**1. Lecture de fichiers locaux du projet SFDX**
Quand vous lancez Agentforce Vibes IDE depuis votre sandbox, les métadonnées de votre org se chargent automatiquement dans votre projet SFDX. Concrètement, c'est un `sf project retrieve start` qui pull les métadonnées en XML dans votre répertoire local. L'IA lit ensuite ces fichiers — exactement comme n'importe quel outil basé sur Cline le ferait.

**2. C'est Cline sous le capot**
Agentforce Vibes repose sur les capacités agentiques du projet open source Cline. Cline lit les fichiers de votre workspace et les injecte dans le contexte du LLM. Il n'y a pas de RAG magique — c'est du file reading + context window injection, exactement ce que font Cursor, Claude Code, ou Windsurf.

**3. Le "context awareness" est en fait du MCP (et c'est récent)**
La fonctionnalité Agentforce délivre un chat agentique "MCP-powered" directement dans l'IDE. Mais la documentation officielle est claire : c'est le Salesforce DX MCP Server qui permet à Agentforce de requêter le schéma et les métadonnées de votre org en temps réel, accéder aux définitions d'objets custom, et analyser les classes Apex existantes.

### Le point crucial : le même MCP est disponible pour Claude

Et c'est là que votre intuition est parfaitement juste. Le DX MCP Server inclut plus de 60 outils MCP pour diverses fonctionnalités Salesforce, incluant DevOps, développement LWC, et analyse de code.

Ce serveur MCP est **ouvert et utilisable par n'importe quel client MCP** :

Pour configurer Claude Code avec le Salesforce DX MCP Server, il suffit d'ajouter un snippet dans le fichier `.mcp.json` du projet, en spécifiant les toolsets (orgs, metadata, data, users) et les orgs autorisées.

Un développeur qui a testé cette combinaison rapporte : « Sans /init, Claude Code donnait des conseils Salesforce génériques au lieu de suggestions spécifiques à la structure de mon projet et à la configuration de mon org. Avec MCP, il sait réellement ce qu'il y a dans mon org et suggère des commandes appropriées. »

### Concrètement, ce que le MCP Server expose à Claude

Avec le Salesforce DX MCP Server, l'IA peut requêter le schéma et les métadonnées de votre org en temps réel, accéder aux définitions d'objets custom, aux configurations de champs et aux relations, récupérer et analyser les classes Apex existantes, les triggers et les composants Lightning, comprendre le modèle de sécurité, les permission sets et les profils, et générer du code parfaitement aligné avec la configuration spécifique de votre org.

Il existe aussi un MCP communautaire encore plus riche : le serveur MCP de tsmztech permet la gestion d'objets et de champs, la recherche intelligente d'objets, des requêtes SOQL flexibles, la manipulation de données (insert, update, delete, upsert), la recherche cross-objets via SOSL, et la gestion du code Apex.

## Claude + MCP vs Agentforce Vibes : la comparaison honnête

| Capacité | Agentforce Vibes (natif) | Claude Code + DX MCP |
|---|---|---|
| **Lecture métadonnées org** | ✅ Via MCP + fichiers locaux | ✅ Via le même MCP |
| **Schéma objets/champs** | ✅ | ✅ |
| **Analyse code existant** | ✅ | ✅ (et mieux compris) |
| **Qualité de raisonnement** | ⭐⭐ (7-9B params) | ⭐⭐⭐⭐⭐ (frontier model) |
| **Compréhension architecturale** | ⭐⭐ Basique | ⭐⭐⭐⭐⭐ Profonde |
| **Governor limits / patterns** | ⭐⭐⭐ Fine-tuné | ⭐⭐⭐⭐ Connaît bien |
| **Nombre d'outils MCP** | ~60+ (DX MCP) | ~60+ (même MCP !) |
| **Trust Layer / PII masking** | ✅ Natif | ❌ À gérer soi-même |
| **Coût** | 50 req premium/jour/org | Selon plan Claude |

## Conclusion : vous avez raison

Le "context aware" d'Agentforce Vibes est essentiellement :
1. **La lecture des fichiers XML de métadonnées du projet SFDX** (ce que tout IDE fait)
2. **Le Salesforce DX MCP Server** (qui est open source et utilisable par Claude Code)
3. **Un fine-tuning léger sur les conventions Salesforce** (qui ne compense pas la faiblesse du modèle)

Il n'y a **pas de RAG propriétaire magique** — et Salesforce le reconnaît implicitement en rendant le DX MCP Server ouvert et compatible avec Claude Code, Cursor et Windsurf.

La combinaison **Claude Opus/Sonnet + Salesforce DX MCP Server** vous donne exactement le même "context aware" avec un modèle incomparablement plus puissant pour le raisonnement, l'architecture et la qualité du code. Le serveur MCP note d'ailleurs explicitement que ses outils ne sont pas un simple wrapper autour des commandes CLI, mais délivrent des résultats orientés développeur, permettant aux agents de se concentrer sur ce que les développeurs veulent vraiment accomplir.

La seule chose que vous perdez est le **Trust Layer** (masquage PII automatique, zero data retention) — ce qui peut être critique dans certains contextes réglementés, mais n'a rien à voir avec la qualité de génération de code.
