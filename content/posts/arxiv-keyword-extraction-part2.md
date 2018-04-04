+++
title = "Keyword Extraction from arXiv - Part 2"
date = "2017-10-12"
+++

In [Part 1]({filename}./arxiv-keyword-extraction-part1.md), we developed a keyword extraction algorithm.
The next step is to modify the algorithm to use database.
Configuring Postgres is more involved than in Flask by Example, since we need models to store article data.
The following diagram shows what our finished product will look like.

![App diagram]({filename}/images/keyword-extraction-diagram.svg)

We use the end product of [Flask by Example](https://realpython.com/blog/python/flask-by-example-part-1-project-setup/) tutorial as a boilerplate.
Complete Part 1-4 of Flask by Example, or clone [the repo](https://github.com/realpython/flask-by-example) of and configure Postgres by following these steps:

* Install and initialize PostgreSQL.
* Create database `arxiv_project`. This can be accomplished in many ways, including psql and SQLAlchemy.
* Add `DATABASE\_URL="postgresql://localhost/arxiv_project"` in `.envrc`.
* Run `python manage.py db {init,migrate,upgrade}`.
   `init` creates `migrations/` with a few files that are managed by Flask-Migrate.
   `migrate` creates an Alembic migration script in `migrations/versions/` based on definitions in `model.py`.
   `upgrade` runs the migration script to make changes to the database.

We will modify `app.py` and `models.py` in this article.


## 3. Modify keyword extraction to use Postgres

The following code shows how our keyword extraction function will be called.

<details open>
<summary>`app.py` - `get_keywords`</summary>

    @app.route('/start', methods=['POST'])
    def get_keywords():
        data = json.loads(request.data.decode())
        category = data['category']
        job = q.enqueue_call(
            func = get_keywords_func, args=(category,), result_ttl=3000
            )
        return job.get_id()

    def get_keywords_func(category):
        keywords = extract_keywords(category)
        result = Result(
                category = category,
                keywords = keywords
                )
        db.session.add(result)
        db.session.commit()
        return result.id
</details>
As in Flask by Example, the user makes a POST request to Flask's `/start` endpoint with an argument to initiate a function call that gets queued to Redis.

This is what our main function looks like currently.

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

The only part that needs an update is the `load_data` function.
To define our new `load_data`, however, we first need to define the structure of the database.

### 3.1 Create data models 

Each article can have several authors, and (almost) every author publishes multiple articles.
An article and category have the same relationship.
So we will model an article as many-to-many relationships between `Article` and `Author`, and `Article` and `Category`.
Flask-SQLAlchemy's [documentation](http://flask-sqlalchemy.pocoo.org/2.3/models/#many-to-many-relationships) has a concise explanation of how to construct a many-to-many relationship.

<details open>
<summary>`models.py` - `Article`</summary>

    class Article(db.Model):
        __tablename__='articles'
        id = db.Column(db.Integer, primary_key=True)
        identifier = db.Column(db.String, unique=True, nullable=False)
        title = db.Column(db.String, nullable=False)
        authors = db.relationship('Author', secondary=article_author,\
                backref='articles', lazy='dynamic') # backref furnishes Author with .articles
        abstract = db.Column(db.Text, nullable=False)
        categories = db.relationship('Category', secondary=article_category,\
                backref='articles', lazy='dynamic')
        submitted = db.Column(db.Date, nullable=False)
    
        def __init__(self, **entries):
            self.__dict__.update(entries)
    
        def __repr__(self):
            return '<article {}>'.format(self.identifier)
</details>

<details open>
<summary>`models.py` - `Author`</summary>

    class Author(db.Model):
        __tablename__ = 'authors'
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String, unique=True, nullable=False)
    
        def __init__(self, name):
            self.name = name
    
        def __repr__(self):
            return '<author {}>'.format(self.name)
</details>

<details open>
<summary>`models.py` - `Category`</summary>

    class Category(db.Model):
        __tablename__ = 'categories'
        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String, unique=True, nullable=False)
    
        def __init__(self, category):
            self.name = category
        
        def __repr__(self):
            return '<category {}>'.format(self.name)
</details>

Then, relate the models by the following relation tables.

<details open>
<summary>`models.py` - relation tables</summary>

    article_category = db.Table('article_category',
            db.Column('category_id', db.Integer, db.ForeignKey('categories.id')),
            db.Column('article_id', db.Integer, db.ForeignKey('articles.id'))
            )

    article_author = db.Table('article_author',
            db.Column('author_id', db.Integer, db.ForeignKey('authors.id')),
            db.Column('article_id', db.Integer, db.ForeignKey('articles.id'))
            )
</details>

The `Result` table, where Redis writes results of `extract_keywords`, is the same as in Flask by Example.

### 3.2 Populate Postgres with articles

The code to insert an article to the database is similar to `metha_to_df.py` from Part 1.
The following is the gist of creating an Article object from a parsed xml.
    
    article = Article()
    author = Author(author_name)
    article.authors.append(author)

An instance of `Author` is created with `author_name`, which is the string parsed from a record in a xml.
The last line adds the author to `article.authors` which is like a list.
*Note that we do not write commands to modify `Author` and `article_author`*, which is a detail that SQLAlchemy handles for us.
When commited, SQLAlchemy will tell the database to populate the `Author` table with the `author` above, and create a row in `article_author` to link the `article` and `author`.

Add categories to `article` in the same fasion, supply other entries such as title and identifier, and run `db.session.add(article)` to insert the article into Postgres.
See this [Gist](https://gist.github.com/atkm/301a50776b2831a52c58a4b0e397508c) for the full code to populate the database with articles in `~/.metha`.

### 3.3 Updated `load_data`

Now that the database is set up for use, we redefine `load_data`.
Define `pull_abstracts` that reads the database and creates a DataFrame.
The new `load_data` calls `pull_abstracts` to obtain the DataFrame instead of reading from a pickle file.

<details open>
<summary>`pull_abstracts`</summary>

    :::python
    from app import db
    from models import *

    def pull_abstracts(category):
        articles = db.session.query(Article)\
                .filter(Article.id == article_category.c.article_id)\
                .filter(Category.id == article_category.c.category_id)\
                .filter(Category.name == category)
    
        identifier, title, authors, abstract, categories, submitted = ([] for i in range(6))
        for row in articles:
            title.append(row.title)
            abstract.append(row.abstract)
    
        df = pd.DataFrame({
            'title': title,
            'abstract': abstract,
            })
        return df
</details>
The first two `filter` operations accomplishe the join of `Article` and `Category` tables.

<details open>
<summary>`load_data`</summary>

    def load_data(category):
        print('Pulling data from Postgres.')
        df = pull_abstracts(category)
        df['abstract'] = df['abstract'].apply(lambda t: t.replace('\n',' '))\
                .apply(remove_stopwords)
        df['title'] = df['title'].apply(lambda t: t.replace('\n',' '))\
                .apply(remove_stopwords)
        abs_c = df.apply(lambda row:
                '. '.join([row['title'], row['abstract']]), axis=1)
        print('Data loaded.')
        return abs_c
</details>


## 4. Configure API endpoints of Flask 

In this section, we modify `app.py` to replace the wordcount function developed in Flask by Example with our keyword extraction function.
Other files (`manage.py`, `worker.py`, etc.) are not modified.

The actual implementation of the /start endpoint differs from the snippet shown in the previous section.
When the user submits a category, Flask first looks up the Results table to see if the keywords of the category has been computed.
If that is the case, it returns the id of the keywords in the table; otherwise, the job to compute keywords is pushed to the queue.

<details open>
<summary>/start endpoint</summary>

    @app.route('/start', methods=['POST'])
    def get_keywords():
        data = json.loads(request.data.decode())
        category = data['category']
        search = Result.query.filter_by(category=category)
        if db.session.query(search.exists()).scalar():
            id = search.first().id
            return '_'.join(['nocompute', str(id)])
        else:
            job = q.enqueue_call(
                    func = get_keywords_func, args=(category,), result_ttl=3000
                    )
            return job.get_id()
    
    def get_keywords_func(category):
        keywords = extract_keywords(category)
        result = Result(
                category = category,
                keywords = keywords
                )
        db.session.add(result)
        db.session.commit()
        return result.id
</details>

The /results endpoint is configured to handle the two cases.
<details open>
<summary>/results endpoint</summary>
    
    @app.route('/results/<job_key>', methods=['GET'])
    def get_results(job_key):
        nocompute = re.search('nocompute_(\d+)', job_key)
        if nocompute:
            id = nocompute.group(1)
            return return_keywords(id)
        else:
            job = Job.fetch(job_key, connection=conn)
            if job.is_finished:
                return return_keywords(job.result)
            else:
                return 'Nope!', 202
    
    def return_keywords(id):
        result = Result.query.filter_by(id=id).first()
        return jsonify(result.keywords)
</details>

## Conclusion of Part 2
We are done rewriting our extraction algorithm to use Postgres.
This is the function that we will push to Heroku.
All that remains is to create the UI that communicates with Flask.
Go to [Part 3]({filename}./arxiv-keyword-extraction-part3.md)
