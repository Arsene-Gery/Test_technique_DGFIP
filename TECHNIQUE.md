# Chatbot DGFiP - Fonctionnement technique du moteur de recherche

Ce document détaille pas à pas les mécanismes internes du chatbot, afin de permettre à tout data scientist de comprendre le **pipeline algorithmique**, les choix techniques, et la logique d'exécution.

---

## Étape 1 — Chargement et préparation des données

### Données utilisées :
- `info_particulier_impot.csv` : 113 fiches fiscales avec colonnes `Titre`, `Texte`, `URL`
- Seule cette base est utilisée dans cette version console du chatbot

### Objectif :
Préparer les fiches en concaténant le **titre et le contenu**, puis tronquer les textes pour éviter les dépassements de séquence (limite ≈ 512 tokens pour SBERT).

```python
df_fiches["Texte_Concat"] = df_fiches["Titre"] + " : " + df_fiches["Texte"]
df_fiches["Texte_Concat"] = df_fiches["Texte_Concat"].apply(lambda x: x[:1000])
```

---

## Étape 2 — Filtrage hiérarchique par mots-clés

Avant tout encodage ou calcul de similarité, on filtre les fiches **en amont** à partir des mots-clés présents dans la question.

### Objectif :
Réduire la taille du sous-espace de recherche (≈ 10-30 fiches pertinentes), pour :
- améliorer la rapidité
- réduire les collisions sémantiques
- favoriser les correspondances contextuelles

```python
def normaliser_texte(texte):
    texte = texte.lower()
    texte = re.sub(r"[^a-zàâçéèêëîïôûùüÿœ\s-]", "", texte)
    return texte
```

### Construction d’un index thématique :
Chaque mot du titre de chaque fiche est stocké dans un `defaultdict(set)` qui sert d'index.

```python
index_theme[mot].add(index_fiche)
```

---

## Étape 3 — Encodage SBERT

On utilise Sentence-BERT pour transformer :
- chaque `Texte_Concat` de fiche en vecteur denses de dimension 768
- chaque question en vecteur unique, calculé à la volée

```python
from sentence_transformers import SentenceTransformer

modele = SentenceTransformer("dangvantuan/sentence-camembert-large")
fiches_embeddings = modele.encode(fiches_textes, convert_to_tensor=True).cpu().numpy()
```

Ce modèle est **pré-entraîné sur des tâches de similarité sémantique en français**.

---

## Étape 4 — Similarité cosinus

On compare les vecteurs de la question et des fiches filtrées à l’aide de la similarité cosinus, une mesure d’angle entre vecteurs dans un espace vectoriel.

```python
from sklearn.metrics.pairwise import cosine_similarity
similarites = cosine_similarity(embedding_question, sous_ensemble_embeddings).flatten()
```

### Pourquoi la similarité cosinus ?
- Robuste aux différences de longueur
- Insensible aux valeurs absolues (on mesure la direction sémantique)
- Compatible avec les embeddings SBERT

---

## Étape 5 — Sélection de la fiche la plus pertinente

Le chatbot renvoie la fiche ayant le **score de similarité maximal** parmi les fiches candidates :

```python
index_max = np.argmax(similarites)
fiche = fiches[index_max]
```

Sont retournés :
- `Titre`
- `URL`
- `Score de similarité`
- `Nombre de fiches analysées`

---

## Résumé du pipeline complet

```
1. Question posée par l’usager
2. → Normalisation et extraction des mots-clés
3. → Filtrage des fiches par index thématique
4. → Encodage SBERT de la question
5. → Similarité cosinus avec les fiches filtrées
6. → Sélection de la fiche avec score max
7. → Affichage du résultat
```

---

## Pistes d'amélioration de l'algorithme

---

Ce moteur est optimisé pour la **vitesse, la transparence et l'interprétabilité**, ce qui le rend idéal pour des environnements institutionnels ou semi-structurés.

- Le seuil de troncature (par défaut 1000 caractères)
- Le modèle SBERT (autre modèle HuggingFace compatible sentence-transformers)
- La stratégie de filtrage (lemmatisation, TF-IDF préalable, etc.)
- L’ajout de métadonnées (tags thématiques explicites)

---

### Optimisation des performances et du comportement
- **Normalisation linguistique approfondie** : intégration de `spaCy` ou `Stanza` pour la lemmatisation et la désambigüisation morphologique.
- **Troncature conditionnelle par tokens** : utiliser le tokenizer du modèle (CamemBERT) pour tronquer les textes selon la limite exacte de 512 tokens.
- **Fusion pondérée Titre + Texte** : renforcer l'importance du titre en le répétant ou en le pondérant lors de la concaténation.
- **Combinaison de scores** : addition ou pondération entre similarité cosinus et des scores lexicaux simples (Jaccard, correspondance exacte dans le titre).

### Amélioration du modèle sémantique
- **Fine-tuning SBERT** avec supervision partielle sur un jeu de données fiscales labellisé.
- **Utilisation de modèles multilingues** pour élargir le périmètre du chatbot (LaBSE, paraphrase-multilingual-mpnet).

### Structuration avancée du corpus
- **Clustering thématique non supervisé** avec `UMAP` + `HDBSCAN` pour créer des familles de fiches.
- **Construction d’un graphe de similarité entre fiches** pour naviguer dynamiquement dans les contenus proches.
- **Ajout de méta-tags** manuels ou automatisés (LDA, keyBERT) pour un meilleur filtrage.

### Vers une logique conversationnelle
- Stockage de l’historique de la session (ex : nom du thème sélectionné précédemment).
- Gestion des incertitudes : suggestions alternatives si score faible.
- Réponses enrichies avec top-3 + bouton de feedback utilisateur.

---

## Pistes de mise en production pour usage institutionnel (DGFiP / Ministère des Finances)

### A. Intégration dans un intranet ou application interne
- Interface web simple (Streamlit / Flask) hébergée sur un serveur Linux existant
- Sécurisation via reverse proxy NGINX + certificat SSL
- Connexion à l’annuaire LDAP ou à un système d’authentification interne

### B. Déploiement sous forme d’API REST
- Déploiement sous Docker avec FastAPI
- Serveur de production avec `gunicorn` ou `uvicorn` supervisé (ex : systemd)
- Monitoring via Prometheus + Grafana, logging journalisé

### C. Hébergement cloud pour démonstration
- Plateformes simples : Render, Railway, OVHcloud
- Version publique en sandbox pour tests internes ou retours utilisateurs
- Protection par mot de passe ou proxy SSH inversé

### D. Intégration à un SI existant
- API embarquée dans un outil métier (NOEMIE, Espace Agent, portail RH ou fiscal)
- Intégration à un moteur documentaire (ex : ElasticSearch, Solr) pour croisement de sources
- Exploitation des logs anonymisés pour améliorer continuellement la base de connaissances

---