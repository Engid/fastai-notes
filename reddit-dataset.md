# Reddit Dataset

Greetings fellow [fast.ai](http://course.fast.ai) students and nerds alike. I've been very interested in the publicly available datasets for 
Reddit comments and posts and so I'd like to share with y'all my notes on how to interact with it for fun and profit (no guarantees!). 

## Raw
Raw Datasets include (sourced from [niderhoff/nlp-datasets](https://github.com/niderhoff/nlp-datasets/blob/master/README.md))


*   [Reddit Comments](https://www.reddit.com/r/datasets/comments/3bxlg7/i_have_every_publicly_available_reddit_comment/): every publicly available reddit comment as of july 2015. 1.7 billion comments (250 GB)

*   [Reddit Comments (May ‘15) [Kaggle]](https://www.kaggle.com/reddit/reddit-comments-may-2015): subset of above dataset (8 GB)

*   [Reddit Submission Corpus](https://www.reddit.com/r/datasets/comments/3mg812/full_reddit_submission_corpus_now_available_2006/): all publicly available Reddit submissions from January 2006 - August 31, 2015). (42 GB)


# BigQuery

While this raw data is awesome, I've found that the best way to play around with the data without having to downloading 
all 250 GB worth of it is through the [BigQuery tables](https://bigquery.cloud.google.com/table/fh-bigquery:reddit_comments.2015_05) created by [Felipe Hoffa](https://www.reddit.com/user/fhoffa) (thank you Mr Hoffa!). **This is a perfect solution if you're already familiar with SQL** and want to select a sub-set of this massive
treasure-trove of data. 

Go [here](https://bigquery.cloud.google.com/table/fh-bigquery:reddit_comments.2015_05) for a web-based query interface. Google
is generous enough to give away a terabyte's worth of query processing, so you can start exploring the data before putting 
any credit cards down. Also, Google has $300 worth of credits for new signups to their cloud service, so if you havent used it 
up yet that will come in handy!

You can find a lot of great queries into the data on 
  [this](https://www.reddit.com/r/bigquery/comments/3cej2b/17_billion_reddit_comments_loaded_on_bigquery/) post.

## Tips and Tricks
If you're new to [BigQuery](https://cloud.google.com/bigquery/docs/) then here's a few tips that will help you explore the dataset. 

### BigQuery SQL Syntax: 'Legacy' VS 'Standard'
First of all, there are two versions of the BigQuery SQL syntax: 
[Standard](https://cloud.google.com/bigquery/docs/reference/standard-sql/) and 
[Legacy](https://cloud.google.com/bigquery/docs/reference/legacy-sql) It's reccommended that you use the Standard syntax, 
but you'll need to be able to recognize the difference between it and Legacy. 

As far as I can tell, the main giveaway is the table references in `FROM` statements. For instance, most of the queries from 
[fhoffa's original thread](https://www.reddit.com/r/bigquery/comments/3cej2b/17_billion_reddit_comments_loaded_on_bigquery/) look something like this:
```SQL
# Legacy SQL
SELECT author 
FROM [fh-bigquery:reddit_comments.bots_201505]
```
where the reference is surrounded by square brackets `[]` and the dataset name `fh-bigquery` is followed by a colon `:`.
The Standard version of this would be surrounded in backticks and the colon replaced with a dot `.`: 
```SQL 
# Standard SQL
SELECT author 
FROM `fh-bigquery.reddit_comments.bots_201505` 
```
There's a lot more that the Standard version will do so be sure to checkout the [docs](https://cloud.google.com/bigquery/docs/reference/standard-sql/)


#### Legacy Must Be Dissabled!!!
Please note that if you'd like to use the Standard syntax **you must dissable the Legacy version**. 
See [the docs](https://cloud.google.com/bigquery/docs/reference/standard-sql/enabling-standard-sql) to find the switch. 
Also note that the BigQuery team is working on an updated web-app that will use Standard by default, so you may not need to 
worry about this in the near future. (As of 1/7/2018 Legacy is still on by default). 

### Joining Posts to Comments
This took me a disturbingly long time to [figure out](https://stackoverflow.com/questions/48133693/joining-posts-with-comments-in-the-bigquery-reddit-dataset), 
but there is a little secret to joining comments to data in their parent posts. 

The first step is that we have to do some string manipulation. Each post has an `id` field that is referenced by the `link_id`
of its child comments, but for some reason reddit engineers tacked on three preceding characters which we have to chop off by
using `SUBSTR()`. For example, each comment will have a `link_id` looking something like `t3_43go1r`, so to match the post's 
`id` of `43go1r` we must call `SUBSTR(link_id, 4)`.

Here is an example of a join (thanks to [Mikhail Berlyant](https://stackoverflow.com/a/48133745/4821960) for the help with the Standard SQL)
``` SQL
#Standard SQL
SELECT posts.title, comments.body
FROM `fh-bigquery.reddit_comments.2016_01` AS comments
JOIN `fh-bigquery.reddit_posts.2016_01`  AS posts
ON posts.id = SUBSTR(comments.link_id, 4) 
WHERE posts.id = '43go1r'
```

Also, if you want to get the root comment (meaning a direct comment to the post and not a reply to another comment) using the 
previous query add another predicate statement like so: `AND comments.parent_id = comments.link_id`. This is because a root 
comment is its own parent (according to reddits original JSON schema for comments). 

### more coming soon..

# Sources and Attributions

First off, I'd like to give a warm thanks to [u/Stuck_In_the_Matrix](https://www.reddit.com/user/Stuck_In_the_Matrix) for 
creating and curating these datasets. Also, thanks to [u/fhoffa](https://www.reddit.com/user/fhoffa) for curating the data 
on BigQuery. 

I originally found out about the Reddit datasets via [niderhoff/nlp-datasets](https://github.com/niderhoff/nlp-datasets/blob/master/README.md)

More info on [BigQuery](https://cloud.google.com/bigquery/docs/)
