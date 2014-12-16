# Adding Java 8 Lambda goodness to JDBC #

source: dotneverland.blogspot.fi/2013/11/adding-java-8-lambda-goodness-to-jdbc.html

Data access, specifically SQL access from within Java has never been nice. 
This is in large part due to the fact that the JDBC api has a lot of ceremony.

Java 7 vastly improved things with ARM blocks by taking away a lot of the ceremony 
around managing database objects such as Statements and ResultSets but fundamentally 
the code flow is still the same.

Java 8 Lambdas gives us a very nice tool for improving the flow of JDBC.

Out first attempt at improving things here is very simply to make it easy to work 
with a java.sql.ResultSet.

Here we simply wrap the ResultSet iteration and then delegate it to Lambda function.

This is very similar in concept to Spring's JDBCTemplate.

NOTE: I've released All the code snippets you see here under an Apache 2.0 
license on Github.

First we create a functional interface called ResultSetProcessor as follows:

@FunctionalInterface
public interface ResultSetProcessor {

    public void process(ResultSet resultSet, 
                        long currentRow) 
                        throws SQLException;

}

Very straightforward. This interface takes the ResultSet and the current row of the ResultSet  
as a parameter.

Next we write a simple utility to which executes a query and then calls our ResultSetProcessor 
each time we iterate over the ResultSet:

```java
public static void select(Connection connection, 
                          String sql, 
                          ResultSetProcessor processor, 
                          Object... params) {
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            int cnt = 0;
            for (Object param : params) {
                ps.setObject(++cnt, param));
            }
            try (ResultSet rs = ps.executeQuery()) {
                long rowCnt = 0;
                while (rs.next()) {
                    processor.process(rs, rowCnt++);
                }
            } catch (SQLException e) {
                throw new DataAccessException(e);
            }
        } catch (SQLException e) {
            throw new DataAccessException(e);
        }
}
```

Note I've wrapped the SQLException in my own unchecked DataAccessException.

Now when we write a query it's as simple as calling the select method with 
a connection and a query:

```java
select(connection, 
       "select * from MY_TABLE",
       (rs, cnt)-> {System.out.println(rs.getInt(1)+" "+cnt)});
```

So that's great but I think we can do more...

One of the nifty Lambda additions in Java is the new Streams API. 
This would allow us to add very powerful functionality with which to process a ResultSet.

Using the Streams API over a ResultSet however creates a bit more of a challenge 
than the simple select with Lambda in the previous example.

The way I decided to go about this is create my own Tuple type which represents a 
single row from a ResultSet.

My Tuple here is the relational version where a Tuple is a collection of elements 
where each element is identified by an attribute, basically a collection of key value pairs. 
In our case the Tuple is ordered in terms of the order of the columns in the ResultSet.

The code for the Tuple ended up being quite a bit so if you want to take a look, 
see the GitHub project in the resources at the end of the post.

Currently the Java 8 API provides the java.util.stream.StreamSupport object which provides 
a set of static methods for creating instances of java.util.stream.Stream. 
We can use this object to create an instance of a Stream.

But in order to create a Stream it needs an instance of java.util.stream.Spliterator. 
This is a specialised type for iterating and partitioning a sequence of elements, 
the Stream needs for handling operations in parallel.

Fortunately the Java 8 api also provides the java.util.stream.Spliterators class 
which can wrap existing Collection and enumeration types. One of those types being 
a java.util.Iterator.

Now we wrap a query and ResultSet in an Iterator:

```java
public class ResultSetIterator implements Iterator {

    private ResultSet rs;
    private PreparedStatement ps;
    private Connection connection;
    private String sql;

    public ResultSetIterator(Connection connection, String sql) {
        assert connection != null;
        assert sql != null;
        this.connection = connection;
        this.sql = sql;
    }

    public void init() {
        try {
            ps = connection.prepareStatement(sql);
            rs = ps.executeQuery();

        } catch (SQLException e) {
            close();
            throw new DataAccessException(e);
        }
    }

    @Override
    public boolean hasNext() {
        if (ps == null) {
            init();
        }
        try {
            boolean hasMore = rs.next();
            if (!hasMore) {
                close();
            }
            return hasMore;
        } catch (SQLException e) {
            close();
            throw new DataAccessException(e);
        }

    }

    private void close() {
        try {
            rs.close();
            try {
                ps.close();
            } catch (SQLException e) {
                //nothing we can do here
            }
        } catch (SQLException e) {
            //nothing we can do here
        }
    }

    @Override
    public Tuple next() {
        try {
            return SQL.rowAsTuple(sql, rs);
        } catch (DataAccessException e) {
            close();
            throw e;
        }
    }
}
```

This class basically delegates the iterator methods to the underlying result set and 
then on the next() call transforms the current row in the ResultSet into my Tuple type.

And that's the basics done. All that's left is to wire it all together to make 
a Stream object.  Note that due to the nature of a ResultSet it's not a good idea to 
try process them in parallel, so our stream cannot process in parallel.

```java
public static Stream stream(final Connection connection, 
                                       final String sql, 
                                       final Object... parms) {
  return StreamSupport
                  .stream(Spliterators.spliteratorUnknownSize(
                          new ResultSetIterator(connection, sql), 0), false);
}
```

Now it's straightforward to stream a query. In the usage example below I've got 
a table TEST_TABLE with an integer column TEST_ID which basically filters out all 
the non even numbers and then runs a count:

```java
     long result = stream(connection, "select TEST_ID from TEST_TABLE")
                .filter((t) -> t.asInt("TEST_ID") % 2 == 0)
                .limit(100)
                .count();
```                
    
And that's it!, we now have a very powerful way of working with a ResultSet.

So all this code is available under an Apache 2.0 license on GitHub here. 
I've rather lamely dubbed the project "lambda tuples, and the purpose really is to experiment 
and see where you can take Java 8 and Relational DB access, 
so please download or feel free to contribute.
