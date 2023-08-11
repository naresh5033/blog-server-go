### RSS Aggregator Project

- lets build a backend svr, the purpose of this svr will aggregate the RSS feed, the Rss is a protocol that makes the distributing things like podcast and blog post easily..
  - now our svr will users to add diff rss feed to the DB and then it will go automatically collect all of those post from the feeds and download em and save em in the DB..so we can view em later.
- and this svr we will be building is gon to be the json REST api.
- to get/ pull the package from the github `go get github.com/joho/godotenv` in this case we just need the dotenv package
- then `go mod vender` to copy that package/code into our vender dir -- kinda ve the local copy of that.
- for this project we will be using the **chi router** a 3rd party router, which is very light wight. just like the go std package `go get github.com/go-chi/chi` and also `go get github.com/go-chi/cors` the cors from the same package.

  - again to add the vender `go mod tidy` && `go mod vendor`

- the err code in the range with in 500 are the client side err so anything is gt that is the svr side error ex - err > 499

  - when we create our cust error struct we can mark it as the string type corresponding to the json, and this is called the "json reflex tag" ex - type errRes struct { err string `json:"error"`} -- so now for this json str type the error is the key and it looks like {"error": "some thing went wrong"}

- for the query we ve to install sqlc and goose packages, he uses the local postgresql db and pg admin to connect that (these packages allow us to write raw sql queries)
- the way sqlc works is it takes the sql statements and it generates types safe go code that matches sql

  - ex - --name: CreateUsesr : one .. create a user and it returns one record
  - and we can pass the args like VALUEs ($1, $2, $3) these are the parameters for the user and we can return with RETURNING \*;
  - to gen the query `sqlc generate` this will gen db dir which contains db.go and models.go and users.sql.go
  - and inside the schema dir where we create the migrations for the user and an if we wanted to add any new migrations then we ve to keep the order ex - 001_users.sql , 002_user_apikey.sql .. so by this way the goose will know which order the migrations will be done..
  - in this 002 aka the second migration we can alter the user table with add a new column to store the api key (which is of type VAR CHAR)

- when we use the error (the std package from go ), we should not use the caps ex - errors.New("must be case sensitive") .. otherwise it will throws the linting error..

- **Context()**

  - in go we ve the context package, basically it gives us the way to track something that happens across the multiple go routines.. ex - GetUserByApikey(r.Context(), apiKey)
  - the most important thing we can do with the context is we can cancel it..by cancelling the context it will effectively kill the http req.
  - and every httpreq ve the context, and we can use the context with any calls that we make within the handler that requires context..

- to grab the param id we can use the chi.URLParam(r, "feedfollowID") .. req and the id we wana grab..

- so far we ve been buit the crud part of our server..

**to fetch the Rss Feed**

- now the part of the svr goes out(periodically) and fetches the posts from the rss feed in our db..
- for that we can create a "scrapper.go" which is a long running job, so this will running in the BG, as our server runs..
- the fn will takes the no.of goroutines that we wanna run on, db, and the time b/w the reqs

**Wait Group**

- the wait Group frm the std lib - sync.WaitGroup ..
- and the way the wg works is anytime we spawn a new go routine with in the context of the wg we do like `wg := &sync.WaitGroup .. and we can use this wg inside our feed loop, and iterate as how much as feed we ve.. and for every feed it adds one wg..
  wg.Add(1)
  go AddScrap(wg)
  wg.Wait() ... then finally we can create a fn and inside we can defer the wg.Done()

- then the final feature we wanna add to our rss feed agg is, we wana able to get our users the list of newest post from the feeds they re follwing

- the project in the workplace dir - go/workplace/fcc

# Posts

## Add a `posts` table to the database

A post is a single entry from a feed. It should have:

* `id` - a unique identifier for the post
* `created_at` - the time the record was created
* `updated_at` - the time the record was last updated
* `title` - the title of the post
* `url` - the URL of the post *this should be unique*
* `description` - the description of the post
* `published_at` - the time the post was published
* `feed_id` - the ID of the feed that the post came from

Some of these fields can probably be null, others you might want to be more strict about - it's up to you.

## Add a "create post" SQL query to the database

This should insert a new post into the database.

## Add a "get posts by user" SQL query to the database

Order the results so that the most recent posts are first. Make the number of posts returned configurable.

## Update your scraper to save posts

Instead of just printing out the titles of the posts, save them to the database! If you encounter an error where the post with that URL already exists, just ignore it. That will happen a lot. If it's a different error, you should probably log it.

Make sure that you're parsing the "published at" time properly from the feeds. Sometimes they might be in a different format than you expect, so you might need to handle that.

## Add a "get posts by user" HTTP endpoint

Endpoint: `GET /v1/posts`

*This is an authenticated endpoint*

This endpoint should return a list of posts for the authenticated user. It should accept a `limit` query parameter that limits the number of posts returned. The default if the parameter is not provided can be whatever you think is reasonable.

## Start scraping some feeds!

Test your scraper to make sure it's working! Go find some of your favorite websites and add their RSS feeds to your database. Then start your scraper and watch it go to work.
