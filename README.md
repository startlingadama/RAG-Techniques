# **RAG** (Retrieval-Augmented Generation) - Techniques d’encodage

## **Stratégies de chunking** pour RAG :

---

| **Stratégie**             | **Description**                                                                 | **Avantages**                                             | **Inconvénients**                                   | **Cas d’usage recommandé**                                             |
| ------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------- |
| **Fixed-size chunking**   | Découpe en blocs de *n tokens* (ex. 512). Possibilité d’overlap.                | Simple, rapide, compatible avec tous les modèles.           | Coupe les phrases/paragraphes → perte de sens.        | Docs techniques, manuels longs, quand la cohérence locale importe peu. |
| **Sliding window**        | Variante du fixed-size avec chevauchement (ex. 512 tokens avec 50–100 overlap). | Évite de couper brutalement une idée. Améliore rappel.      | Double parfois la taille de la base (plus de chunks). | FAQ, dialogues, contextes narratifs.                                   |
| **Paragraph-based**       | Chaque paragraphe devient un chunk.                                             | Naturel, conserve le sens. Facile à interpréter.            | Taille inégale : trop court ou trop long.             | Articles, rapports, documentation avec mise en page claire.            |
| **Sentence-based**        | Chaque phrase (ou 2–3 phrases regroupées) = chunk.                              | Très précis, idéal pour répondre à une question spécifique. | Beaucoup trop de chunks → coûteux.                    | QA fine-grained, extraction de faits précis.                           |
| **Semantic chunking**     | Détecte frontières logiques via NLP/LLM (changement de thème, sous-section).    | Chunks plus "intelligents" et cohérents.                    | Complexe, parfois lent.                               | Articles académiques, rapports, thèses, docs complexes.                |
| **Hierarchical chunking** | Indexation multi-niveaux : petits chunks + gros chunks.                         | Combine précision (fine) + rappel (coarse).                 | Implémentation plus lourde.                           | Grands PDF (livres, rapports annuels), bases mixtes.                   |
| **Dynamic (LLM-guided)**  | LLM choisit les meilleures coupures (ex : GPT découpe 300–500 tokens).          | Très flexible, chunks sémantiquement optimaux.              | Coûteux, dépend d’un LLM externe.                     | Données hétérogènes, documents mal structurés.                         |
| **Metadata-aware**        | Ajout d’info contextuelle (titre, section, page, auteur) au chunk.              | Améliore fortement la recherche, moins d’ambiguïté.         | Préprocessing plus complexe.                          | Recherche juridique, médicale, réglementaire, catalogues.              |

---

Recommandation pratique :
Pour un **PDF complet** → combiner :

* **Semantic/paragraph chunking** (base).
* * **Fixed-size + overlap** si chunks trop longs.
* * **Metadata-aware** pour améliorer le recall et la précision.



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