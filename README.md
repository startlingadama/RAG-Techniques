# **RAG** (Retrieval-Augmented Generation) - Techniques d’encodage

## Comparatif des méthodes d’encodage pour RAG

| Type d’encoder                 | Principe                                                                  | Avantages                                                                                                       | Inconvénients                                                                         | Cas d’usage                                     |
| ------------------------------ | ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Bi-encoder**                 | Encode séparément query + docs dans le même espace vectoriel              |  Très rapide, scalable (index vectordb)<br> Embeddings calculés une seule fois<br> Simple à mettre en place |  Moins précis (ne capte pas les interactions fines)                                  | Retrieval initial sur gros corpus               |
| **Cross-encoder**              | Encode query + doc ensemble et produit un score direct                    |  Très précis<br> Comprend l’interaction mot à mot                                                            |  Lent, pas scalable (il faut scorer chaque doc à la volée)                           | Re-ranking d’un top-k (ex. 50 → 5)              |
| **Poly-encoder**               | Docs encodés une fois, query fait attention aux docs via vecteurs “codes” |  Compromis vitesse/précision<br> Plus efficace que cross-encoder                                             |  Plus complexe à entraîner<br>Moins précis que cross-encoder                         | Chatbots, retrieval conversationnel             |
| **ColBERT (late interaction)** | Encode tokens séparés, matching query–doc token-level                     |  Très précis<br> Plus rapide que cross-encoder                                                                |  Plus lourd qu’un simple bi-encoder<br>Besoin de mémoire (token-level)               | Dense passage retrieval (DPR), recherche fine   |
| **Hybrid (BM25 + dense)**      | Combine recherche lexicale (mots-clés) + dense embeddings                 |  Robustesse sur noms propres, chiffres, entités<br> Bonne couverture                                         |  Mise en œuvre plus complexe (fusion des scores)<br>Moins efficace sur sens abstrait | Recherche juridique, médicale, bases techniques |
| **Hierarchical**               | Encode morceaux (paragraphe, section) puis leurs résumés                  |  Adapté aux documents longs (PDF, rapports)<br> Multi-niveaux de granularité                                 |  Plus complexe<br>Besoin de plusieurs passes                                         | RAG sur PDF, livres, rapports longs             |

---

En pratique, le **pipeline optimal** est souvent :

* **Bi-encoder dense retrieval** → rapide.
* **Hybrid lexical + dense** → robustesse.
* **Cross-encoder re-ranker** → précision finale.