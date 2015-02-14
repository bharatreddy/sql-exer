> SQL exercise questions from Courseera - Introduction to Databases course
>> SQL Movie-Rating Query Exercises (extra set)

{% highlight ipy %}
In [2]: import pandas
import mysql.connector
# set up connections to the DB
conn = mysql.connector.Connect(host='localhost',user='root',\
                        password='',database='rating')
{% endhighlight %}

{% highlight ipy %}
In [1]: import pandas
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
{% endhighlight %}

## Question-1 : Find the names of all reviewers who rated Gone with the Wind.

### Solution using SQL.

{% highlight ipy %}
In [3]: #SQL query
qry = """
        SELECT DISTINCT re.name FROM Reviewer re
        INNER JOIN Rating ra ON ra.rID = re.rID
        INNER JOIN Movie mv ON ra.mID = mv.mID 
        WHERE mv.title = 'Gone with the Wind'
        ORDER BY re.name
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
             name
0   Mike Anderson
1  Sarah Martinez

[2 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner), sort, unique()

{% highlight ipy %}
In [5]: # First merge all the three DFs
resDF = pandas.merge( movieDF, ratDF, \
                     on='mID', how='inner' )
resDF = pandas.merge( resDF, rvwrDF,\
                     on='rID', how='inner')
# select the rows with title = 'Gone with the wind'
resDF = resDF[ resDF['title'] == 'Gone with the Wind' ]\
        ['name'].reset_index(drop=True)
# sort the columns
resDF.sort( ['name'] )
print resDF.unique()
[u'Mike Anderson' u'Sarah Martinez']

{% endhighlight %}

## Question-2 : For any rating where the reviewer is the same as the director of the movie, return the reviewer name, movie title, and number of stars.

### Solution using SQL.

{% highlight ipy %}
In [4]: #SQL query
qry = """
        SELECT re.name, mv.title, ra.stars FROM Movie mv
        INNER JOIN Rating ra ON mv.mID = ra.mID
        INNER JOIN Reviewer re ON ra.rID = re.rID
        WHERE mv.director = re.name
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
            name   title  stars
0  James Cameron  Avatar      5

[1 rows x 3 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner)

{% highlight ipy %}
In [6]: # First merge all the three DFs
resDF = pandas.merge( movieDF, ratDF, \
                     on='mID', how='inner' )
resDF = pandas.merge( resDF, rvwrDF,\
                     on='rID', how='inner')
# select the rows where director and reviewer are same
resDF = resDF[ resDF['director'] == resDF['name'] ]\
        .reset_index(drop=True)
resDF = resDF[ ['name', 'title', 'stars'] ]
print resDF
            name   title  stars
0  James Cameron  Avatar      5

[1 rows x 3 columns]

{% endhighlight %}

## Question-3 : Return all reviewer names and movie names together in a single list, alphabetized. (Sorting by the first name of the reviewer and first word in the title is fine; no need for special processing on last names or removing "The".)

### Solution using SQL.

{% highlight ipy %}
In [5]: #SQL query
qry = """
        SELECT name list FROM Reviewer 
        UNION
        SELECT title list FROM Movie 
        ORDER BY list
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
                       list
0              Ashley White
1                    Avatar
2           Brittany Harris
3             Chris Jackson
4              Daniel Lewis
5                      E.T.
6          Elizabeth Thomas
7        Gone with the Wind
8             James Cameron
9             Mike Anderson
10  Raiders of the Lost Ark
11           Sarah Martinez
12               Snow White
13                Star Wars
14       The Sound of Music
15                  Titanic

[16 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : concat()

{% highlight ipy %}
In [7]: # instead of merging we concatenate
# the dataframes
resDF = pandas.concat( [ rvwrDF['name'], movieDF['title'] ] )
resDF.sort()
print resDF.reset_index(drop=True)
0                Ashley White
1                      Avatar
2             Brittany Harris
3               Chris Jackson
4                Daniel Lewis
5                        E.T.
6            Elizabeth Thomas
7          Gone with the Wind
8               James Cameron
9               Mike Anderson
10    Raiders of the Lost Ark
11             Sarah Martinez
12                 Snow White
13                  Star Wars
14         The Sound of Music
15                    Titanic
dtype: object

{% endhighlight %}

## Question-4 : Find the titles of all movies not reviewed by Chris Jackson.

### Solution using SQL.

{% highlight ipy %}
In [4]: #SQL query
qry = """
        SELECT title FROM Movie
        WHERE title NOT IN ( SELECT title FROM Movie mv
        INNER JOIN Rating ra ON ra.mID = mv.mID
        INNER JOIN Reviewer re ON re.rID = ra.rID 
        WHERE re.name = 'Chris Jackson' )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
                title
0  Gone with the Wind
1           Star Wars
2             Titanic
3          Snow White
4              Avatar

[5 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> merge(inner and left), isnull()

{% highlight ipy %}
In [8]: # Merge ratDF and rvwrDF
resDF = pandas.merge( ratDF, rvwrDF,\
                     on='rID', how='inner')
# select the rows where revieewer is Chris Jackson
resDF = resDF[ resDF['name'] == 'Chris Jackson' ]
# merge(left) with movieDF to get those movies
# not rated by Chris (they will be none)
resDF = pandas.merge( movieDF, resDF,\
                     on='mID', how='left')
resDF = resDF[ resDF['rID'].isnull() ]
print resDF['title']
0    Gone with the Wind
1             Star Wars
4               Titanic
5            Snow White
6                Avatar
Name: title, dtype: object

{% endhighlight %}

## Question-5 : For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers. Eliminate duplicates, don't pair reviewers with themselves, and include each pair only once. For each pair, return the names in the pair in alphabetical order.

### Solution using SQL.

{% highlight ipy %}
In [16]: #SQL query
qry = """
        SELECT DISTINCT re1.name name1, re2.name name2 FROM Reviewer re1
        INNER JOIN Rating ra1 ON ra1.rID=re1.rID
        INNER JOIN Rating ra2 ON ra2.mID=ra1.mID AND ra2.rID != ra1.rID
        INNER JOIN Reviewer re2 ON ra2.rID=re2.rID
        WHERE re2.name > re1.name
        ORDER BY re1.name
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
              name1             name2
0      Ashley White     Chris Jackson
1   Brittany Harris     Chris Jackson
2      Daniel Lewis  Elizabeth Thomas
3  Elizabeth Thomas     James Cameron
4     Mike Anderson    Sarah Martinez

[5 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner and self), drop_duplicates()

{% highlight ipy %}
In [9]: # Merge rvwrDF with ratDF twice
# this gives us rvwvr name and stars
resDF1 = pandas.merge( rvwrDF, ratDF,\
                     on='rID', how='inner')
resDF2 = pandas.merge( rvwrDF, ratDF,\
                     on='rID', how='inner')
# Merge both resDFs with themselves
resDF = pandas.merge( resDF1, resDF2,\
                     on='mID', how='inner')
# filter for reviewer names
resDF = resDF[ resDF['name_x'] < resDF['name_y'] ]
resDF = resDF[ ['name_x','name_y'] ].sort( ['name_x'] )
# drop all duplicate rows
print resDF.drop_duplicates()
              name_x            name_y
28      Ashley White     Chris Jackson
14   Brittany Harris     Chris Jackson
10      Daniel Lewis  Elizabeth Thomas
31  Elizabeth Thomas     James Cameron
6      Mike Anderson    Sarah Martinez

[5 rows x 2 columns]

{% endhighlight %}

## Question-6 : For each rating that is the lowest (fewest stars) currently in the database, return the reviewer name, movie title, and number of stars.

### Solution using SQL.

{% highlight ipy %}
In [8]: #SQL query
qry = """
        SELECT re.name, mv.title, ra.stars FROM Reviewer re
        INNER JOIN Rating ra ON ra.rID = re.rID
        INNER JOIN Movie mv ON ra.mID = mv.mID
        WHERE ra.stars = ( SELECT MIN(stars) FROM Rating )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
              name                    title  stars
0   Sarah Martinez       Gone with the Wind      2
1  Brittany Harris       The Sound of Music      2
2  Brittany Harris  Raiders of the Lost Ark      2
3    Chris Jackson                     E.T.      2

[4 rows x 3 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner)

{% highlight ipy %}
In [10]: # get the min star rating
minStars = ratDF['stars'].min()
# combine all three DFs
resDF = pandas.merge( movieDF, ratDF, \
                     on='mID', how='inner' )
resDF = pandas.merge( resDF, rvwrDF,\
                     on='rID', how='inner')
# get rows which have min stars
resDF = resDF[ resDF['stars'] == minStars ].\
        reset_index(drop=True)
# get the cols
resDF = resDF[ [ 'name', 'title', 'stars'] ]
print resDF
              name                    title  stars
0   Sarah Martinez       Gone with the Wind      2
1  Brittany Harris       The Sound of Music      2
2  Brittany Harris  Raiders of the Lost Ark      2
3    Chris Jackson                     E.T.      2

[4 rows x 3 columns]

{% endhighlight %}

## Question-7 : List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order.

### Solution using SQL.

{% highlight ipy %}
In [10]: #SQL query
qry = """
        SELECT mv.title, AVG(ra.stars) as avg_stars FROM Movie mv
        INNER JOIN Rating ra ON mv.mID = ra.mID
        GROUP BY ra.mID
        ORDER BY avg_stars DESC, mv.title
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
                     title  avg_stars
0               Snow White     4.5000
1                   Avatar     4.0000
2  Raiders of the Lost Ark     3.3333
3       Gone with the Wind     3.0000
4                     E.T.     2.5000
5       The Sound of Music     2.5000

[6 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner), groupby, sort

{% highlight ipy %}
In [11]: # combine movieDF and ratDF
resDF = pandas.merge( movieDF, ratDF, \
                     on='mID', how='inner' )
# keep the required cols
resDF = resDF[ ['title', 'stars'] ]
# group by title
titleGrps = resDF.groupby( ['title'] )
# get the average star rating of the groups
avgRats = titleGrps['stars'].mean()
# sorting on multiple columns is easier with DFs
# in other words, I couldn't find a simpler way to
# sort the series on ratings(values) and index
# the order() function of pandas series didn't work either
avgRats = pandas.DataFrame(avgRats)
# make the index as a col
avgRats['title'] = avgRats.index
avgRats.reset_index(drop=True, inplace=True)
# Now sort on stars and title
avgRats.sort( ['stars', 'title'], \
             ascending=[ False, True ],\
             inplace=True )
print avgRats
      stars                    title
4  4.500000               Snow White
0  4.000000                   Avatar
3  3.333333  Raiders of the Lost Ark
2  3.000000       Gone with the Wind
1  2.500000                     E.T.
5  2.500000       The Sound of Music

[6 rows x 2 columns]

{% endhighlight %}

## Question-8 : Find the names of all reviewers who have contributed three or more ratings.

### Solution using SQL.

{% highlight ipy %}
In [11]: #SQL query
qry = """
        SELECT re.name FROM Reviewer re
        INNER JOIN Rating ra ON ra.rID=re.rID
        GROUP BY ra.rID
        HAVING COUNT(ra.rID) >= 3
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
              name
0  Brittany Harris
1    Chris Jackson

[2 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner), groupby, filter, drop_duplicates()

{% highlight ipy %}
In [13]: # combine rvwrDF and ratDF
resDF = pandas.merge( rvwrDF, ratDF, \
                     on='rID', how='inner' )
# keep the required cols
resDF = resDF[ ['name', 'stars'] ]
# group by title
nameGrps = resDF.groupby( ['name'] )
# get only those rows which have more than 3 ratings
# we'll use filter to implement having statement of SQL
nameGrps = nameGrps.filter(lambda x: len(x) >= 3)
nameGrps = nameGrps['name']
# drop duplicate rows
print nameGrps.drop_duplicates()\
        .reset_index(drop=True)
0    Brittany Harris
1      Chris Jackson
Name: name, dtype: object

{% endhighlight %}

## Question-9 : Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. Sort by director name, then movie title.

### Solution using SQL.

{% highlight ipy %}
In [12]: #SQL query
qry = """
        SELECT title, director FROM Movie
        WHERE director IN (
        SELECT director FROM Movie
        GROUP BY director 
        HAVING COUNT(title) > 1)
        ORDER BY director, title
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
                     title          director
0                   Avatar     James Cameron
1                  Titanic     James Cameron
2                     E.T.  Steven Spielberg
3  Raiders of the Lost Ark  Steven Spielberg

[4 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : groupby, filter, sort

{% highlight ipy %}
In [15]: # Methods used : groupby, filter, sort
# groupby directors
dirGrps = movieDF.groupby( ['director'] )
# get those directors who directed > 1 movie
dirGrps = dirGrps.filter(lambda x: len(x) > 1)
# get the required cols
dirGrps = dirGrps[ ['title', 'director'] ]
# sort by director, movie
dirGrps.sort( ['director', 'title'],\
             inplace=True )
print dirGrps.reset_index(drop=True)
                     title          director
0                   Avatar     James Cameron
1                  Titanic     James Cameron
2                     E.T.  Steven Spielberg
3  Raiders of the Lost Ark  Steven Spielberg

[4 rows x 2 columns]

{% endhighlight %}

## Question-10 : Find the movie(s) with the highest average rating. Return the movie title(s) and average rating.

### Solution using SQL.

{% highlight ipy %}
In [13]: #SQL query
qry = """
        SELECT mv.title, AVG(stars) avg_stars FROM Rating ra
        INNER JOIN Movie mv
        ON mv.mID=ra.mID
        GROUP BY ra.mID
        HAVING AVG(stars) = 
        (SELECT AVG(stars) max_avg_rat FROM Rating
        GROUP BY mID
        ORDER BY max_avg_rat DESC 
        LIMIT 1 )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
        title  avg_stars
0  Snow White        4.5

[1 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner), groupby, reset_index(using level option)

{% highlight ipy %}
In [18]: # Merge ratDF and movieDF
resDF = pandas.merge( movieDF, ratDF,\
                     on='mID', how='inner')
# groupby ratDF by mID to get avg rating
mvGrps = resDF.groupby( ['mID', 'title'] )
# get average ratings
avgStars = mvGrps['stars'].mean()
# get max value of the average ratings
avgMaxStars = mvGrps['stars'].mean().max()
# get mIDs of movies which have max avg ratings
avgStars = avgStars[ avgStars == avgMaxStars ]
# print title and level
print avgStars.reset_index(drop=True,level='mID')
title
Snow White    4.5
Name: stars, dtype: float64

{% endhighlight %}

## Question-11 : Find the movie(s) with the lowest average rating. Return the movie title(s) and average rating.

### Solution using SQL.

{% highlight ipy %}
In [14]: #SQL query
qry = """
        SELECT mv.title, AVG(stars) avg_stars FROM Rating ra
        INNER JOIN Movie mv
        ON mv.mID=ra.mID
        GROUP BY ra.mID
        HAVING AVG(stars) = 
        (SELECT AVG(stars) max_avg_rat FROM Rating
        GROUP BY mID
        ORDER BY max_avg_rat ASC 
        LIMIT 1 )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
                title  avg_stars
0  The Sound of Music        2.5
1                E.T.        2.5

[2 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner), groupby, reset_index(using level option)

{% highlight ipy %}
In [16]: # Methods used : merge(inner), groupby, reset_index(using level option)
# Merge ratDF and movieDF
resDF = pandas.merge( movieDF, ratDF,\
                     on='mID', how='inner')
# groupby ratDF by mID to get avg rating
mvGrps = resDF.groupby( ['mID', 'title'] )
# get average ratings
avgStars = mvGrps['stars'].mean()
# get max value of the average ratings
avgMinStars = mvGrps['stars'].mean().min()
# get mIDs of movies which have max avg ratings
avgStars = avgStars[ avgStars == avgMinStars ]
# print title and level
print avgStars.reset_index(drop=True,level='mID')
title
The Sound of Music    2.5
E.T.                  2.5
Name: stars, dtype: float64

{% endhighlight %}

## Question-12 : For each director, return the director's name together with the title(s) of the movie(s) they directed that received the highest rating among all of their movies, and the value of that rating. Ignore movies whose director is NULL.

### Solution using SQL.

{% highlight ipy %}
In [15]: #SQL query
qry = """
        SELECT mv.director, mv.title, MAX(ra.stars) FROM Movie mv
        INNER JOIN Rating ra ON ra.mID=mv.mID AND mv.director IS NOT NULL
        GROUP BY mv.director
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
           director                    title  MAX(ra.stars)
0     James Cameron                   Avatar              5
1       Robert Wise       The Sound of Music              3
2  Steven Spielberg  Raiders of the Lost Ark              4
3    Victor Fleming       Gone with the Wind              4

[4 rows x 3 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner), groupby, dropna

{% highlight ipy %}
In [17]: # Merge ratDF and movieDF
resDF = pandas.merge( movieDF, ratDF,\
                     on='mID', how='inner')
# drop NaN vals
resDF.dropna(inplace=True)
# groupby ratDF by mID to get avg rating
mvGrps = resDF.groupby( ['director'] )
# get max value of the average ratings
maxRatings = mvGrps['title', 'stars'].max()
print maxRatings
                                    title  stars
director                                        
James Cameron                      Avatar      5
Robert Wise            The Sound of Music      3
Steven Spielberg  Raiders of the Lost Ark      4
Victor Fleming         Gone with the Wind      4

[4 rows x 2 columns]

{% endhighlight %}

