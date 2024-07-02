---
id: nh-075
title: 'Challenges with Semantic Searches and a potential Solution'
date: 2024-07-02 01:01:00 +0100
author: 'Niklas Heidloff'
layout: post
guid: 'heidloff.net/nh-075'
permalink: /article/reverse-semantic-searches/
custom_permalink:
    - article/reverse-semantic-search/
image: /assets/img/2024/07/semantic-alternatives.png
---

*For Generative AI scenarios like Retrieval Augmented Generation many patterns suggest using semantic searches to get best search results. For large datasets several search engines struggle since a lot of resources are required. This post describes an idea how to fix these issues.*

For more context read my previous posts:

* [Semantic Searches with Elasticsearch]({{ "/article/semantic-search-vector-eslasticsearch/" | relative_url }})
* [Introduction to Neural Information Retrieval]({{ "/article/introduction-neural-information-retrieval/" | relative_url }})
* [Running Generative AI Experiments for Question Answering]({{ "/article/running-generative-ai-experiments-for-question-answering/" | relative_url }})

## Challenges

In general, semantic searches require a lot of resources and are expensive. Especially when dealing with millions or even billions of documents a lot of resources are needed, much more than for classic full text searches (BM25). There are several challenges.

* For the creation of the embeddings additional compute is required, either via many CPUs or GPUs if the embedding models support GPUs.
* The size of the documents is much bigger since embeddings are often stored as vectors which are essentially long lists of long numbers which require more space than only the original text.
* Semantic searches also often require splitting of documents and additional indexing functionality, for example via pipelines, which increases the complexity of the deployment.
* Finding the best embedding model can be work intensive, since it often requires experimentation.
* When separate clusters are utilized for the indexing, the data needs to be transformed and often merged with the main index on the production cluster.
* To achieve a good runtime performance with short response times, in the best case all data should be stored in memory which can become very expensive or even impossible.

## Solution

With all these challenges, you might be curious to know how to fix semantic searches with embeddings, vector databases, etc. Here is one potential concept, which could be called `Reverse Semantic Search`.

* Do NOT use semantic searches with embeddings! 
* Switch back to classic (BM25 searches).
* Generate alternatives of the input text via a LLM which are semantically similar.
* Invoke a BM25 search with the original text and the alternatives.

While the generation of the alternatives requires an additional large language model, the efficiency of classic search systems is leveraged. Compared to semantic searches with embeddings this gives you a huge saving of resources.

Here is how an instruction and sample text could look like.

```text
You are a full text search expert that helps users to optimize search results.
1. Rephrase the provided TEXT while maintaining the original semantic meaning.
2. Do not add any new information or alter the intended content.
3. The rephrased text should be clear, precise, and professional and have the same length.
4. Use synonyms whenever possible.
5. Create three alternatives.

TEXT:
A foundation model, also known as large AI model, is a machine learning or deep learning model that is trained on broad data such that it can be applied across a wide range of use cases. Foundation models have transformed artificial intelligence (AI), powering prominent generative AI applications like ChatGPT. 

ALTERNATIVE 1:
```

Here is the generated result from Mixtral. To improve the results, the prompt can be further optimized and/or a temperature higher than '0' can be tried.

```text
A base model, also termed extensive AI model, refers to a machine learning or deep learning model that undergoes training on extensive data, making it adaptable for diverse applications. Base models have revolutionized artificial intelligence (AI), fueling popular generative AI systems, such as ChatGPT.

ALTERNATIVE 2:
A fundamental model, also identified as expansive AI model, is a machine learning or deep learning model that receives training on extensive data, enabling it to be utilized in a multitude of scenarios. Fundamental models have significantly advanced artificial intelligence (AI), driving prominent generative AI systems, like ChatGPT.

ALTERNATIVE 3:
A base AI model, also known as a comprehensive model, is a machine learning or deep learning model that is trained on extensive data, making it applicable across various use cases. Base AI models have revolutionized artificial intelligence (AI), empowering popular generative AI applications, such as ChatGPT.
```

## Rerankers

There is another option to use Rerankers on the classic search results, see the links above. Rerankers find semantically similar documents, but typically don't require any persistence and work on a smaller set of documents which means less resources. Since they require another compute step, which means typically another GPU, experiments need to be run to compare whether the better performance justifies the additional compute and the longer duration.