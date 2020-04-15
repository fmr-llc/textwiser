# TextWiser: Text Featurization Library

TextWiser is a research library that provides a unified framework for text featurization based on a rich set of methods
while taking advantage of pretrained models provided by the state-of-the-art 
[Flair](https://github.com/zalandoresearch/flair) library. The main contributions include:

* **Rich Set of Embeddings:** A wide range of available [embeddings](#available-embeddings) and [transformations](#available-transformations)
 to choose from.  

* **Fine-Tuning:** Designed to support a ``PyTorch`` backend, and hence, retains the ability to 
[fine-tune featurizations](#fine-tuning-for-downstream-tasks) for downstream tasks. 
That means, if you pass the resulting fine-tunable embeddings to a training method, the features will 
be optimized automatically for your application. 

* **Parameter Optimization:** Interoperable with the standard ```scikit-learn``` pipeline for hyper-parameter tuning and rapid experimentation. All underlying parameters are exposed to the user.

* **Grammar of Embeddings:** Introduces a novel approach to design embeddings from components. 
The [compound embedding](#compound-embedding) allows forming arbitrarily complex embeddings in accordance with a 
[context-free grammar](#a-context-free-grammar-of-embeddings) that defines a formal language for valid text featurization.

* **GPU Native:** Built with GPUs in mind. If it detects available hardware, the relevant models are automatically placed on the GPU. 
 
<br> 
TextWiser is developed by the Artificial Intelligence Center of Excellence at Fidelity Investments.

## Quick Start

```python
# Conceptually, TextWiser is composed of an Embedding, potentially with a pretrained model,
# that can be chained into zero or more Transformations
from textwiser import TextWiser, Embedding, Transformation, WordOptions, PoolOptions

# Data
documents = ["Some document", "More documents. Including multi-sentence documents."]

# Model: TFIDF `min_df` parameter gets passed to sklearn automatically
emb = TextWiser(Embedding.TfIdf(min_df=1))

# Model: TFIDF followed with an NMF + SVD
emb = TextWiser(Embedding.TfIdf(min_df=1), [Transformation.NMF(n_components=30), Transformation.SVD(n_components=10)])

# Model: Word2Vec with no pretraining that learns from the input data
emb = TextWiser(Embedding.Word(word_option=WordOptions.word2vec, pretrained=None), Transformation.Pool(pool_option=PoolOptions.min))

# Model: BERT with the pretrained bert-base-uncased embedding
emb = TextWiser(Embedding.Word(word_option=WordOptions.bert), Transformation.Pool(pool_option=PoolOptions.first))

# Features
vecs = emb.fit_transform(documents)
```

### Available Embeddings

| Embeddings | Notes |
| :--------------------: | :-----: |
| [Bag of Words (BoW)](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html#sklearn.feature_extraction.text.CountVectorizer) | Supported by ``scikit-learn`` <br> Defaults to training from scratch|
| [Term Frequency Inverse Document Frequency (TfIdf)](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html) | Supported by ``scikit-learn`` <br> Defaults to training from scratch|
| [Document Embeddings (Doc2Vec)](https://radimrehurek.com/gensim/models/doc2vec.html)| Supported by ``gensim`` <br> Defaults to training from scratch |
| [Universal Sentence Encoder (USE)](https://tfhub.dev/google/universal-sentence-encoder-large/5) | Supported by ``tensorflow``, see [requirements](requirements) <br> Defaults to [large v5](https://tfhub.dev/google/universal-sentence-encoder-large/5) |
| [Compound Embedding](#compound-embedding) | Supported by a [context-free grammar](#a-context-free-grammar-of-embeddings)|
| Word Embedding: [Word2Vec](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/CLASSIC_WORD_EMBEDDINGS.md) | Supported by these [pretrained embeddings](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/CLASSIC_WORD_EMBEDDINGS.md) <br> Common pretrained options include ``crawl``, ``glove``, ``extvec``, ``twitter``, and ``en-news`` <br> When the pretrained option is ``None``, trains a new model from the given data <br> Defaults to ``en``, FastText embeddings trained on news |
| Word Embedding: [Character](https://github.com/zalandoresearch/flair/blob/master/resources/docs/TUTORIAL_3_WORD_EMBEDDING.md#character-embeddings)| Initialized randomly and not pretrained <br> Useful when trained for a downstream task <br> Enable [fine-tuning](#fine-tuning-for-downstream-tasks) to get good embeddings |
| Word Embedding: [BytePair](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/BYTE_PAIR_EMBEDDINGS.md) | Supported by these [pretrained embeddings](https://nlp.h-its.org/bpemb/#download>) <br> Pretrained options can be specified with the string ``<lang>_<dim>_<vocab_size>`` <br> Default options can be omitted like ``en``, ``en_100``, or ``en__10000`` <br> Defaults to ``en``, which is equal to ``en_100_10000`` |
| Word Embedding: [ELMo](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/ELMO_EMBEDDINGS.md) | Supported by these [pretrained embeddings](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/ELMO_EMBEDDINGS.md) from [AllenNLP](https://allennlp.org) <br> Defaults to ``original`` |
| Word Embedding: [Flair](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/FLAIR_EMBEDDINGS.md) |  Supported by these [pretrained embeddings](https://github.com/zalandoresearch/flair/blob/master/resources/docs/embeddings/FLAIR_EMBEDDINGS.md) <br> Defaults to ``news-forward-fast`` |
| Word Embedding: [BERT](https://github.com/huggingface/transformers#model-architectures)| Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``bert-base-uncased`` |
| Word Embedding: [OpenAI GPT](https://github.com/huggingface/transformers#model-architectures)| Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``openai-gpt`` |
| Word Embedding: [OpenAI GPT2](https://github.com/huggingface/transformers#model-architectures) |Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``gpt2-medium`` |
| Word Embedding: [TransformerXL](https://github.com/huggingface/transformers#model-architectures) |Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``transfo-xl-wt103`` |
| Word Embedding: [XLNet](https://github.com/huggingface/transformers#model-architectures)|Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``xlnet-large-cased`` |
| Word Embedding: [XLM](https://github.com/huggingface/transformers#model-architectures)|Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``xlm-mlm-en-2048`` |
| Word Embedding: [RoBERTa](https://github.com/huggingface/transformers#model-architectures) |Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``roberta-base`` |
| Word Embedding: [DistilBERT](https://github.com/huggingface/transformers#model-architectures) | Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``distilbert-base-uncased`` |
| Word Embedding: [CTRL](https://github.com/huggingface/transformers#model-architectures) | Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``ctrl`` |
| Word Embedding: [ALBERT](https://github.com/huggingface/transformers#model-architectures) | Supported by these [pretrained embeddings](https://huggingface.co/transformers/pretrained_models.html) <br> Defaults to ``albert-base-v2`` |

### Available Transformations

| Transformations | Notes |
| :---------------: | :-----: |
| [Singular Value Decomposition (SVD)](https://pytorch.org/docs/stable/torch.html#torch.svd) | Differentiable |
| [Latent Dirichlet Allocation (LDA)](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.LatentDirichletAllocation.html#sklearn.decomposition.LatentDirichletAllocation) | Not differentiable |
| [Non-negative Matrix Factorization (NMF)](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.NMF.html#sklearn.decomposition.NMF) | Not differentiable |
| [Uniform Manifold Approximation and Projection (UMAP)](https://umap-learn.readthedocs.io/en/latest/parameters.html) | Not differentiable |  
| Pooling Word Vectors | Applies to word embeddings only <br> Reduces word-level vectors to document-level <br> Pool options include ``max``, ``min``, ``mean``, ``first``, and ``last`` <br> Defaults to ``max`` |

## Usage Examples
Examples can be found under the [notebooks](notebooks) folder.
 
## Installation
TextWiser can be installed from the provided wheel package or by building from source by following the instructions
in our [documentation](docs/installation.html). 

## Compound Embedding
A unique research contribution of TextWiser lies in its novel approach in creating embeddings from components, 
called the Compound Embedding. 

This method allows forming arbitrarily complex embeddings, thanks to a 
context-free grammar that defines a formal language for valid text featurization. You can see the details
in our [documentation](docs/compound.html) and in the [usage example](notebooks/basic_usage_example.ipynb).

## Fine-Tuning for Downstream Tasks
All Word2Vec and transformer-based embeddings and any embedding followed with an ``svd`` transformation are fine-tunable for downstream tasks. 
In other words, if you pass the resulting fine-tunable embedding to a PyTorch training method, the features will automatically 
be trained for your application. You can see the details in our [documentation](docs/fine_tuning.html)
and in the [usage example](notebooks/finetune_example.ipynb).

## Tokenization
In general, text data should be **whitespace-tokenized** before being fed into TextWiser. 
Customized tokenization is also supported as described in more detail 
in our [documentation](docs/fine_tuning.html)

## Support

Please submit bug reports and feature requests as [Issues]().

For additional questions and feedback, please contact us at 
[textwiser@fmr.com](mailto:textwiser@fmr.com?subject=[Github]%20TextWiser%20Feedback).

## License
TextWiser is licensed under the [Apache License 2.0](LICENSE).

<br>