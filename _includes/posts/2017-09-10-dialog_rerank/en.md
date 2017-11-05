# Build QA Bot from Dumped Dialog

*author: Xuan Wang*  


## 1. Intro
This is a summary from my work in [Laiye, Inc](http://www.laiye.com). I will be mainly looking at the processes not specific algorithems or models. 
### 1.1 Our Context
Customer service covers from presale to postsale, traditionally this work relies heavily on humans. But with a QA bot, we can:  
* alleviate CSR's(Customer Service Representitive) workload, and raise the efficiency, (statistics from [Laiye, Inc](http://www.laiye.com) has shown a leap from 1:100 CSR-User rate to 1:400!)  
* standardize the service and its quality, enable the company to accumulate and develop its knowledge base  
* build a positive loop of "knowledge base - QA bot - human labeling", also note that we include human in the loop, which not only put a little cusion between QA bot and users but as well provide labeling to the dialog data  


### 1.2 System Overview
The leading QA bots nowadays follow a cascade pattern of "Search" and "Rerank":  
![](/img/in-post/1/architecture.png)
courtesy: [AliMe Chat](http://www.aclweb.org/anthology/P17-2079)  

First, the search engine(the green part) retrieves some number of response candidates; then hand those to reranking system(the yellow part) to do a finer-grained ranking to output.  
The reason to this design is to reduce the response time while maintain the most recall. Search engine is flashing fast but lacks of the ability to capture semantics. So reranking system is thus designed to make up. Reranking system is modeled more complicatedly, e.g. using deep networks, and usually it is intolerable to feed all candidates from knowledge base to it.  So, we stack them.  

Here we talk about the reranking system.  


### 1.3 Multi-turn Reranking System
In production cases, a dialog about a topic may includes multiple turns of utterances. One can use slot fitting to collect all the arguments if it is in a taskbot setting when an API is meant to be called. But in our case, we only have the knowledge base at hand(i.e. logs of questions and answers), so we do not have predefined slots to fit. Thus if we want to capture more contextual information, we need to look further back to take more utterances into account, which means a "multi-turn" model.  
My model is a reimplementation of [Sequential Matching Networks](http://www.aclweb.org/anthology/P17-1046).

### 1.4 Terminology
![](/img/in-post/1/table1.png)
  
  

## 2. Corpus Preparation
### 2.1 Munging
What we have is historical dumped dialog data, cleansing works include(non-exhaustively):  
* remove {illegal chars, timestamps/sys log/ID info or other tags, ...}  
* combine consecutive utterances from the same ID into the one  
* split out sessions(e.g. by '\n')


If you don't have a satisfying amount of data, you may want to try some data augmentation, e.g. shuffling in small granularities.  

### 2.2 Make Train/Valid/Test Sets
#### 2.2.1. Pick a window size, slide through
Say if we consider three turns, then it will be a total of 3 * 2 = 6 utterances, so the window size will be 6.  
#### 2.2.2. Mix in negative samples  
We assume all the responses from CSRs are gold answers, but since our system is a scoring system, we need to train with negative samples otherwise the system will just learn to always give a high score.  
For every context, random draw a response from corpus to it to make a negative sample. Train set mix at 1:1, valid and test 1:9. Train set best to keep a balance between positive and negative samples, but valid and test are best to be decided according to the actual business. For example if the production system returns 10 candidates, then we should recall from the 10, so we mix valid and test at a rate of 1:9. But the final system may have several good candidates, which is when we need human to evaluate subjectively.

## 3. Vocabulary and Embedding Preparation
### 3.1 User-defined words  
A major difference of Chinese NLP from English is we need to do word segmentation first. Typically a tokenizer is used, such as "jieba". But for domain specific task we need to expand the built-in vocabulary of the tokenizer so that those domain specific words can be captured.  
Note, we must make sure: 
```
Always make the vocabulary consistent when 1)train embeddings, 2)train model and 3)online serving.
```
Concretely, add user-defined words after `import jieba` immediately. This bunch words should also keep the same version with upstream searching teammates.  

### 3.2 Embeddings Preparation
#### 3.2.1. Collect corpus
Collect in-domain articles, together with our dumped dialog data.
#### 3.2.2 Export two files after done training embeddings:  
![](/img/in-post/1/table2.png)
In fact `w2i` and `w2v` have two more vectors than embeddings, namely `<PAD>` and `<UNK>`.  

## 4. Offline Stage
#### Codes Needed
![](/img/in-post/1/table3.png)
  
## 5. Online Stage
Just need to be familiar TFserving. Server side code can adopt from Tensorflow's official ranking serving code, but pay a little more attention to client side.  
Still one thing to reiterate:  
```
Always make the vocabulary consistent when 1)train embeddings, 2)train model and 3)online serving.
```
