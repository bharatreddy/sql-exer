> SQL exercise questions from Courseera - Introduction to Databases course
>> SQL Social-Network Query Exercises (core set)

{% highlight ipy %}
In [13]: import pandas
import mysql.connector
# set up connections to the DB
conn = mysql.connector.Connect(host='localhost',user='root',\
                        password='',database='social')
{% endhighlight %}

{% highlight ipy %}
In [14]: import pandas
import mysql.connector
# set up connections to the DB
conn = mysql.connector.Connect(host='localhost',user='root',\
                        password='',database='social')
#LOAD the SQL tables into DF
qryFr = """
        SELECT * From Friend
      """
qryHs = """
        SELECT * From Highschooler
      """
qryLi = """
        SELECT * From Likes
      """
frndDF = pandas.read_sql( qryFr, conn )
hsDF = pandas.read_sql( qryHs, conn )
likesDF = pandas.read_sql( qryLi, conn )
{% endhighlight %}

## Question-1 : Find the names of all students who are friends with someone named Gabriel.

### Solution using SQL.

{% highlight ipy %}
In [15]: #SQL query
qry = """
        SELECT hs1.name FROM Highschooler hs1
        INNER JOIN Friend fr1 ON fr1.ID1=hs1.ID
        INNER JOIN Highschooler hs2 ON hs2.ID=fr1.ID2 
        WHERE hs2.name='Gabriel'
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
        name
0     Jordan
1     Alexis
2  Cassandra
3     Andrew
4    Jessica

[5 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner, using left_on, right_on), tolist(), isin()

{% highlight ipy %}
In [16]: # First merge the ID col of hsDF with ID1 of frndDF
resDF = pandas.merge( hsDF, frndDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner')
# select the 'ID2' cols for rows which have name='Gabriel'
frndIDs = resDF[ resDF['name'] == 'Gabriel' ]['ID2']\
            .tolist()
# Now from hsDF get the names of frndIDs
frndNames = hsDF[ hsDF['ID'].isin(frndIDs) ]
print frndNames['name']
#SQL query
0        Jordan
3     Cassandra
5        Andrew
8        Alexis
11      Jessica
Name: name, dtype: object

{% endhighlight %}

## Question-2 : For every student who likes someone 2 or more grades younger than themselves, return that student's name and grade, and the name and grade of the student they like. 

### Solution using SQL.

{% highlight ipy %}
In [17]: #SQL query
qry = """
        SELECT hs1.name, hs1.grade, hs2.name, hs2.grade FROM Likes l1
        INNER JOIN Highschooler hs1 ON l1.ID1=hs1.ID
        INNER JOIN Highschooler hs2 ON l1.ID2=hs2.ID
        WHERE hs1.grade-hs2.grade>=2
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
   name  grade   name  grade
0  John     12  Haley     10

[1 rows x 4 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner, using left_on and right_on)

{% highlight ipy %}
In [18]: # First merge the ID col of hsDF with ID1 of likesDF
resDF = pandas.merge( hsDF, likesDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner')
# Now merge hsDF again with resDF on ID2
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner')
# Now select rows which satisfy greater than 2 grades condition
resDF = resDF[ resDF['grade_x'] - resDF['grade_y'] >= 2 ].\
        reset_index(drop=True)
# select the required cols
resDF = resDF[ [ 'name_x', 'grade_x', 'name_y', 'grade_y' ] ]
print resDF
  name_x  grade_x name_y  grade_y
0   John       12  Haley       10

[1 rows x 4 columns]

{% endhighlight %}

## Question-3 : For every pair of students who both like each other, return the name and grade of both students. Include each pair only once, with the two names in alphabetical order.

### Solution using SQL.

{% highlight ipy %}
In [19]: #SQL query
qry = """
        SELECT hs1.name, hs1.grade, hs2.name, hs2.grade FROM Likes l1
        INNER JOIN Likes l2 ON l1.ID1=l2.ID2 AND l1.ID2=l2.ID1
        INNER JOIN Highschooler hs1 ON hs1.ID=l1.ID1
        INNER JOIN Highschooler hs2 ON hs2.ID=l2.ID1
        WHERE hs1.name < hs2.name
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
        name  grade     name  grade
0  Cassandra      9  Gabriel      9
1    Jessica     11     Kyle     12

[2 rows x 4 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner, self, using left_on and right_on)

{% highlight ipy %}
In [20]: # First merge likesDF with itself
resDF = pandas.merge( likesDF, likesDF, \
                     left_on = 'ID1',\
                     right_on = 'ID2', how='inner' )
# select the rows where ID2_x = ID1_y
resDF = resDF[ resDF['ID2_x'] == resDF['ID1_y'] ].\
        reset_index(drop=True)
# merge hsDF with resDF on ID1_x and also with ID2_x
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID1_x',\
                     right_on = 'ID', how='inner' )
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2_x',\
                     right_on = 'ID', how='inner' )
# get name_x, name_y cols from rows where name_x < name_y
resDF = resDF[ resDF['name_x'] < resDF['name_y'] ]\
        [ ['name_x', 'grade_x','name_y', 'grade_y'] ]
print resDF
      name_x  grade_x   name_y  grade_y
1  Cassandra        9  Gabriel        9
2    Jessica       11     Kyle       12

[2 rows x 4 columns]

{% endhighlight %}

## Question-4 : Find all students who do not appear in the Likes table (as a student who likes or is liked) and return their names and grades. Sort by grade, then by name within each grade.

### Solution using SQL.

{% highlight ipy %}
In [21]: #SQL query
qry = """
        SELECT hs.name, hs.grade FROM Highschooler hs
        LEFT JOIN Likes l1 ON l1.ID1=hs.ID OR l1.ID2=hs.ID
        WHERE l1.ID1 IS NULL AND l1.ID2 IS NULL
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
      name  grade
0   Jordan      9
1  Tiffany      9
2    Logan     12

[3 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : tolist(), isin()

{% highlight ipy %}
In [22]: # Get a unique list of all IDs (both ID1 and ID2) in likesDF
likeIds = set( likesDF['ID1'].tolist() + likesDF['ID2'].tolist() )
# get names/grades from hsDF
resDF = hsDF[ ~hsDF['ID'].isin(likeIds) ][['name', 'grade']].\
        reset_index(drop=True)
print resDF
      name  grade
0   Jordan      9
1  Tiffany      9
2    Logan     12

[3 rows x 2 columns]

{% endhighlight %}

## Question-5 : For every situation where student A likes student B, but we have no information about whom B likes (that is, B does not appear as an ID1 in the Likes table), return A and B's names and grades. 

### Solution using SQL.

{% highlight ipy %}
In [23]: #SQL query
qry = """
        SELECT hs1.name, hs1.grade, hs2.name, hs2.grade 
        FROM Highschooler hs1 
        INNER JOIN Likes l1 ON l1.ID1=hs1.ID
        INNER JOIN Highschooler hs2 ON l1.ID2=hs2.ID
        WHERE hs2.ID NOT IN ( SELECT ID1 FROM Likes )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
       name  grade    name  grade
0      John     12   Haley     10
1  Brittany     10    Kris     10
2    Alexis     11    Kris     10
3    Austin     11  Jordan     12

[4 rows x 4 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : tolist(), merge(inner, using left_on and right_on)

{% highlight ipy %}
In [24]: # Get list of all ID1 from likesDF
likesId1 = set( likesDF['ID1'].tolist() )
# merge likesDF with hsDF on ID1
resDF = pandas.merge( hsDF, likesDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner' )
# Now merge resDF with hsDF on ID2
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner' )
# Now get all rows which dont have likesId1 in ID2 col
resDF = resDF[ ~resDF['ID2'].isin(likesId1) ].\
        reset_index(drop=True)
# filter required cols
resDF = resDF[ ['name_x', 'grade_x', 'name_y', 'grade_y' ] ]
print resDF
     name_x  grade_x  name_y  grade_y
0  Brittany       10    Kris       10
1    Alexis       11    Kris       10
2    Austin       11  Jordan       12
3      John       12   Haley       10

[4 rows x 4 columns]

{% endhighlight %}

## Question-6 : Find names and grades of students who only have friends in the same grade. Return the result sorted by grade, then by name within each grade. 

### Solution using SQL.

{% highlight ipy %}
In [25]: #SQL query
qry = """
        SELECT name, grade FROM Highschooler WHERE ID NOT IN
        (SELECT hs1.ID FROM Highschooler hs1
        INNER JOIN Friend fr ON fr.ID1=hs1.ID
        INNER JOIN Highschooler hs2 ON hs2.ID=fr.ID2
        WHERE hs2.grade-hs1.grade != 0 )
        ORDER BY grade, name
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
       name  grade
0    Jordan      9
1  Brittany     10
2     Haley     10
3      Kris     10
4   Gabriel     11
5      John     12
6     Logan     12

[7 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : tolist(), merge(), isin(), drop_duplicates()

{% highlight ipy %}
In [26]: # Merge frndDF with hsDF on ID1 and ID2 seperately
resDF = pandas.merge( hsDF, frndDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner' )
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner' )
# select the rows where friends are in different grades
id1List = resDF[ resDF['grade_x'] - resDF['grade_y'] != 0 ]['ID1']
# Now select rows which are not in rowNums
# also filter the cols
resDF = resDF[ ~resDF['ID1'].isin(id1List) ]\
        [ ['name_x', 'grade_x'] ]\
    .drop_duplicates()\
    .reset_index(drop=True)
    
print resDF.sort( ['grade_x', 'name_x'] )
     name_x  grade_x
0    Jordan        9
4  Brittany       10
3     Haley       10
1      Kris       10
2   Gabriel       11
6      John       12
5     Logan       12

[7 rows x 2 columns]

{% endhighlight %}

## Question-7 : For each student A who likes a student B where the two are not friends, find if they have a friend C in common (who can introduce them!). For all such trios, return the name and grade of A, B, and C. 

### Solution using SQL.

{% highlight ipy %}
In [27]: #SQL query
qry = """
        SELECT hs1.name, hs1.grade, hs3.name, hs3.grade , hs2.name, hs2.grade
        FROM Highschooler hs1
        INNER JOIN Likes l1 ON l1.ID1=hs1.ID
        LEFT JOIN Friend fl ON fl.ID1=l1.ID1 AND fl.ID2=l1.ID2
        INNER JOIN Friend f1 ON f1.ID1=l1.ID1
        INNER JOIN Friend f2 ON f2.ID1=l1.ID2 AND f2.ID2=f1.ID2
        INNER JOIN Highschooler hs2 ON hs2.ID = f2.ID2
        INNER JOIN Highschooler hs3 ON hs3.ID = f2.ID1
        WHERE fl.ID2 IS NULL
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
     name  grade       name  grade     name  grade
0  Andrew     10  Cassandra      9  Gabriel      9
1  Austin     11     Jordan     12   Andrew     10
2  Austin     11     Jordan     12     Kyle     12

[3 rows x 6 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge()

{% highlight ipy %}
In [28]: # Merge frndDF with hsDF on ID1 and ID2 seperately
resFrndDF = pandas.merge( hsDF, frndDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner' )
resFrndDF = pandas.merge( resFrndDF, hsDF, \
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner' )
# Merge likesDF with hsDF on ID1 and ID2 seperately
resLikesDF = pandas.merge( hsDF, likesDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner' )
resLikesDF = pandas.merge( resLikesDF, hsDF, \
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner' )
# Merge the resLikesDF, resFrndDF on ID1, ID2
resDF = pandas.merge( resLikesDF, resFrndDF, \
                     left_on = ['ID1', 'ID2'],\
                     right_on = ['ID1', 'ID2'], how='left' )
# Get the rows with Null values in ID_x_y
# Merge resDF with frndDF
resDF = resDF[ resDF['ID_x_y'].isnull() ].\
        reset_index(drop=True)
# Now we have a DF containing students in like table
# but not are not in frnds table.
# drop un-necessary cols
resDF = resDF[ ['ID_x_x', 'name_x_x', \
                'grade_x_x', 'ID_y_x', 'name_y_x', \
                'grade_y_x'] ]
# Make a new DF merging resDF with Frnd on ID1, ID_x_x
resDFIdx = pandas.merge( resDF, frndDF,\
                     left_on = 'ID_x_x',\
                     right_on = 'ID1', how='inner' )
# Make another DF merging resDF with Frnd on ID1, ID_y_x
resDFIdy = pandas.merge( resDF, frndDF,\
                     left_on = 'ID_y_x',\
                     right_on = 'ID1', how='inner' )
# Merge the new DF's on ID2
resDF = pandas.merge( resDFIdx, resDFIdy,\
                     on = 'ID2', how='inner' )
# to actually get the names of common friends
# merge resDF with hsDF ( on ID2 )
resDF = pandas.merge( resDF, hsDF,\
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner' )
# filter the results
resDF = resDF[ (resDF['name_x_x_x'] == resDF['name_x_x_y']) & \
              (resDF['name_y_x_x'] == resDF['name_y_x_y']) ]
# get the req cols
resDF = resDF[ ['name_x_x_x', 'grade_x_x_x',\
                 'name_y_x_x', 'grade_y_x_x',\
                'name', 'grade'] ]
print resDF
  name_x_x_x  grade_x_x_x name_y_x_x  grade_y_x_x     name  grade
1     Andrew           10  Cassandra            9  Gabriel      9
2     Austin           11     Jordan           12     Kyle     12
4     Austin           11     Jordan           12   Andrew     10

[3 rows x 6 columns]

{% endhighlight %}

## Question-8 : Find the difference between the number of students in the school and the number of different first names. 

### Solution using SQL.

{% highlight ipy %}
In [29]: #SQL query
qry = """
        SELECT COUNT(DISTINCT ID)-COUNT(DISTINCT name) FROM Highschooler
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
   COUNT(DISTINCT ID)-COUNT(DISTINCT name)
0                                        2

[1 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : nunique()

{% highlight ipy %}
In [30]: # We can use nunique function on columns
print hsDF['ID'].nunique() - hsDF['name'].nunique()
2

{% endhighlight %}

## Question-9 : Find the name and grade of all students who are liked by more than one other student.

### Solution using SQL.

{% highlight ipy %}
In [31]: #SQL query
qry = """
        SELECT name, grade FROM Highschooler 
        WHERE ID IN
        ( SELECT ID2 FROM Likes 
        GROUP BY ID2
        HAVING COUNT(ID1) > 1 )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
        name  grade
0  Cassandra      9
1       Kris     10

[2 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : groupby(), count(), isin()

{% highlight ipy %}
In [32]: # Groupby likesDF
likeGroups = likesDF.groupby( 'ID2' )
# get the num of likes for each person
nLikes = likeGroups['ID1'].count()
# get ID's where count > 1
nLikes = nLikes[ nLikes > 1 ]
# get the index values from nLikes
# then get corresponding details from hsDF 
# about the ID
idList = nLikes.index.values
resDF = hsDF[ hsDF['ID'].isin(idList) ]
print resDF[ ['name', 'grade'] ].reset_index(drop=True)
        name  grade
0  Cassandra      9
1       Kris     10

[2 rows x 2 columns]

{% endhighlight %}

