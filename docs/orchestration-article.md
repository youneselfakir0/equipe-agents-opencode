# Orchestrer OpenClaw, Hermes et OpenCode en équipe multi-agents

**Younes El Fakir** — DEP Soutien informatique, Cégep de Longueuil

> Article de portfolio. Projet personnel d'apprentissage.
> Aucun modèle payant utilisé. 100 % gratuit.

---

## Résumé

Trois agents IA orchestrés sur trois machines différentes : OpenClaw (orchestrateur),
Hermes (exécuteur IA), OpenCode (exécuteur code). Le tout avec des modèles gratuits,
des permissions système appliquées par l'OS, et une vérification SHA-256 des résultats.

```
OpenClaw (Core, Windows 11)
  ├── Hermes (Edge, Ubuntu 24.04)  — modèles API gratuits
  └── OpenCode (Core, Windows 11)  — modèles OpenCode free tier
```

---

## 1. Pourquoi orchestrer trois agents ?

Un seul agent IA qui fait tout (planifier, exécuter, vérifier) a un problème
fondamental : il ne peut pas se corriger lui-même. Quand il fait une erreur,
il la justifie.

Une équipe de trois agents avec des rôles distincts permet :

| Rôle | Agent | Responsabilité |
|---|---|---|
| Orchestrateur | OpenClaw | Découpe, délègue, vérifie les résultats |
| Exécuteur IA | Hermes | Tâches de recherche, analyse, contenu |
| Exécuteur code | OpenCode | Code, fichiers, shell, tests |

---

## 2. Architecture technique

### 2.1 OpenClaw — l'orchestrateur

OpenClaw tourne sur Core (Windows 11) en tant que Tray. Il a déjà deux nodes :
- **Core** (lui-même, Windows 11)
- **DC-DELL** (Windows Server 2025)

Pour ajouter Hermes et OpenCode comme agents distants, on utilise le pont A2A
already en place :

```
OpenClaw (Core)
  ├── node DC-DELL (Windows Server 2025) — shell, fichiers
  ├── Hermes via A2A (http://127.0.0.1:9900) — recherche, analyse
  └── OpenCode via A2A (http://127.0.0.1:9910) — code, tests
```

### 2.2 Hermes — l'exécuteur IA

Hermes est installé sur les trois machines. Il est exposé en A2A sur
`127.0.0.1:9900`. Le pont MCP dans `opencode.json` permet à OpenCode de
l'appeler directement.

**Sans Ollama** : Hermes utilise des modèles API gratuits (Pollinations,
OpenCode free tier). Aucun LLM local, pas de GPU requis.

### 2.3 OpenCode — l'exécuteur code

OpenCode tourne sur les trois machines. Le serveur A2A (`opencode-a2a-server.mjs`)
l'expose sur `127.0.0.1:9910` avec authentification par token.

---

## 3. Sécurité appliquée par le système

### 3.1 Principe du moindre privilège

Chaque agent a des permissions strictes :

| Agent | Peut lire | Peut écrire | Peut exécuter shell |
|---|---|---|---|
| OpenClaw | tout | rien (via nodes) | via nodes uniquement |
| Hermes | tout | rien | rien (pas de shell direct) |
| OpenCode | tout | tout (dans son répertoire) | oui (avec permissions) |

### 3.2 Authentification A2A

Le serveur A2A OpenCode vérifie un token Bearer sur chaque requête.
Sans token, il force le bind sur `127.0.0.1` (fail-closed).

### 3.3 Vérification des résultats

Chaque artefact rendu par un agent est vérifié par l'orchestrateur :
- Empreinte SHA-256 recalculée
- Chemin absolu confirmé
- Taille comparée

Un agent qui annonce « OK » sans preuve est rejeté.

---

## 4. Ce que j'ai mesuré

| Métrique | Avant | Après |
|---|---|---|
| Processus OpenCode actifs | 7 | 1 |
| Temps de réponse health check | N/A | < 50 ms |
| Latence OpenClaw → OpenCode | ~200 ms | ~80 ms |
| Tokens gratuits utilisés | 0 $ / mois | 0 $ / mois |

---

## 5. Limites assumées

- **Pas de coordination directe entre agents** : Hermes et OpenCode ne se parlent
  pas entre eux. Tout passe par OpenClaw.
- **Pas de mémoire partagée** : chaque agent a son propre contexte. Pour de la
  continuité, il faut passer par des fichiers.
- **Modèles gratuits uniquement** : pas de GPT-4, pas de Claude payant. Les
  modèles gratuits sont moins performants mais suffisants pour l'orchestration.

---

## 6. Compétences transférables

| Compétence | Où ça sert en support informatique |
|---|---|
| Orchestration multi-agents | Automatisation des tickets, workflows ITSM |
| Permissions et sécurité | Gestion des accès, moindre privilège |
| Scripting PowerShell | Tâches automatisées, health checks |
| Réseau (ports, A2A, MCP) | Diagnostic, résolution de problèmes |
| Documentation | Knowledge base, runbooks |

---

## 7. Reproduire l'expérience

```powershell
# 1. Cloner le depot
git clone https://github.com/youneselfakir0/equipe-agents-opencode.git

# 2. Copier les agents
Copy-Item -Recurse agents "$env:USERPROFILE\.config\opencode\"

# 3. Demarrer les services
Start-ScheduledTask -TaskName "Hermes_Gateway"
Start-ScheduledTask -TaskName "OpenCode_Serve"
Start-ScheduledTask -TaskName "OpenCode_A2A_Server"
```

---

## 8. Conclusion

L'orchestration multi-agents avec des modèles gratuits est possible et utile.
La clé, ce n'est pas le modèle — c'est la séparation des responsabilités,
les permissions système, et la vérification des résultats.

Ce projet m'a appris plus sur l'ingénierie logicielle que n'importe quel cours.
Et ça coûte 0 $ par mois.

---

**Younes El Fakir** — DEP Soutien informatique, Cégep de Longueuil
Projet personnel. Tous les fichiers sont sur https://github.com/youneselfakir0/equipe-agents-opencode