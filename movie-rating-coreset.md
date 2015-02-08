
> SQL exercise questions from Coursera - Introduction to Databases course
>> SQL Movie-Rating Query Exercises (core set)

> In this part we'll use a database called 'rating'. I downloaded it from the
Introduction to databases course in coursera. The database has three tables (
'Movie', 'Rating', 'Reviewer' ). The schema is shown below.

>                                                       Movie table

>| mID | title | year | director |
  | ------ |:---:| ----:|----:|
  |    -    |   -  |   -   |   -   |

>                                                      Rating table

>| rID | mID | stars | ratingDate |
  | ------ |:---:| ----:|----:|
  |    -    |   -  |   -   |   -   |

>                                                     Reviewer table

>| rID | name |
  | ------ |:---:|
  |    -    |   -  |



    import pandas
    import mysql.connector
    # set up connections to the DB
    conn = mysql.connector.Connect(host='localhost',user='root',\
                            password='',database='rating')


    import pandas
    import mysql.connector
    # set up connections to the DB
    conn = mysql.connector.Connect(host='localhost',user='root',\
                            password='',database='rating')
    #LOAD the SQL tables into DF
    qryMv = """
            SELECT * From Movie
          """
    qryRt = """
            SELECT * From Rating
          """
    qryRe = """
            SELECT * From Reviewer
          """
    movieDF = pandas.read_sql( qryMv, conn )
    ratDF = pandas.read_sql( qryRt, conn )
    rvwrDF = pandas.read_sql( qryRe, conn )

## Question-1 : Find the titles of all movies directed by Steven Spielberg.

### Solution using SQL.


    #SQL query
    qry = """
            SELECT title FROM Movie
            WHERE director = 'Steven Spielberg'
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

                         title
    0                     E.T.
    1  Raiders of the Lost Ark
    
    [2 rows x 1 columns]


### Solution using Pandas.
> Methods used : selection


    # simple selection
    print movieDF[ movieDF['director'] == 'Steven Spielberg' ]['title']

    3                       E.T.
    7    Raiders of the Lost Ark
    Name: title, dtype: object


## Question-2 : Find all years that have a movie that received a rating of 4 or
5, and sort them in increasing order.

### Solution using SQL.


    #SQL query
    qry = """
            SELECT DISTINCT mv.year FROM Movie mv
            INNER JOIN Rating ra
            ON ra.mID = mv.mID
            WHERE ra.stars >= 4
            ORDER BY mv.year ASC
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

       year
    0  1937
    1  1939
    2  1981
    3  2009
    
    [4 rows x 1 columns]


### Solution using Pandas.
> Methods used : selection, merge(inner), sort, unique


    # store the ratings in new DF for ease of operation
    # we also reset the index.
    ratDFNew = ratDF[ ratDF['stars'] >= 4 ].reset_index()
    # Now merge (similar to join in SQL) the new ratingDF
    # into the movieDF.
    resDF = pandas.merge( ratDFNew, movieDF, \
                         on='mID', how='inner' )
    # Now sort according to the year
    # note here that we can sort 
    # the DF in place (using 
    # the keyword 'inplace') without
    # creating a new DF instance.
    resDF.sort( ['year'], ascending=True, inplace=True )
    # get the year column only and 
    resDF = resDF['year']
    # Print the unique values using unique() statement
    print resDF.unique()

    [1937 1939 1981 2009]


## Question-3 : Find the titles of all movies that have no ratings.

### Solution using SQL.


    #SQL query
    qry = """
            SELECT mv.title FROM Movie mv
            LEFT JOIN Rating ra
            ON ra.mID = mv.mID
            WHERE ra.mID is NULL
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

           title
    0  Star Wars
    1    Titanic
    
    [2 rows x 1 columns]


### Solution using Pandas.
> Methods used : merge(left), isnull


    # We'll merge the two DFs using
    # 'left' method. This is like the 
    # left outer join in SQL.
    resDF = pandas.merge( movieDF, ratDF, \
                         on='mID', how='left' )
    # Now retreive 'title' column from records
    #  which have a Null value in the stars column
    resDF = resDF[ resDF['stars'].isnull() ]\
    .reset_index()['title']
    print resDF

    0    Star Wars
    1      Titanic
    Name: title, dtype: object


## Question-4 : Some reviewers didn't provide a date with their rating. Find the
names of all reviewers who have ratings with a NULL value for the date.

### Solution using SQL.


    #SQL query
    qry = """
            SELECT re.name FROM Reviewer re
            INNER JOIN Rating ra
            ON ra.rID = re.rID
            WHERE ra.ratingDate IS NULL
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

                name
    0   Daniel Lewis
    1  Chris Jackson
    
    [2 rows x 1 columns]


### Solution using Pandas.
> Methods used : Methods used : merge(inner), isnull()


     # We'll merge rvwrDF and ratDF
    # and retreive rows which have Null
    # in date.
    resDF = pandas.merge( rvwrDF, ratDF, \
                         on='rID', how='inner' )
    # Now retreive the records which have null
    # value in the date column
    resDF = resDF[ resDF['ratingDate'].isnull() ]\
    .reset_index()['name']
    print resDF

    0     Daniel Lewis
    1    Chris Jackson
    Name: name, dtype: object


## Question-5 : Write a query to return the ratings data in a more readable
format: reviewer name, movie title, stars, and ratingDate. Also, sort the data,
first by reviewer name, then by movie title, and lastly by number of stars.

### Solution using SQL


    #SQL query
    qry = """
            SELECT re.name, mv.title, ra.stars, ra.ratingDate
            FROM Movie mv 
            INNER JOIN Rating ra ON ra.mID = mv.mID
            INNER JOIN Reviewer re ON ra.rID = re.rID
            ORDER BY re.name, mv.title, ra.stars
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

                    name                    title  stars  ratingDate
    0       Ashley White                     E.T.      3  2011-01-02
    1    Brittany Harris  Raiders of the Lost Ark      2  2011-01-30
    2    Brittany Harris  Raiders of the Lost Ark      4  2011-01-12
    3    Brittany Harris       The Sound of Music      2  2011-01-20
    4      Chris Jackson                     E.T.      2  2011-01-22
    5      Chris Jackson  Raiders of the Lost Ark      4        None
    6      Chris Jackson       The Sound of Music      3  2011-01-27
    7       Daniel Lewis               Snow White      4        None
    8   Elizabeth Thomas                   Avatar      3  2011-01-15
    9   Elizabeth Thomas               Snow White      5  2011-01-19
    10     James Cameron                   Avatar      5  2011-01-20
    11     Mike Anderson       Gone with the Wind      3  2011-01-09
    12    Sarah Martinez       Gone with the Wind      2  2011-01-22
    13    Sarah Martinez       Gone with the Wind      4  2011-01-27
    
    [14 rows x 4 columns]


### Solution using Pandas.
> Methods used : merge(inner), sort


    # first merge all the three DFs
    resDF = pandas.merge( movieDF, ratDF, \
                         on='mID', how='inner' )
    resDF = pandas.merge( resDF, rvwrDF,\
                         on='rID', how='inner')
    # sort the DF and select the required columns
    resDF.sort( ['name','title','stars'], inplace=True )
    resDF = resDF[ [ 'name', 'title', 'stars', 'ratingDate' ] ]\
    .reset_index(drop=True)
    print resDF

                    name                    title  stars  ratingDate
    0       Ashley White                     E.T.      3  2011-01-02
    1    Brittany Harris  Raiders of the Lost Ark      2  2011-01-30
    2    Brittany Harris  Raiders of the Lost Ark      4  2011-01-12
    3    Brittany Harris       The Sound of Music      2  2011-01-20
    4      Chris Jackson                     E.T.      2  2011-01-22
    5      Chris Jackson  Raiders of the Lost Ark      4        None
    6      Chris Jackson       The Sound of Music      3  2011-01-27
    7       Daniel Lewis               Snow White      4        None
    8   Elizabeth Thomas                   Avatar      3  2011-01-15
    9   Elizabeth Thomas               Snow White      5  2011-01-19
    10     James Cameron                   Avatar      5  2011-01-20
    11     Mike Anderson       Gone with the Wind      3  2011-01-09
    12    Sarah Martinez       Gone with the Wind      2  2011-01-22
    13    Sarah Martinez       Gone with the Wind      4  2011-01-27
    
    [14 rows x 4 columns]


## Question-6 : For all cases where the same reviewer rated the same movie twice
and gave it a higher rating the second time, return the reviewer's name and the
title of the movie.

### Solution using SQL.


    #SQL query
    qry = """
            SELECT re1.name, mv.title FROM Reviewer re1
            INNER JOIN Rating ra1 ON ra1.rID = re1.rID
            INNER JOIN Movie mv ON mv.mID = ra1.mID
            INNER JOIN Rating ra2 ON ra1.rID = ra2.rID AND ra1.mID = ra2.mID
            WHERE ra2.stars > ra1.stars AND ra2.ratingDate > ra1.ratingDate
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

                 name               title
    0  Sarah Martinez  Gone with the Wind
    
    [1 rows x 2 columns]


### Solution using Pandas.
> Methods used : selection, merge(inner,self)


    # merge ratingDF on itself
    resDF = pandas.merge( ratDF, ratDF,\
                         on=['rID','mID'], how='inner')
    # Note in joins like the self join here, Pandas 
    # renames columns with same names with _x, _y suffixes.
    # Now retreive the rows which satisfy the requirement of
    # the question, i.e., stars_y > stars_x and 
    # ratingDate_y > ratingDate_x.
    resDF = resDF[ (resDF['stars_y'] > resDF['stars_x']) & \
                  (resDF['ratingDate_y'] > resDF['ratingDate_x']) ]
    # Now merge the other DFs for required info
    resDF = pandas.merge( resDF, movieDF,\
                         on=['mID'], how='inner')
    resDF = pandas.merge( resDF, rvwrDF,\
                         on=['rID'], how='inner')
    # get the required cols
    resDF = resDF[ ['name', 'title'] ]
    print resDF

                 name               title
    0  Sarah Martinez  Gone with the Wind
    
    [1 rows x 2 columns]


## Question-7 : For each movie that has at least one rating, find the highest
number of stars that movie received. Return the movie title and number of stars.
Sort by movie title.

### Solution using SQL


    #SQL query
    qry = """
            SELECT mv.title, tab.max_stars FROM Movie mv
            INNER JOIN
            (SELECT mID, MAX(stars) max_stars FROM Rating ra
            GROUP BY mID
            HAVING COUNT(mID) > 1) tab
            ON tab.mID = mv.MID
            ORDER BY mv.title
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

                         title  max_stars
    0                   Avatar          5
    1                     E.T.          3
    2       Gone with the Wind          4
    3  Raiders of the Lost Ark          4
    4               Snow White          5
    5       The Sound of Music          3
    
    [6 rows x 2 columns]


### Solution using Pandas.
> Methods used : groupby, filter(), merge(inner), rename()


    # Do a groupby operation on ratDF
    ratGrps = ratDF.groupby( ['mID'] )
    # we'll use filter to implement a having type
    # operation in pandas, here we're using filter to
    # get all mIDs which have more than 1 rating
    ratGrps = ratGrps.filter(lambda x: len(x) > 1)
    # Now get the rows which have max values in stars
    # for the selected mIDs
    ratGrpMax = ratGrps.groupby(['mID'], sort=False)\
                 ['stars'].max()
    # Merge ratGrpMax with movieDF, before that convert
    # the series to dataframe
    ratGrpMax = pandas.DataFrame( ratGrpMax )
    # Make index as column with name 'mID' for merging
    ratGrpMax['mID'] = ratGrpMax.index
    ratGrpMax.reset_index(drop=True, inplace=True)
    # Also rename 'stars' column to 'max_stars'
    ratGrpMax.rename( columns={'stars':'max_stars'},\
                     inplace=True )
    ratGrpMax = pandas.merge( ratGrpMax, movieDF,\
                             on='mID', how='inner' )
    # select the required cols
    ratGrpMax = ratGrpMax[ [ 'title', 'max_stars' ] ].sort( 'title' )
    print ratGrpMax

                         title  max_stars
    5                   Avatar          5
    4                     E.T.          3
    0       Gone with the Wind          4
    3  Raiders of the Lost Ark          4
    1               Snow White          5
    2       The Sound of Music          3
    
    [6 rows x 2 columns]


## Question-8 : For each movie, return the title and the 'rating spread', that
is, the difference between highest and lowest ratings given to that movie. Sort
by rating spread from highest to lowest, then by movie title.

### Solution using SQL.


    #SQL query
    qry = """
            SELECT mv.title, tab.rat_spread FROM Movie mv
            INNER JOIN
            (SELECT mID, MAX(stars)-MIN(stars) rat_spread FROM Rating ra
            GROUP BY mID)  tab
            ON tab.mID = mv.MID
            ORDER BY tab.rat_spread DESC, mv.title
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

                         title  rat_spread
    0                   Avatar           2
    1       Gone with the Wind           2
    2  Raiders of the Lost Ark           2
    3                     E.T.           1
    4               Snow White           1
    5       The Sound of Music           1
    
    [6 rows x 2 columns]


### Solution using Pandas.
> Methods used : groupby, concat, rename, sort


    # Group by mID
    ratGrps = ratDF.groupby( ['mID'] )
    # Get max and min star values from groupby ops
    ratMax = ratDF.groupby(['mID'], sort=False)\
                 ['stars'].max()
    ratMin = ratDF.groupby(['mID'], sort=False)\
                 ['stars'].min()
    # rename the names of ratMin and ratMax Series
    ratMax.name = 'max_stars'
    ratMin.name = 'min_stars'
    # merge(concat) the series
    resDF = pandas.concat( [ratMax, ratMin], axis=1 )
    # set the mID col
    resDF['mID'] = resDF.index
    # reset the index
    resDF.reset_index(drop=True, inplace=True)
    # get the rating spread col
    resDF['rat_spread'] = resDF['max_stars'] - resDF['min_stars']
    # merge with movieDF and select req cols
    resDF = pandas.merge( resDF, movieDF,\
                             on='mID', how='inner' )
    resDF = resDF[ [ 'title', 'rat_spread' ] ]\
            .sort('rat_spread', ascending=False)\
            .reset_index(drop=True)
    print resDF

                         title  rat_spread
    0                   Avatar           2
    1  Raiders of the Lost Ark           2
    2       Gone with the Wind           2
    3                     E.T.           1
    4       The Sound of Music           1
    5               Snow White           1
    
    [6 rows x 2 columns]


## Question-9 : Find the difference between the average rating of movies
released before 1980 and the average rating of movies released after 1980. (Make
sure to calculate the average rating for each movie, then the average of those
averages for movies before 1980 and movies after. Don't just calculate the
overall average rating before and after 1980.)

### Solution using SQL.


    #SQL query
    qry = """
            SELECT MAX(tab2.rat_rel_date) - MIN(tab2.rat_rel_date) difference FROM
            ( SELECT AVG(avg_rat) rat_rel_date FROM
            ( SELECT Movie.mID, AVG(Rating.stars) avg_rat, 
            CASE WHEN Movie.year < 1980 THEN 'BEFORE' ELSE 'AFTER' END rel_date
             FROM Rating
            INNER JOIN Movie ON Movie.mID = Rating.mID
            GROUP BY Movie.mID ) tab
            GROUP BY tab.rel_date) tab2
          """
    # get the data
    qDF = pandas.read_sql( qry, conn )
    # print the data
    print qDF

       difference
    0    0.055567
    
    [1 rows x 1 columns]


### Solution using Pandas.
> Methods used : merge(inner), sort, groupby


    # Merge ratDF and movieDF
    resDF = pandas.merge( movieDF, ratDF, \
                         on='mID', how='inner' )
    # Now make two new DFs one with movies
    # before 1980 and the other after 1980
    beforeDF = resDF[ resDF['year'] < 1980 ]
    afterDF = resDF[ resDF['year'] >= 1980 ]
    # get average rating for each movie
    bfrGrps = beforeDF.groupby(['mID'], sort=False)\
                 ['stars'].mean()
    aftrGrps = afterDF.groupby(['mID'], sort=False)\
                 ['stars'].mean()
    # get the mean of each series and get differences
    print bfrGrps.mean() - aftrGrps.mean()

    0.0555555555556

