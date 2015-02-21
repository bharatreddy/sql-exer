> SQL exercise questions from Courseera - Introduction to Databases course
>> SQL Social-Network Query Exercises (extra set)

{% highlight ipy %}
In [8]: import pandas
import mysql.connector
# set up connections to the DB
conn = mysql.connector.Connect(host='localhost',user='root',\
                        password='',database='social')
{% endhighlight %}

{% highlight ipy %}
In [9]: import pandas
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

## Question-1 : For every situation where student A likes student B, but student B likes a different student C, return the names and grades of A, B, and C.

### Solution using SQL.

{% highlight ipy %}
In [10]: #SQL query
qry = """
        SELECT hs1.name, hs1.grade, hs2.name, hs2.grade, hs3.name, hs3.grade
        FROM Highschooler hs1
        INNER JOIN Likes l1 ON l1. ID1 = hs1.ID
        INNER JOIN Likes l2 ON l1.ID2=l2.ID1 AND l1.ID1 != l2.ID2
        INNER JOIN Highschooler hs2 ON hs2.ID=l2.ID1
        INNER JOIN Highschooler hs3 ON hs3.ID=l2.ID2
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
      name  grade       name  grade     name  grade
0   Andrew     10  Cassandra      9  Gabriel      9
1  Gabriel     11     Alexis     11     Kris     10

[2 rows x 6 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge(inner, using left_on, right_on), drop()

{% highlight ipy %}
In [11]: # Merge likesDF on itself
resDF = pandas.merge( likesDF, likesDF, \
                     left_on = 'ID2',\
                     right_on = 'ID1', how='inner')
# Now take out the ID's who mutually like each other
resDF = resDF[ resDF['ID1_x'] != resDF['ID2_y'] ]
# drop unnecessary columns and merge resDF with hsDF
# to get the name and grade details from all the IDs
resDF.drop( 'ID1_y', 1, inplace=True )
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID1_x',\
                     right_on = 'ID', how='inner')
resDF.drop( ['ID1_x', 'ID'], 1, inplace=True )
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2_x',\
                     right_on = 'ID', how='inner')
resDF.drop( ['ID2_x', 'ID'], 1, inplace=True )
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2_y',\
                     right_on = 'ID', how='inner')
resDF.drop( ['ID2_y', 'ID'], 1, inplace=True )
print resDF
    name_x  grade_x     name_y  grade_y     name  grade
0   Andrew       10  Cassandra        9  Gabriel      9
1  Gabriel       11     Alexis       11     Kris     10

[2 rows x 6 columns]

{% endhighlight %}

## Question-2 : Find those students for whom all of their friends are in different grades from themselves. Return the students' names and grades.

### Solution using SQL.

{% highlight ipy %}
In [12]: #SQL query
qry = """
        SELECT name, grade FROM Highschooler WHERE ID NOT IN
        (SELECT hs1.ID FROM Highschooler hs1
        INNER JOIN Friend f1 ON f1.ID1=hs1.ID
        INNER JOIN Highschooler hs2 ON f1.ID2=hs2.ID
        WHERE hs1.grade-hs2.grade = 0 )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
     name  grade
0  Austin     11

[1 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : merge, drop(), tolist(), isin()

{% highlight ipy %}
In [13]: # Merge hsDF with frndDF
resDF = pandas.merge( hsDF, frndDF, \
                     left_on = 'ID',\
                     right_on = 'ID1', how='inner')
resDF = pandas.merge( resDF, hsDF, \
                     left_on = 'ID2',\
                     right_on = 'ID', how='inner')
resDF.drop( ['ID1', 'ID2'], 1, inplace=True )
# select all students who have frnds in same grade
resDF = resDF[ resDF['grade_x'] - resDF['grade_y'] == 0 ]
# get a list of their IDs
idList = resDF['ID_x'].tolist()
# get the name and grades of students 
# not in the list from hsDF
resDF = hsDF[ ~hsDF['ID'].isin(idList) ]
print resDF[ ['name', 'grade'] ].reset_index(drop=True)
     name  grade
0  Austin     11

[1 rows x 2 columns]

{% endhighlight %}

### Question-3 : What is the average number of friends per student? (Your result should be just one number.) 

### Solution using SQL.

{% highlight ipy %}
In [14]: #SQL query
qry = """
        SELECT AVG(T.cnt_by_frnd) FROM ( SELECT ID1, COUNT(ID2) cnt_by_frnd FROM Friend
        GROUP BY ID1 ) T
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
   AVG(T.cnt_by_frnd)
0                 2.5

[1 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : groupby(), count(), mean()

{% highlight ipy %}
In [15]: # groupby frndDF on ID1 and get counts of ID2
frndGrps = frndDF.groupby( ['ID1'] )
# get number of frnds for each student
nFrnds = frndGrps['ID2'].count()
print nFrnds.mean()
2.5

{% endhighlight %}

## Question-4 : Find the number of students who are either friends with Cassandra or are friends of friends of Cassandra. Do not count Cassandra, even though technically she is a friend of a friend. 

### Solution using SQL.

{% highlight ipy %}
In [16]: #SQL query
qry = """
        SELECT COUNT(ID2) FROM Friend WHERE ID1 IN ( SELECT ID2 FROM Friend 
        WHERE ID2 != ( SELECT ID FROM Highschooler 
        WHERE name = 'Cassandra' ) 
        AND (ID1 = ( SELECT ID FROM Highschooler 
        WHERE name = 'Cassandra' ) ) 
        OR ID1 =  ( SELECT ID FROM Highschooler 
        WHERE name = 'Cassandra' ) )
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
   COUNT(ID2)
0           7

[1 rows x 1 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : tolist()

{% highlight ipy %}
In [17]: # Get the ID of 'Cassandra'
idCsn = hsDF[ hsDF['name'] == 'Cassandra' ]\
        ['ID'].values[0]
# get all friends of this particular ID
frndList = frndDF[ frndDF['ID1'] == idCsn ]\
            ['ID2'].tolist()
# get friends of friends as well and make 
# it a set to get unique values
frndList = set( frndList + frndDF\
        [ ( frndDF['ID1'].isin(frndList) )\
         & (frndDF['ID2'] != idCsn) ]['ID2'].\
        tolist() )
print len(frndList)
7

{% endhighlight %}

## Question-5 : Find the name and grade of the student(s) with the greatest number of friends. 

### Solution using SQL.

{% highlight ipy %}
In [18]: #SQL query
qry = """
        SELECT name, grade FROM Highschooler
        INNER JOIN ( 
        SELECT ID1 FROM Friend
        GROUP BY ID1
        HAVING COUNT(ID1) =
        ( SELECT COUNT(ID2) FROM Friend
        GROUP BY ID1
        ORDER BY COUNT(ID2) DESC 
        LIMIT 1 )
        ) T
        ON ID = T.ID1
      """
# get the data
qDF = pandas.read_sql( qry, conn )
# print the data
print qDF
     name  grade
0  Andrew     10
1  Alexis     11

[2 rows x 2 columns]

{% endhighlight %}

### Solution using Pandas.
> Methods used : groupby(), count(), isin()

{% highlight ipy %}
In [19]: # Groupby ID1 and get max frnd number
frndGrps = frndDF.groupby( 'ID1' )
maxFrnds = frndGrps['ID2'].count().max()
# get ID with max frnds
countIdSer = frndGrps['ID2'].count()
idMaxFrnd = countIdSer[ countIdSer == maxFrnds ]
# get name, grade for the IDs
resDF = hsDF[ hsDF['ID'].isin(idMaxFrnd.index.values) ]\
        [ ['name','grade'] ]
print resDF.reset_index(drop=True)
     name  grade
0  Andrew     10
1  Alexis     11

[2 rows x 2 columns]

{% endhighlight %}

