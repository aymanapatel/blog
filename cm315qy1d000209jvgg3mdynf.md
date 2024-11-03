---
title: "End to End RAG with Postgres"
seoTitle: "End to End RAG with Postgres"
seoDescription: "And how Timescale is fixing RAG pipelines using its pgai ecosystem"
datePublished: Sun Nov 03 2024 05:34:38 GMT+0000 (Coordinated Universal Time)
cuid: cm315qy1d000209jvgg3mdynf
slug: end-to-end-rag-with-postgres
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1730611911022/a5324e9a-f25c-4a57-a810-b08d1c09e9de.jpeg
tags: ai, postgresql, rag

---

In the world of **Retrieval-Augmented Generation** (RAG) systems that cater to specific internal data, there is a need to store embedding in a Vector Database. As it is the 2nd step after retrieval, it echoes the ETL pipeline in Big Data pipelines.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1730611923961/72bb2cb6-e25a-4126-9db8-d2760613cba2.jpeg align="center")

And similar to ETL workflows, there are issues with out-of-sync data due to batch updates that happen at a cadence (nightly, weekly etc.). Internal data sources such as wikis or documentation are also updated frequently; hence Vector databases need to be updated frequently.

External vector database such as [Pinecone](https://www.pinecone.io/), [Milvus](https://milvus.io/) would need an integration with frameworks lile [LlamaIndex](https://www.llamaindex.ai/) or [Langchain](https://www.langchain.com/). Also updating it would mean testing, validating amongst other tasks.

So the question is

> What if the vector data store could handle it natively?

Enter Postgres.

One of my personal favourite videos on Postgres is from “Art of the Terminal” named “**Wait... PostgreSQL can do WHAT?”**

%[https://www.youtube.com/watch?v=VEWXmdjzIpQ] 

Postgres as an ecosystem has everything you could ask for. A REST API with PGRest, Vector store with Pgvector etc. This is just tip of the iceberg. Hence, this allows Postgres to be a smart data store that can handle updates itself.

Timescale has released an extension `pgai vectorizer`. This extension handles:

1. Data storage (as it is Postgres, so it is non-trivial)
    
2. Insert and Update and embedding in Postgres itself.
    

```sql
SELECT ai.create_vectorizer( 
    <my_table_name>::regclass, 
    destination => <embedding_table_name>,
    -- Embedding model to use like `text-embedding-3-small` with dimension number
    embedding => ai.embedding_openai(<model_name>, <dimensions>),
    -- Chunking strategy
    chunking => ai.chunking_recursive_character_text_splitter(<column_name>)
);
```

3. Use SQL statement (just one!) to implement RAGs.
    

```sql
-- 1. Create a reusable RAG function
CREATE OR REPLACE FUNCTION my_rag_response(query_text TEXT)
RETURNS TEXT AS $$
DECLARE
   context_chunks TEXT;
   response TEXT;
BEGIN
   -- Perform similarity search to find relevant wiki
   SELECT string_agg(title || ': ' || chunk, ' ') INTO context_chunks
   FROM (
       SELECT title, chunk
       FROM wiki_embedding
       ORDER BY embedding <=> ai.openai_embed('text-embedding-3-small', query_text)
       LIMIT 3
   ) AS relevant_posts;

   -- Generate a summary using gpt-4o-mini
   SELECT ai.openai_chat_complete(
       'gpt-4o-mini',
       jsonb_build_array(
           jsonb_build_object('role', 'system', 'content', 'You are a helpful assistant. Use only the context provided to answer the question. Also mention the titles of the blog posts you use to answer the question.'),
           jsonb_build_object('role', 'user', 'content', format('Context: %s\n\nUser Question: %s\n\nAssistant:', context_chunks, query_text))
       )
   )->'choices'->0->'message'->>'content' INTO response;

   RETURN response;
END;
$$ LANGUAGE plpgsql;

-- 2. Execute the RAG function
SELECT my_rag_response('How do I onboard this new service?');
```

4. Use [pgvectorscale](https://github.com/timescale/pgvectorscale) to do ANN which is required for RAG applications.
    

For real demo, Timescale has shared the demo here:

%[https://www.youtube.com/watch?v=ZoC2XYol6Zk] 

# Resources:

1. [Timescale Blog](https://www.timescale.com/blog/vector-databases-are-the-wrong-abstraction/)
    
2. [PgAI Github repository](https://github.com/timescale/pgai)
    

# Closing thoughts

So try it out in your Postgres instance. And closing thoughts is

“Postgres is awesome”