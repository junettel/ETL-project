# ETL-project
ETL Project

## Github Repositories:
Julie Wells - https://github.com/jcwells18/ETL-Project \
Junette Lee - https://github.com/junettel/ETL-project \
Amusa Adebayo - https://github.com/aadeba5/ETL-PROJECT


## Data Sources:
Streaming Data - [Movies on Netflix, Prime Video, Hulu and Disney+](https://www.kaggle.com/ruchi798/movies-on-netflix-prime-video-hulu-and-disney) \
IMBb Data - [IMDb movies extensive dataset](https://www.kaggle.com/stefanoleone992/imdb-extensive-dataset)


## Extract
* Downloaded data CSVs from Kaggle and loaded into Python to use Pandas for transformation and cleanup
  * **MoviesOnStreamingPlatforms_updated.csv**
    * Provides comprehensive list of movies on streaming platforms with film information
  * **imdb_movies.csv**
    * Includes additional data (`description` and `imdb_title_id`)
  * **imdb_title_principals.csv** 
    * Provides mapping of `imdb_title_id` and `imdb_name_id` of cast members
  * **imdb_names.csv**
    * Includes information on movie cast and crew members along with an `imdb_name_id`

## Transform
Steps to complete data cleanup & analysis:
  * Cleaning:
    * Dropping unnecessary columns from each source
      * Mainly dropped columns from *imdb_movies.csv* as *MoviesOnStreamingPlatforms_updated.csv* and *imdb_movies.csv* had overlapping/repetitive columns, so we 
    * Cleaning column names to omit spaces so information smoothly flows to pgAdmin
    * Normalizing rating metrics between IMDb (float, max 10) and Rotten Tomatoes (object, max 100%)
      * Stripping ‘%’ sign from Rotten Tomatoes rating so it may be formatted as a float rather than object
    * Split out name of most recent spouse from `spouses_string` in *imdb_names.csv*
    * Split out place of birth data string from *imdb_names.csv* into separate columns
    * Split out place of death data string from *imdb_names.csv* into separate columns

  * Joining: 
    * Joined/merged *MoviesOnStreamingPlatforms_updated.csv* and *imdb_movies.csv* by `title` and `year` to incorporate additional IMDb detail with streaming detail (`description` and `imdb_title_id`)
      * This enables querying *imdb_names.csv* cast and crew member 
    * By joining *MoviesOnStreamingPlatforms_updated.csv* with *imdb_movies.csv*, we are able to add `imdb_title_id`

  * Filtering: 
    * Filtered out all movies not currently on one of the streaming platforms through an inner join between *MoviesOnStreamingPlatforms_updated.csv* and *imdb_movies.csv*
    * Inner join also filters out movies on streaming platforms missing IMDb data or with contradicting `title` and `year` information so only exact matches are included

  * Aggregating: 
    * Average `imdb_rating` and `rotten_tomatoes_converted` ratings calculated after Load step
      * Can be done either in pgAmin by SQL query or by using Python and SQLalchemy
    * Query a list of movies available on each streaming platform meeting a certain criteria. Here are the examples included in [etl-project.ipynb](etl-project.ipynb):
      * Find **`movies`** streaming on each platform with `imdb_rating` greater than or equal to 7
      * Find all cast and crew members from **`names`** table from Chicago
        * This can be taken a step further to employ subqueries to find all movies on streaming platforms based on cast/crew related details (e.g. query all movies with a specific actor)

## Load
* Loaded data tables into relational database, pgAdmin (postgres)
* Final tables loaded to **`streaming_db`** pgAdmin (postgres) database:
  * **`movies`** - streaming platform movies with unique `imdb_title_id` index
  * **`names`** - actor/personnel information with unique `imdb_name_id` index
  * **`title`** - provides mapping between Movies and Actors Tables using `imdb_title_id` and `imdb_name_id`
* Why these tables?
  * We chose these tables to be able to utilize the `imdb_title_id` and `imdb_name_id` mappings to connect movies available on streaming platforms (**`movies`** table) with the corresponding cast and crew information (**`names`** table)
* Why a relational database?
  * We chose a relational database because the `imdb_title_id` and `imdb_name_id`, unique alphanumeric identifiers for each movie and cast/crew, connect the three database tables and broaden the search and query possibilities while keeping the tables more focused