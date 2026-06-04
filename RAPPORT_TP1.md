# TP1 : Installation et configuration IA locale \- Rapport

## Informations

\- Nom : Adam K. ; Nayir S.  
\- Date : 2026-06-02  
\- Durée réelle : environ 4 heures.

## 1\. Installation Ollama

J'ai installé Ollama sur mon serveur via la commande "irm [https://ollama.com/install.ps1](https://ollama.com/install.ps1) | iex" (commande présent sur le site officiel Ollama). Une fois installé, la commande “`curl http://localhost:11434”` renvoie bien le message “`Ollama is running`.”

### Problèmes rencontrés sur la partie Ollama

Plusieurs tags utilisés dans le sujet du tp ne sont plus disponibles sur ollama. Par exemple, les tags “`llama3.2:3b”`,  “`llama3.2:3b-q4_K_M”` et “`mistral:7b”` échouent tous avec le message d’erreur “`model not found”`.

La commande “`ollama run llama3.2:3b --temperature 0.7 --top-p 0.9`” renvoie aussi le message suivant : “`Error: unknown flag: --temperature`”

Pour régler la température ou le top-p il faut passer par l'API REST ou un Modelfile. 

### 3\. Benchmarks

Prompt commun envoyé aux trois modèles :

“Ecris une fonction Python qui prend une liste de nombres et retourne un dictionnaire contenant la somme, la moyenne, le minimum et le maximum. Ajoute des docstrings et des type hints.”

Résultats :

\[

  {"model": "llama3.2:latest", "wall\_seconds": 3.22, "eval\_count": 384, "tokens\_per\_second": 270.12},

  {"model": "codellama:7b",   "wall\_seconds": 4.21, "eval\_count": 407, "tokens\_per\_second": 171.25},

  {"model": "gemma4:latest",  "wall\_seconds": 10.37, "eval\_count": 600, "tokens\_per\_second": 91.51}

\]

`llama3.2:latest` est le plus rapide à environ 270 tokens/s et donne une fonction correcte avec docstring mais ses type hints sont trop génériques, juste `list` au lieu de `list[int | float]`. `codellama:7b` est correct aussi mais plus lent à 171 tokens/s, avec les mêmes type hints vagues et un retour `{}` étrange sur liste vide. `gemma4:latest` a posé un vrai problème : malgré un `eval_count` de 600 et un `done_reason` à `length`, la réponse finale était vide. Le modèle n’as rien produit d'exploitable, donc je ne le retiens pas pour cette tâche.

### Échantillon de code généré

Prompt envoyé à `codellama:7b` :

“Ecris une fonction Python simple qui valide une adresse email avec regex, type hints et docstring.”

Réponse générée :

“  
import re

def validate\_email(email: str) \-\> bool:

    """Validate an email address using a regular expression."""

    pattern \= r"^\[a-zA-Z0-9.\_%+-\]+@\[a-zA-Z0-9.-\]+\\.\[a-zA-Z\]{2,}$"

    return re.search(pattern, email) is not None  
“  
Le code marche sur les cas simples, mais `re.search` n'ancre pas vraiment la chaîne entière, donc `fullmatch` serait plus sûr, et il n'y a aucune gestion de `None` ni de chaîne vide. 

Tests API REST :  
`“curl http://localhost:11434”` renvoie “`Ollama is running”`, et `GET /api/tags` répond en HTTP 200 avec la liste des modèles. Le point intéressant est `/api/generate` sur `llama3.2:latest`, qui répond en 200 mais hallucine quand je lui pose la question : “Qu’est-ce que Ollama ?”.

Il répond :  
“Je n'ai pas d'informations sur un systeme appele "Ollama"...”  
ou encore  
“Ollama é uma tecnologia de reconhecimento de vozes…”

Les deux réponses sont fausses et la deuxième réponse qu’a fait le modèle n'est même pas en Français. “`/api/chat”` répond aussi en 200 et sort un exemple Python correct mais avec des variables mal nommées. Les routes marchent, le contenu doit être relu.

### 4\. Analyse comparative

Pour la vitesse je garde `llama3.2:latest` qui est bien pour les questions rapides, les reformulations et les brouillons. Pour le code `codellama:7b` reste bien même s'il ne creuse pas un écart sur un test simple. J’ai également testé les gros modèles Qwen, notamment qwen3.5:32b sur des tâches plus complexes et il s’en sort étonnamment très bien.

## 

## Partie 2 \- Réflexion éthique et RGPD

### Confidentialité

Passer par Ollama en local réduit fortement le risque de fuite, parce que les prompts vont vers le serveur local sur le port 11434 au lieu d'une API cloud. C'est un vrai plus pour du code propriétaire. Mais local ne veut pas dire hors RGPD. Coller des emails, noms ou données clients dans un prompt reste un traitement de données personnelles. J'applique donc la minimisation, je n'utilise pas de données réelles, et je fais attention aux historiques stockés sur la machine.

### Propriété intellectuelle

Un modèle open source peut produire du code proche d'un projet existant sous licence. Il faut vérifier la licence du modèle, par exemple Llama suit une licence communautaire Meta qui encadre certains usages. Je relis le code, je garde mes prompts pour la traçabilité, et je ne livre pas du code que je ne comprends pas.

### Biais

Les modèles peuvent sortir des exemples stéréotypés, des variables ou commentaires discutables, ou de vieilles pratiques. La responsabilité reste humaine, dire que c'est l'IA qui a généré ne suffit pas. Je relis les exemples, je teste les cas limites et je corrige ce qui pose problème.

## Transparence

Si une entreprise impose d’utiliser Continue.dev, les développeurs doivent savoir ce que l'outil fait comme l’analyse, le stocke et l’envoie, même en local. Une charte d'usage, une revue humaine obligatoire et la conservation des prompts importants pour permettre l'audit et la conformité.

## Comparaison local et cloud

L'IA locale protège mieux les données et fonctionne hors ligne après l’avoir téléchargé. Il n’y a pas de coût par requête mais sa qualité dépend du modèle et la traçabilité est à organiser par soi-même. Le cloud comme ChatGPT ou Claude donne souvent de meilleures réponses mais envoie les données à un tiers ce qui complique la RGPD avec le transfert de données et la sous-traitance. Dans les deux cas, la validation humaine reste obligatoire.

## Conclusion

L'IA locale a de vrais avantages comme la confidentialité, le contrôle des modèles, aucune dépendance au cloud et un accès direct à l'API REST. Ollama tourne bien sur cette machine et utilise le GPU. Mais il faut être conscient de plusieurs points comme les modèles qui peuvent halluciner même en local. Le point à retenir est que local ne veut pas dire fiable, la vérification humaine reste indispensable.  
