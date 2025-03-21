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
   - Utilisation du modèle 🇫🇷 [`dangvantuan/sentence-camembert-large`](https://huggingface.co/dangvantuan/sentence-camembert-large)
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
2. Bash : cd Desktop/dgfip-chatbot-local
3. Bash : pip install -r requirements.txt
4. Bash : python chatbot_dgfip_hierarchique.py
5. Bash : Posez votre question

---

### TECHNIQUE.md

Ce document décrit en détail le fonctionnement du moteur de recherche sémantique du chatbot DGFiP. Il présente :

- la logique de prétraitement des fiches et de filtrage thématique,
- l'utilisation du modèle SBERT pour l'encodage sémantique,
- le calcul de similarité cosinus pour le classement des réponses,
- des suggestions concrètes d'amélioration pour les data scientists,
- et des pistes sérieuses de mise en production pour un usage en administration publique.

---
