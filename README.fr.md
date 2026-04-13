# Hello Hermes Agent ☤

C'est un espace de travail pour explorer et analyser [Hermes Agent](https://github.com/nousresearch/hermes-agent) `v0.8.0 (v2026.4.8)` `86960cdb`.

> ## Prononciation
> 
> Note : Le nom « Hermes » dans ce projet fait référence à la divinité grecque.
> 
> ✔️ **Hermes** : `/ˈhɜːrmiːz/` — Le dieu grec du langage et de l'écriture, messager des dieux.
> 
> ✖️ **Hermès** : `/ɛʁ.mɛs/` — Marque de luxe française.

## Analyse du Code Source

```sh
git clone --depth 1 --branch v2026.4.8 https://github.com/nousresearch/hermes-agent
```

- [Analyse de l'architecture Hermes (1ère partie) : Flux · Cycle de vie complet de l'exécution du code source](./Hermes%20架构解析%20(一)：流程篇%20·%20源代码执行全生命周期.md)
- [Analyse de l'architecture Hermes (2ème partie) : Données · Modèle d'état et gouvernance du contexte](./Hermes%20架构解析%20(二)：数据篇%20·%20状态模型与上下文治理.md)
- [Analyse de l'architecture Hermes (3ème partie) : Extension · Guide complet du développement de plugins et de skills](./Hermes%20架构解析%20(三)：扩展篇%20·%20插件与技能开发全指南.md)

## Démarrage Rapide

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
# Windows
powershell -Command "Set-ExecutionPolicy Bypass -Scope Process -Force; irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex"
```

```bash
# Lancer l'assistant de configuration
hermes setup

# Voir/modifier la configuration
code ~/.hermes/

# Démarrer le chat interactif
hermes
```

## Débogage avec Points d'arrêt PyCharm

Débogage de `hermes-agent/` avec des points d'arrêt PyCharm :

### 1. Configurer le projet

```bash
cd hermes-agent
uv venv venv --python 3.11
source venv/bin/activate   # Windows : venv\Scripts\activate
uv pip install -e ".[all,dev]"
```

### 2. Configurer PyCharm

1. **Interpréteur Python** : File → Project Structure → Add Python Interpreter, sélectionnez `hermes-agent/venv/bin/python` (ou `venv\Scripts\python.exe` sous Windows)
2. **Racine du SDK** : Project → Project Structure, marquez `hermes-agent/` comme Sources
3. **Répertoire de travail** : Project Structure → Modules → Paths → Add Content Root, ajoutez `hermes-agent/src` (si existant)

### 3. Configurer l'exécution

| Champ | Valeur |
|-------|--------|
| **Script path** | `hermes-agent/src/hermes_cli/main.py` |
| **Working directory** | `hermes-agent/` |
| **Environment variables** | `PYTHONPATH=hermes-agent/src` |

> **Point d'entrée** : `hermes_cli/main.py`'s `main()` est le point d'entrée CLI unifié. Vous pouvez aussi déboguer `run_agent.py:main()` (mode kernel agent autonome) ou `acp_adapter/entry.py:main()` (mode adaptateur ACP). Voir [pyproject.toml](https://github.com/nousresearch/hermes-agent/blob/main/pyproject.toml) lignes 99–102.

### 4. Ajouter des points d'arrêt

| Fichier | Emplacement | Objectif |
|---------|-------------|----------|
| `run_agent.py` | `AIAgent.run()` L433 | Entrée de la boucle principale |
| `run_agent.py` | `run_conversation()` | Logique d'exécution du Turn |
| `run_agent.py` | `AIAgent.__call__()` | Interface externe |
| `cli.py` | `HermesCLI.cmd_chat()` | Gestion des commandes TUI |
| `model_tools.py` | `_discover_tools()` | Découverte des outils |
| `hermes_state.py` | Méthodes `SessionDB` | Persistance de l'état |

### 5. Démarrer le débogage

Cliquez sur Debug (icône de coléoptère vert) dans PyCharm, ou clic droit → "Debug 'main'".

### Problèmes courants

- **Points d'arrêt inactifs** : Assurez-vous que `venv` est l'interpréteur unique du projet, pas le Python global système.
- **Module introuvable** (`ModuleNotFoundError`) : Vérifiez que `PYTHONPATH` contient `hermes-agent/src`.
- **Problèmes de chemin Windows** : Utilisez des anti-slashs ou `pathlib.Path`, évitez les chemins Unix codés.
- **Arrêt dans venv** : Marquez le dossier `venv` comme Excluded dans PyCharm.

---

## Ressources

- **Dépôt Officiel** : https://github.com/nousresearch/hermes-agent
- **Site Officiel** : https://hermes-agent.nousresearch.com
- **Documentation de démarrage** : https://hermes-agent.nousresearch.com/docs/getting-started/quickstart

<img src="hello-hermes.png" alt="hello-hermes" style="height:800px; display: block; margin-left: 0;" />
