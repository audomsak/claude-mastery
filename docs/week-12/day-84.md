---
title: "Day 84: Capstone Build — RAG Core"
description: "Implement RAG pipeline: ingest, index, retrieve, rerank"
---

# Day 84: Build — RAG Core ⚙️

<div class="lesson-meta">
⏱️ 5 ชั่วโมง &nbsp;|&nbsp; 📊 Project &nbsp;|&nbsp; 📋 Prerequisites: Day 82-83
</div>

## 🎯 Goal

Build RAG service:
- Ingest pipeline
- Hybrid retrieval
- Re-ranking
- Citation tracking

---

## 1. Project Structure

```
capstone/
├── docs/                      # design docs (Day 82-83)
├── ingest/
│   ├── connectors/            # Confluence, GitHub, Drive
│   ├── chunker.py
│   ├── embedder.py
│   └── indexer.py
├── retrieve/
│   ├── vector.py
│   ├── bm25.py
│   ├── reranker.py
│   └── hybrid.py
├── api/
│   └── main.py
├── tests/
├── infra/                     # Terraform (Day 87)
├── docker-compose.yml
├── requirements.txt
└── README.md
```

---

## 2. Connector Interface

```python
# ingest/connectors/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Document:
    id: str
    title: str
    content: str
    metadata: dict
    source: str
    updated_at: datetime

class Connector(ABC):
    @abstractmethod
    def list_documents(self, since: datetime = None) -> list[Document]:
        pass
```

```python
# ingest/connectors/confluence.py
from atlassian import Confluence

class ConfluenceConnector(Connector):
    def __init__(self, url, user, token, space):
        self.client = Confluence(url=url, username=user, password=token)
        self.space = space
    
    def list_documents(self, since=None):
        pages = self.client.get_all_pages_from_space(self.space, limit=500)
        docs = []
        for p in pages:
            content = self.client.get_page_by_id(p["id"], expand="body.storage")
            html = content["body"]["storage"]["value"]
            text = html_to_text(html)
            docs.append(Document(
                id=p["id"],
                title=p["title"],
                content=text,
                metadata={"space": self.space, "url": ...},
                source="confluence",
                updated_at=...
            ))
        return docs
```

---

## 3. Chunker

```python
# ingest/chunker.py
from langchain.text_splitter import RecursiveCharacterTextSplitter

class Chunker:
    def __init__(self, chunk_size=500, overlap=50):
        self.splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=overlap,
            separators=["\n\n", "\n", ". ", " "]
        )
    
    def chunk(self, doc: Document) -> list[dict]:
        chunks = self.splitter.split_text(doc.content)
        return [{
            "id": f"{doc.id}-{i}",
            "text": chunk,
            "doc_id": doc.id,
            "doc_title": doc.title,
            "doc_source": doc.source,
            "doc_url": doc.metadata.get("url"),
            "chunk_idx": i
        } for i, chunk in enumerate(chunks)]
```

---

## 4. Embedder

```python
# ingest/embedder.py
import cohere

class Embedder:
    def __init__(self):
        self.co = cohere.Client()
    
    def embed_batch(self, texts: list[str], input_type: str = "search_document"):
        resp = self.co.embed(
            texts=texts,
            model="embed-english-v3.0",
            input_type=input_type
        )
        return resp.embeddings
```

---

## 5. Indexer (Qdrant)

```python
# ingest/indexer.py
from qdrant_client import QdrantClient, models

class Indexer:
    def __init__(self, url="http://localhost:6333"):
        self.client = QdrantClient(url=url)
    
    def setup_collection(self, name, dim=1024):
        self.client.recreate_collection(
            collection_name=name,
            vectors_config=models.VectorParams(size=dim, distance=models.Distance.COSINE)
        )
    
    def index_batch(self, name, chunks_with_embeddings):
        points = [
            models.PointStruct(
                id=hash(c["id"]) % (2**63),
                vector=c["embedding"],
                payload={k: v for k, v in c.items() if k != "embedding"}
            )
            for c in chunks_with_embeddings
        ]
        self.client.upsert(collection_name=name, points=points)
```

---

## 6. Ingest Pipeline

```python
# ingest/run.py
def run_ingestion(connector_name="confluence"):
    connector = get_connector(connector_name)
    chunker = Chunker()
    embedder = Embedder()
    indexer = Indexer()
    indexer.setup_collection("docs")
    
    docs = connector.list_documents()
    print(f"Got {len(docs)} documents")
    
    all_chunks = []
    for doc in docs:
        chunks = chunker.chunk(doc)
        all_chunks.extend(chunks)
    
    print(f"Total chunks: {len(all_chunks)}")
    
    # Batch embed
    for i in range(0, len(all_chunks), 96):  # Cohere batch size
        batch = all_chunks[i:i+96]
        embeddings = embedder.embed_batch([c["text"] for c in batch])
        for c, e in zip(batch, embeddings):
            c["embedding"] = e
        indexer.index_batch("docs", batch)
        print(f"Indexed {i+len(batch)}/{len(all_chunks)}")
```

---

## 7. Hybrid Retrieval

```python
# retrieve/hybrid.py
from rank_bm25 import BM25Okapi

class HybridRetriever:
    def __init__(self, qdrant_client, all_chunks):
        self.q = qdrant_client
        self.bm25 = BM25Okapi([c["text"].split() for c in all_chunks])
        self.chunks = all_chunks
        self.embedder = Embedder()
        self.reranker = cohere.Client()
    
    def retrieve(self, query, top_k=10):
        # 1. Vector
        q_emb = self.embedder.embed_batch([query], input_type="search_query")[0]
        vec_hits = self.q.search("docs", q_emb, limit=top_k)
        vec_results = [{"text": h.payload["text"], "metadata": h.payload, "score": h.score, "source": "vector"} for h in vec_hits]
        
        # 2. BM25
        bm_scores = self.bm25.get_scores(query.split())
        top_bm = sorted(enumerate(bm_scores), key=lambda x: x[1], reverse=True)[:top_k]
        bm_results = [{"text": self.chunks[i]["text"], "metadata": self.chunks[i], "score": s, "source": "bm25"} for i, s in top_bm]
        
        # 3. Fuse with RRF
        fused = self.rrf([vec_results, bm_results])[:20]
        
        # 4. Re-rank
        rerank_input = [r["text"] for r in fused]
        reranked = self.reranker.rerank(query=query, documents=rerank_input, top_n=5, model="rerank-v3.5")
        
        return [fused[r.index] for r in reranked.results]
    
    @staticmethod
    def rrf(rank_lists, k=60):
        scores = {}
        items = {}
        for ranking in rank_lists:
            for rank, item in enumerate(ranking):
                key = item["text"][:100]
                scores[key] = scores.get(key, 0) + 1 / (k + rank)
                items[key] = item
        return sorted(items.values(), key=lambda i: scores[i["text"][:100]], reverse=True)
```

---

## 8. Citation Tracker

```python
# retrieve/citation.py
def format_for_llm(retrieved):
    """Add numbered citations"""
    parts = []
    citations = []
    for i, r in enumerate(retrieved, 1):
        m = r["metadata"]
        parts.append(f"[{i}] {r['text']}")
        citations.append({
            "id": i,
            "title": m.get("doc_title"),
            "source": m.get("doc_source"),
            "url": m.get("doc_url")
        })
    return "\n\n".join(parts), citations
```

---

## 9. Tests

```python
# tests/test_retrieval.py
def test_hybrid_finds_keyword_query():
    """BM25 should help with exact keyword 'PRD-2024-Q3'"""
    results = retriever.retrieve("PRD-2024-Q3 status")
    assert any("PRD-2024-Q3" in r["text"] for r in results)

def test_hybrid_finds_semantic():
    """Vector should help with paraphrase"""
    results = retriever.retrieve("vacation policy")  # KB uses "PTO policy"
    assert any("PTO" in r["text"] for r in results)

def test_eval_set_accuracy():
    test_set = load_eval_set("eval/retrieval_tests.json")
    correct = 0
    for case in test_set:
        results = retriever.retrieve(case["query"])
        if any(case["expected_doc_id"] == r["metadata"]["doc_id"] for r in results):
            correct += 1
    assert correct / len(test_set) >= 0.80
```

---

## 🛠️ Day 84 Deliverables

- [ ] Connector for at least 1 data source
- [ ] Ingest pipeline (chunk → embed → index)
- [ ] Hybrid retriever
- [ ] Tests with accuracy ≥ 75%
- [ ] 50+ documents indexed
- [ ] CLI: `python ingest/run.py` works end-to-end

[ต่อไป → Day 85 :material-arrow-right:](day-85.md){ .md-button .md-button--primary }
