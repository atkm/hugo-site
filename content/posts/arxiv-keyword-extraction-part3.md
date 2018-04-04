+++
title = "Keyword Extraction from arXiv - Part 3"
date = "2017-10-13"
+++

This is the final part of the tutorial.
We furnish the app with an UI, and deploy it to Heroku.

## 5. Build UI

### 5.1 `index.html` and `main.js`
We first create a dropdown list.
The selected category is stored in `selected`, and it is posted to /start when `submit` is called.
Afterwards, the component polls /results.

<details open>
<summary>`main.js` - `categoryDropdown`</summary>

    var categoryDropdown = new Vue({
      el: '#category-dropdown',
      data: {
        selected: ''
      },
      methods: {
        submit: function() {
          keywordsResult.hide();
          keywordsResult.beginLoading();
          axios.post('/start', {'category': this.selected})
          .then(
            response => {
              this.pollResult(response.data);
            } 
          )
          .catch(error => { console.log(error); });
        },
    
        pollResult: function(jobID) {
          axios.get('/results/' + jobID)
          .then(
            response => {
              status = response.status;
              data = response.data;
              if (status === '202') {
                console.log(data, status);
                setTimeout(this.pollResult, 5000, jobID); // modify to give up after a while
              } else if (status === '200') {
                console.log(data, status);
                keywordsResult.supply(data);
                keywordsResult.doneLoading();
                keywordsResult.appear();
              }
            }
          ).catch( error => {
            keywordsResult.doneLoading();
            console.log(error);
          })
        }
      }
    })
</details>

The following is the corresponding div in `index.html`.
`math_categories` is a list that will be defined in `app.py`
The classes are from bootstrap.

<details open>
<summary>`index.html` - div `category-dropdown`</summary>

    <div class="col-sm-3 col-sm-offset-1">
      <div id='category-dropdown'>
        <div class='dropdown form-group'>
          <select type='text' class='form-control' style='max-width: 150px;'
            v-model='selected' autofocus required>
            <option disabled selected value=''>Select category</option>
            {% for c in math_categories %}
            <option>{{ c }}</option>
            {% endfor %}
          </select>
        </div>
        <button type='submit' class='btn btn-default' v-on:click='submit'>Submit</button>
      </div>
    </div>
</details>

Once `categoryDropdown` receives keywords from /results, it passes those to `keywordsResult`, which in turn displays the keywords.
This components is hidden until keywords are received and it shows a spinner gif while waiting for the result.

<details open>
<summary>`main.js` - `keywordsResult`</summary>

    var keywordsResult = new Vue({
      el: '#keywords-result',
      data: {
        keywords: [],
        show: false,
        loading: false
      },
      methods: {
        supply: function(d) {
          this.keywords = d;
        },
        appear: function() { this.show = true; },
        hide: function() { this.show = false; },
        beginLoading: function() { this.loading = true; },
        doneLoading: function() { this.loading = false; }
      }
    })
</details>

<!-- need to escape jinja templating -->
<details open>
<summary>`index.html` - div `keywords-result`</summary>

{% raw %} 
    <div class="col-sm-5 col-sm-offset-1">
      <div id='keywords-result'>
        <table class='table table-striped' v-if='show'>
          <thead>
            <tr>
              <th>Keywords</th>
            </tr>
          </thead>
          <tbody>
            {% raw %}
            <tr v-for='k in keywords'>
              <td>{{ k }}</td>
            </tr>
          </tbody>
        </table>
        <br>
        <img class='col-sm-4'
        src="{{ url_for('static', filename='spinner.gif') }}"
        v-if='loading'>
      </div>
    </div>
{% endraw %}
</details>

Finally, include packages in the header and `main.js` in the body.

<details open>
<summary>`index.html` - header</summary>

{% raw %}
    <head>
      <title>{ keywords(x) | x \in {math research area} }</title>
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <link href="//netdna.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet" media="screen">
      <link rel='stylesheet' type='text/css' href="{{ url_for('static', filename='main.css') }}">
      <link rel='shortcut icon' href="{{ url_for('static', filename='favicon.ico') }}" type='image/x-icon'>
      <style>
        .container {
          max-width: 1000px;
        }
      </style>
      <script src="//code.jquery.com/jquery-2.2.1.min.js"></script>
      <script src="//netdna.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
      <script src="https://unpkg.com/vue"></script>
      <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    </head>
{% endraw %}
</details>

<details open>
<summary>`index.html` - somewhere in body</summary>
    
{% raw %}
    <script src="{{ url_for('static', filename='main.js') }}"></script>
{% endraw %}
</details>

### 5.2 Configure the root endpoint

<details open>
<summary>`app.py` - root endpoint</summary>

    @app.route('/', methods=['GET', 'POST'])
    def index():
        with open(os.path.join(app.config['STATIC_PATH'], 'math_categories.txt')) as f:
            math_categories = f.read().splitlines()
        return render_template('index.html', math_categories=math_categories)
</details>

To make this work, create `static/` in the project root, and place `math_categories.txt` containing all categories.
Also, add `STATIC_PATH = os.path.join(basedir, 'static/')` in the base Config class in `config.py` to have Flask recognize the directory.

<details open>
<summary>`/static/math_categories.txt`</summary>

    math.OC
    math.IT
    math.AP
    math.OA
    math.RA
    math.NA
    math.SG
    math.AT
    math.NT
    math.AG
    math.CO
    math.GT
    math.DG
    math-ph
    math.MP
    math.DS
    math.QA
    math.SP
    math.CA
    math.PR
    math.LO
    math.FA
    math.MG
    math.GN
    math.CV
    math.CT
    math.KT
    math.GR
    math.RT
    math.ST
    math.HO
    math.AC
    math.GM
</details>

### 5.3. Run it locally
Our app is ready to run locally!
Start an instance of Redis server (usually `redis-server`), followed by `python worker.py` which starts a [worker](http://python-rq.org/docs/workers/) that takes jobs from a Redis queue.
Finally, run `python manage.py runserver`.

## 6. Deploy to Heroku.
- Login to your account via Heroku CLI.
- Place `nltk.txt` in the project root containing a single line `stopwords`. This file tells the Heroku app to download the 'stopwords' corpora.
- Place `Procfile` in the project root containing a single line `sh heroku.sh`.
  Also create the following in the project root.
  Heroku executes `Procfile`, which in turn executes `heroku.sh`.
  This starts a gunicorn HTTP server and a worker.

<detail open>
<summary>`heroku.sh`</summary>
    
    :::
    #!/bin/bash
    gunicorn app:app --daemon
    python worker.py
</detail>

- Run `git init` in the project root to make it a git repo. Edit `.gitignore`, run `git add -A`, then commit.
- `heroku create yourappname` creates an app named `yourappname` under your Heroku account.
- `git remote add heroku git@heroku.com:yourappname.git`. This prepares your git repo to be pushed to the address associated with your Heroku app.
- `git push heroku master`. Now your code is on Heroku.
- This command tells Heroku to use the right config class in `config.py`: `heroku config:set APP_SETTINGS=config.StagingConfig --remote heroku` 
- `heroku addons:create heroku-postgresql:hobby-dev --app yourappname` attaches a Postgres database to your Heroku app.
  `DATABASE_URL` of the database is in `heroku config --app yourappname`.
- Attach Redis: `heroku addons:create redistogo:nano --app yourappname`
- You can review the configuration of your app with `heroku config --app yourappname`.

The commands thus far set up our Flask app in Heroku with Postgres and Redis.
The app is good to go once the database on Heroku is configured.
We need the database to have the tables filled with rows.

- Run `pg_dump -U postgres -Fc --no-acl --no-owner arxiv_project > arxiv_project.dump` to export the local database in the binary format.
- Place `arxiv_project.dump` in AWS, Dropbox, or somewhere accessible via HTTP.
- Import the dumped database: `heroku pg:backups:restore 'https://url.to.arxiv_project.dump' DATABASE_URL --app yourappname`. Try http if https fails.

## Conclusion
That's it!
Our app is ready for the whole world to see.
I hope this gave you an idea as to how you might go about deploying your algorithm.

Our goal, that is to deploy a machine learning code, is achieved.
Our keyword extraction algorithm, however, leaves a lot to be desired.
What immediately comes to my mind are the following.

- Update the metadata database on a daily basis.
- Decide how long to cache computed keywords. Currently, they are stored in the database indefinitely.
- Visualization of keywords. Use D3, for example.
- Implement the per-cluster keyword extraction routine in a distributed computing framework such as Spark.
- The keyword extraction algorithm needs work. There are many possibilities---use different extraction algorithms, feature engineering, tweak the existing tf-idf algorithm, use different clustering algorithms, or use more data, namely, [full-text data](https://arxiv.org/help/bulk_data_s3).

Improving the algorithm is a difficult task in its own right, but that is not the only hurdle to overcome.
The app developer needs to ensure that the improved app is computationally feasible (e.g. memory usage), introduce an infrastructure that supports a new version of the algorithm (e.g. Redis data structures, new tables or constraints in the database), and check that each component fits together after modifications.
Thus, app development is both a scientific and engineering problem. 
Note that none of the possible improvements listed above is a purely scientific problem; in the meantime, every one of them requires engineering work.
