# Anorm

Anorm is a simple data access layer that uses plain SQL to interact with the database and provides an API to parse and transform the resulting datasets: [More information](docs/manual/working/scalaGuide/main/sql/ScalaAnorm.md).

## This Fork

If `connection.getAutoCommit` is false then invokes the `setFetchSize` of the `java.sql.Statement` to a large
number and invokes `println` as an indication of this. Internally only `resultSet` is affected, so any execution
path which does not include `resultSet` in its stack won't have the fetch size set.

This behavior is specific to the Redshift and PostgreSQL JDBC implementations.

Because you will likely need to build this for a Scala 2.11 configuration:

```bash
$ sbt
$ sbt (anorm-parent)> ++ 2.11.7 publishLocal
```

An example of use:

```scala
// assume connection is in scope
connection.setAutoCommit(false) // necessary for PGSQL/Redshift to page

@tailrec
def recursive(cursor: Option[Cursor], count: Long): Long = cursor match {
    case Some(c) =>
      val record = parseRecord(c.row)
      printer.println(record)
      recursive(c.next, count + 1L)
    case None =>
      count
}

val result: Either[List[Throwable], Long] = SQL(sql).withResult(recursive(_, 0L))
result match {
    case Left(errors) =>
      errors.foreach(logger.error("While executing recursive", _))
      sys.exit(1)
    case Right(count) =>
      logger.info(d"Count: $count%,d") 
  }
}

connection.setAutoCommit(true) // good idea if you return connections to a pool that does not expect autocommit to false
```

## Usage

In a projects built with SBT, dependency to Anorm can be added as following:

```scala
libraryDependencies ++= Seq(
  "com.typesafe.play" %% "anorm" % "2.5.1f-SNAPSHOT")
```

## Build manually

Anorm can be built from this source repository.

    sbt publish-local

To run the tests, use:

    sbt test

[Travis](https://travis-ci.org/playframework/anorm): ![Travis build status](https://travis-ci.org/playframework/anorm.svg?branch=master)

## Documentation

To run the documentation server, run:

    sbt docs/run

To test the documentation code samples, run:

    sbt docs/test
