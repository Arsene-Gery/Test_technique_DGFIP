# Chatbot DGFiP — Recherche de fiches fiscales

---

Ce cours projet propose un mini-chatbot capable d’associer une question d’usager à la fiche fiscale la plus pertinente parmi celles publiées sur le site officiel [impots.gouv.fr](https://www.impots.gouv.fr).  
Il repose sur des techniques de traitement automatique du langage (NLP) pour enrichir l’accès à l’information administrative en français.

---

### Objectif

Simuler un assistant intelligent capable de répondre à des interrogations d’usagers concernant :
- les déclarations de revenus,
- les situations familiales,
- les revenus fonciers, réductions d’impôts, etc.

Le système agit comme une **FAQ intelligente**, reposant sur un moteur sémantique et une indexation hiérarchique des thèmes fiscaux.

---

### Fonctionnement de l'algorithme

### Données
- **113 fiches fiscales** extraites de l’espace particulier du site des impôts (`info_particulier_impot.csv`)
- **Questions synthétiques** d’usagers avec la fiche attendue (`questions_fiches_fip.csv`)

### Étapes de traitement

1. **Prétraitement** :
   - Nettoyage et troncature des textes
   - Concaténation `Titre : Texte` pour un encodage plus contextuel

2. **Indexation hiérarchique par thème** :
   - Chaque fiche est indexée par des mots-clés issus de son titre
   - La question d’un usager est analysée pour filtrer les fiches par mots-clés
   - Fallback automatique sur toutes les fiches si aucun mot-clé ne matche

3. **Encodage sémantique (SBERT)** :
   - Utilisation du modèle [`dangvantuan/sentence-camembert-large`](https://huggingface.co/dangvantuan/sentence-camembert-large)
   - Chaque fiche est transformée en vecteur dense

4. **Recherche par similarité cosinus** :
   - La question est encodée et comparée aux fiches filtrées
   - La fiche avec la similarité la plus élevée est renvoyée

---

### Technologies utilisées

| Technologie             | Usage |
|-------------------------|-------|
| `pandas`                | Manipulation des jeux de données |
| `scikit-learn`          | Calcul de similarité cosinus |
| `sentence-transformers` | Encodage SBERT des phrases |
| `Python` (standard)     | Aucune base de données requise |

---

### **Exécution locale (zéro infrastructure)** :

1. Téléchargez le .zip dgfip-chatbot-local dans votre Desktop
2. Téléchargez les datasets "info_particulier_impot.csv" et "questions_fiches_fip.csv"
3. Changer le chemin absolu des datasets dans le notebook "chatbot_dgfip_hierarchique.ipynb" : FICHES_PATH = "..." ; QUESTIONS_PATH = "..."
4. Bash : pip install pandas numpy scikit-learn sentence-transformers
5. Bash : cd Desktop/dgfip-chatbot-local
6. Bash : pip install -r requirements.txt
7. Bash : python chatbot_dgfip_hierarchique.py
8. Bash : Posez votre question

---

### TECHNIQUE.md

Ce document décrit en détail le fonctionnement du moteur de recherche sémantique du chatbot DGFiP :

- la logique de prétraitement des fiches et de filtrage thématique,
- l'utilisation du modèle SBERT pour l'encodage sémantique,
- le calcul de similarité cosinus pour le classement des réponses,
- des suggestions concrètes d'amélioration pour les data scientists,
- et des pistes sérieuses de mise en production pour un usage en administration publique.

---

### High_Performance_Model.ipynb

Ce document renferme le chatbot le plus performant, pouvant être déployé dans l'immédiat :

- **Indexation complète du texte** :  
  Chaque mot du texte complet (titre + contenu) est indexé, ce qui augmente fortement la probabilité de trouver une fiche pertinente.
- **Recherche hybride (mots-clés + SBERT)** :  
  Le moteur combine un filtrage rapide par mots-clés avec un classement sémantique profond via SBERT pour des résultats précis et rapides.
- **Filtrage initial rapide** :  
  L’index inversé réduit le nombre de fiches à comparer, accélérant significativement la recherche tout en conservant les fiches pertinentes.
- **Classement sémantique précis** :  
  SBERT permet de classer les fiches selon leur proximité sémantique avec la question, même sans mots exacts en commun.
- **Pas de seuil de mots-clés strict** :  
  Tout document contenant au moins un mot commun avec la question est analysé, ce qui augmente le rappel (recall) global.
- **Retour des N meilleurs résultats** :  
  Le moteur retourne plusieurs fiches pertinentes (Top-N), offrant plus d’options si la première réponse ne convient pas.
- **Extraits de contexte (snippets)** :  
  Chaque fiche retournée inclut un passage contenant des mots-clés de la question pour juger rapidement de sa pertinence.
- **Seuil de similarité avec message d’incertitude** :  
  Si la similarité est trop faible, un message indique à l’utilisateur que la réponse est incertaine, renforçant la transparence.
- **Normalisation linguistique (stemming)** :  
  Les mots sont réduits à leur racine (ex. : "imposition" → "impos"), ce qui améliore la correspondance sémantique.
- **Évaluation quantitative (Accuracy / MRR)** :  
  La fonction d’évaluation mesure la qualité du moteur via Accuracy@1, @5, et MRR sur un corpus annoté.
- **Code clair, francisé et commenté** :  
  Variables et fonctions en français avec commentaires explicites rendent le code accessible, modifiable et maintenable.
- **Gestion des cas sans résultats** :  
  Si aucun résultat n’est pertinent, une réponse vide est retournée, évitant les suggestions incorrectes.

---
