Описание сущности
```scala
final case class Movie(
    id: Long,
    name: String,
    releaseDate: LocalDate,
    lengthInMin: Int
)

// movies - Схема (если есть)
// Movie - Таблица
class MovieTable(tag: Tag) extends Table[Movie](tag, Some("movies"),"Movie") {
    def id = column[Long]("movie_id", O.PrimaryKey, O.AutoInc)
    def name = column[String]("name")
    def releaseDate = column[LocalDate]("release_date")
    def lengthInMin = column[Int]("length_in_min")
    override def * = (id, name, releaseDate, lengthInMin) <> (Movie.tupled, Movie.unapply)
  }

val shawshank = Movie(1L, "Shawshank Redemptions", LocalDate.of(1994, 4, 2), 162)
```

Select *
```scala
// Future[Seq[Movie]]
SlickTables.movieTable.result
```

Filter
```scala
// Future[Option[Movie]]
SlickTables.movieTable.filter(_.name === name).result.headOption

// Future[Seq[Movie]]
SlickTables.movieTable.filter(_.name.like("%Matrix%")).result
```

Insert
```scala
// Future[Movie]
SlickTables.movieTable += movie

// Future[Int]
SlickTables.movieTable.returning(SlickTables.movieTable) += movie

SlickTables.movieTable.returning(SlickTables.movieTable) ++= Seq(...)
```

Update
```scala
// Future[Int]
SlickTables.movieTable.filter(_.id === movieId).update(movie)

// Future[Int]
SlickTables.movieTable.filter(_.id === movieId).map(_.name).update("updatedName")
```

Delete
```scala
// Future[Int]
SlickTables.movieTable.filter(_.id === movieId).delete
```

PlainQuery
```scala
implicit val getResultMovie = 
	GetResult(r => Movie(
					r.<<, 
					r.<<, 
					LocalDate.parse(r.nextString()), 
					r.<<)
	)
// Future[Seq[Movie]]
val moviesQuery = sql"""SELECT * FROM movies."Movie" """.as[Movie]
```

Transaction
```scala
val saveMovieQuery = SlickTables.movieTable += movie
val saveActorQuery = SlickTables.actorTable += actor
val combinedQuery = DBIO.seq(saveMovieQuery, saveActorQuery)
db.run(combinedQuery.transactionally)
```

Join
```scala
final case class Actor(id: Long, name: String)
final case class MovieActorMapping(id: Long, movieId: Long, actorId: Long)

  class ActorTable(tag: Tag) extends Table[Actor](tag, Some("movies"), "Actor") {
    def id = column[Long]("actor_id", O.PrimaryKey, O.AutoInc)
    def name = column[String]("name")
    override def * = (id, name) <> (Actor.tupled, Actor.unapply)
  }

lazy val actorTable = TableQuery[ActorTable]

  class MovieActorMappingTable(tag: Tag)
      extends Table[MovieActorMapping](tag, Some("movies"), "MovieActorMapping") {
    def id = column[Long]("movie_actor_id", O.PrimaryKey, O.AutoInc)
    def movieId = column[Long]("movie_id")
    def actorId = column[Long]("actor_id")
    override def * = (id, movieId, actorId) <> (MovieActorMapping.tupled, MovieActorMapping.unapply)
  }

lazy val movieActorMappingTable = TableQuery[MovieActorMappingTable]

// select * from * movieActorMappingTable m join actorTable a on m.actorId == a.id
val joinQuery = movieActorMappingTable
      .filter(_.movieId === movieId)
      .join(actorTable)
      .on(_.actorId === _.id)
      .map(_._2)

db.run(joinQuery.result) // Future[Seq[Actor]]
```



val apps = TableQuery[OAuthAppTable]  
val users = TableQuery[UserTable]  
  
// select * from OAuthAppTable  
def listAll: Future[Seq[OAuthApp]] = db.run(apps.result)  
  
def add(oAuthApp: OAuthApp): Future[Int] = db.run(apps += oAuthApp)  
  
def findByClientId(clientId: String): Future[Option[OAuthApp]] =  
  db.run(apps.filter(_.clientId === clientId).result.headOption)  
  
def findByToken(token: String): Future[Option[OAuthApp]] =  
  db.run(apps.filter(_.accessToken === token).result.headOption)  
  
def update(clientId: String, userId: Option[Long]): Future[Int] = {  
  val query = apps  
    .filter(_.clientId === clientId)  
    .map(app => (app.accessToken, app.userId))  
    .update((Option(StringUtils.generateUuid()), userId))  
  db.run(query)  
}