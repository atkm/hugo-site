+++
title = "Keyword Extraction from arXiv - Part 1"
date = "2017-10-11"
+++

This is a tutorial on web development written for people with a statistical analysis, scientific computing, or machine learning background.
We start with an algorithm using data that fits comfortably into memory, and modify it to accept a large input.
We then set up an infrastructure to serve the resulting algorithm.
This tutorial focuses on the infrastructure rather than the algorithm, which will remain rudimentary.
The end product is a Heroku deployment of a text summarization algorithm that analyzes articles on arXiv to extract keywords from each research category within mathematics.
Find my deployment [here](https://arxiv-math-kws-stage.herokuapp.com/).

Outline:
  
  1. Scrape articles from arXiv (Part 1; we are here)
  2. Keyword extraction prototype in Pandas (Part 1)
  3. Modify keyword extraction to use Postgres ([Part 2](/posts/arxiv-keyword-extraction-part2))
  4. Configure Flask and Redis (Part 2)
  5. Build UI in Vue.js ([Part 3](/posts/arxiv-keyword-extraction-part3))
  6. Deploy to Heroku (Part 3)

My Flask setup is based on the one described in this tutorial: 
[Flask by Example](https://realpython.com/blog/python/flask-by-example-part-1-project-setup/).

If Flask by Example is incomprehensible, I recommend going through the following tutorials first:

  - [Rails Tutorial](https://www.railstutorial.org/book): this is a good web development primer.
  - [Angular's Official Tutorial](https://angular.io/tutorial): doing this tutorial helps understanding other frameworks, as concepts are similar accross front-end frameworks.

Tools used: Flask, PostgreSQL, SQLAlchemy, Alembic, Redis, Pandas, Scikit-learn, Vue.js, Heroku


## 1. Scrape articles from arXiv

arXiv provides three APIs---OAI-PMH, arXiv API, and RSS---to access article metadata.
The data available via OAI-PMH is updated on a daily basis, while the arXiv API provides access to real-time data.
We use [OAI-PMH](https://arxiv.org/help/oa/index), which is what they recommend for bulk access, and [metha](https://github.com/miku/metha) (thanks!) to scrape data.
Once installed, run:

    ./metha-sync -format arXiv -set math http://export.arxiv.org/oai2

This downloads all metadata of math articles to `~/.metha`.
The choice of format is not crucial, though those who have used arXiv will find the arXiv format more familiar than the oai\_dc format.
The command takes several hours to complete.

Now, we have gunzipped xml files in a directory inside `~/.metha`.
The data is just over 100MB, so all metadata can be loaded into memory.
We will eventually develop an algorithm that loads necessary data from a database to memory; developing a functioning code from scratch, however, takes many small iterations, so we start with a function that acts on in-memory data.

The code below parses the xml files, creates a Pandas DataFrame, and saves it to a pickle file.
Each record in a xml file has a structure like [this](http://export.arxiv.org/oai2?verb=GetRecord&identifier=oai:arXiv.org:0804.2273&metadataPrefix=arXiv), which Python's ElementTree handles.

<details open>
<summary>`metha_to_df.py`</summary>

    import os, gzip, glob
    import pickle
    from datetime import datetime
    import xml.etree.ElementTree as ET
    import pandas as pd
    
    if __name__=='__main__':
        files = glob.glob('/home/atkm/.metha/20170923/*.xml.gz')
        arXiv_prefix = '{http://arxiv.org/OAI/arXiv/}'
    
        def parse_author(a):
            keyname = a.find(arXiv_prefix + "keyname")
            keyname = keyname.text if keyname is not None else ''
            forenames = a.find(arXiv_prefix + "forenames")
            forenames = forenames.text if forenames is not None else ''
            return ' '.join( (forenames, keyname) )
    
    
        identifier, title, authors, abstract, categories, submitted = ([] for i in range(6))
        for xmlgz in files:
            print(os.path.basename(xmlgz), end=' ')
            with gzip.open(xmlgz, 'rb') as f:
                xml = ET.parse(f)
                root = xml.getroot()
                for record in root.find('ListRecords').findall("record"):
                    metadata = record.find('metadata').find(arXiv_prefix + "arXiv")
                    if metadata is None:
                        continue
    
                    identifier.append( metadata.find(arXiv_prefix + "id").text )
                    s = metadata.find(arXiv_prefix + "created").text
                    submitted.append( datetime.strptime(s, "%Y-%m-%d") )
                    title.append( metadata.find(arXiv_prefix + "title").text )
                    abstract.append( metadata.find(arXiv_prefix + "abstract").text.strip() )
    
                    authors_list = metadata.find(arXiv_prefix + 'authors')
                    authors.append( tuple( [parse_author(a) for a in authors_list] ) )
    
                    categories_list = metadata.find(arXiv_prefix + "categories").text
                    categories.append( tuple( categories_list.split() ) )
            print(' Done.')
    
        df = pd.DataFrame({
            'identifier': identifier,
            'title': title,
            'authors': authors,
            'abstract': abstract,
            'categories': categories,
            'submitted': submitted
            })
    
        with open('metha-all-math.pkl', 'rb') as f:
          pickle.dump(df, f)
</details>


## 2. Keyword extraction prototype in Pandas

The algorithm that we wish to develop falls into the category of text summarization.
There are recent developments in the field (
[Salesforce](https://www.salesforce.com/blog/2017/05/ai-salesforce-research-text-summarization.html),
[Google Brain](https://research.googleblog.com/2016/08/text-summarization-with-tensorflow.html),
[Facebook Research](https://research.fb.com/publications/abstractive-summarization-with-attentive-rnn-naacl-2016/)
).
The techniques described in the linked articles seem successful, but too involved for our purpose.
These are some other keyword extraction techniques that are readily available:

- tf-idf: a modification of bag-of-words to put greater weights on rare words.
- Latent Semantic Indexing (LSI or LSA): a low-rank approximation of bag-of-words (or tf-idf) by taking its the principal components. pLSI and Latent Dirichlet Allocation extends LSI.
- TextRank: application of Google's PageRank algorithm to graph of sentences, or, in our case, n-grams.
- Rapid Automatic Keyword Extraction

I tried tf-idf and TextRank, and the former works reasonably well.
This is the outline of our tf-idf-based keyword extraction algorithm:

1. Load the pickled DataFrame and extract articles of a category. 
  Extract title and abstract from each article.
2. Vectorize the articles by tf-idf and run k-means clustering. Each cluster corresponds to a subdiscipline.
3. Within each cluster, compute tf-idf of n-grams of articles to find significant terms.

As I said in the introduction, the algorithm is kept simple---working just well enough to illustrate steps involved in deploying an algorithm.

The main purpose of this section is to discuss out-of-core computation.
Although the entire text data easily fits in memory (~100MB), and a slice of the data by category is even smaller (18k rows for math.DS), computation that uses it can have a large memory footprint.
In our case, a bag of 2-grams for math.DS has 600k columns, and k-means clustering using sklearn's KMeans requires gigabytes of memory, which will render clustering infeasible on a small machine like mine (4GB of RAM).
An attempt to run SVD on the term-document matrix to reduce the dimension faces the same challenge.
A solution to such computational difficulty is to use incremental algorithms.
[This page](http://scikit-learn.org/stable/modules/scaling_strategies.html) provides a list of out-of-core-enabled algorithms.
In our case, we use MiniBatchKMeans instead of KMeans.
This not only solves our space complexity issue, but also expedites computation.
There is, however, a tradeoff that the clustering obtained by the minibatch variant is an approximation.

### 2.1 Load data
Load the Pandas DataFrame from the pickle file created in the previous section,
slice the title and abstract columns, and concatenate title and abstract.
So each article is represented by a single string which is a concatenation of its title and abstract.

<details open>
<summary>`load_data`</summary>

    from nltk.corpus import stopwords
    
    def load_data(category):
        print('Loading data')
        with open('metha-all-math.pkl','rb') as f:
            df = pickle.load(f)
        
        df['abstract'] = df['abstract'].apply(lambda t: t.replace('\n',' '))\
                .apply(remove_stopwords)
        df['title'] = df['title'].apply(lambda t: t.replace('\n',' '))\
                .apply(remove_stopwords)
        
        mask = df['categories'].apply(lambda cs: True if category in cs else False)
        df_c = df[mask]
        abs_c = df_c.apply(lambda row: 
                '. '.join([row['title'], row['abstract']]), axis=1)
        print('Data loaded')
        return abs_c

    def remove_stopwords(text):
        stopWords = set(stopwords.words('english'))
        return ' '.join([w for w in text.split() if w not in stopWords])

</details>


### 2.2 Vectorization of articles and clustering

This is where we use MiniBatchKMeans as discussed above.
MiniBatchKMeans approximates the usual KMeans better when the batch size is closer to the sample size.

<details open>
<summary>`cluster_docs`</summary>

    def cluster_docs(abstracts, K):
        tfidf_vect = TfidfVectorizer()
        abs_tfidf = tfidf_vect.fit_transform(abstracts)
        print('Running kmeans')
        kmeans = MiniBatchKMeans(n_clusters=K, batch_size=1000, reassignment_ratio=0).fit(abs_tfidf)
        print('Kmeans finished')
        cluster = kmeans.predict(abs_tfidf)
        return cluster
</details>

### 2.3 Compute keywords

It computes the tf-idf score of each 2-, 3-, and 4-grams.

<details open>
<summary>`rank_phrases`</summary>

    import heapq

    def rank_phrases(text, n):
        tfidf_vect = TfidfVectorizer(ngram_range=(2,4))
        text_vect = tfidf_vect.fit_transform(text)
        text_vect = np.asarray(text_vect.sum(axis=0)).ravel()
        tfidf_dict = dict()
        idf = tfidf_vect.idf_
        for k, v in tfidf_vect.vocabulary_.items():
            tfidf_dict[k] = text_vect[v]
        return heapq.nlargest(n, tfidf_dict, key=tfidf_dict.get)
</details>

Putting together the snippets above, we get our keyword extraction prototype.
It extracts top 2 keywords from each cluster.

<details open>
<summary>`extract_keywords`</summary>

    def extract_keywords(category):
        K = 100
        abs_c = load_data(category)
        cluster = cluster_docs(abs_c, K)
        abs_clusters = pd.DataFrame({
            'cluster': cluster,
            'text': abs_c
        })
    
        keywords = []
        for j in range(K):
            c = abs_clusters[abs_clusters['cluster']==j]
            for phrase in rank_phrases(c['text'], 2):
                keywords.append(phrase)
    
        print(keywords)
        return keywords
</details>

## Conclusion of Part 1
Our prototype is ready.
Run it with your favorite math category to check that the algorithm returns reasonable keywords.
In the next article, we modify the function to use Postgres so it is deployment-ready.
[Go to Part 2](/posts/arxiv-keyword-extraction-part2).
