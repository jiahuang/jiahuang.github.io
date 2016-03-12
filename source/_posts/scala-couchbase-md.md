---
title: Migrating couchbase entities with Scala
date: 2016-02-27 22:45:31
tags:
---

Due to some refactoring, I had to rework some entity models in couchbase. Since the codebase was already in Scala, I decided to write the migration in Scala as well. In reflection, the typesafety of Scala made this task harder than it needed to be.

A simplified example is to change a model that looks like this:

```js
//OldJobEntity
{
  id: "abc123",
  state: "Complete",
  links: [{
    s3url: "bucket.s3.amazonaws.com/abc123",
    signedUrl: "https://bucket.s3.amazonaws.com/abc123?awsSigning"}],
  doctype: "job"
}
```

to something like this:

```js
// JobEntity
{
  id: "abc123",
  state: "Complete",
  medias: [{id: "abc123Media"}],
  created: "Sun, 28 Feb 2016 07:10:46 GMT",
  doctype: "job"
}

// MediaEntity
// with abc123Media being
{
  id: "abc123Media",
  s3url: "bucket.s3.amazonaws.com/abc123",
  signedUrl: "https://bucket.s3.amazonaws.com/abc123?awsSigning",
  created: "Sun, 28 Feb 2016 07:10:46 GMT",
  doctype: "media"
}
```

I had to convert the existing `OldJob` entities to a new form of a `Job` entity that now referenced a new `Media` entity.

### Type checking when there are 2 correct types

The entities are stored in couchbase as JSON docs, and I used the Spray JSON library for converting back & forth between Scala case classes and JSON. A typical conversion using the [cloud-couchbase-wrapper](https://github.com/3drobotics/cloud-couchbase-wrapper) looks like this:

```scala
val query = couchbase.compoundIndexQueryByRangeToEntity[OldJobEntity]
  ("base", "doctype", // the first two arguments are the design & view names. The "doctype" view in my setup allows me to query for doctypes matching "job"
    Some(Seq("job", "", "")), // starting index for the query
    Some(Seq("job", "ZZZ", "ZZZ"))) // ending index for the query
```

But Spray needs to know how to convert a JSON entity to a Scala entity & vice versa. This is done through extending the DefaultProtocol trait:

```scala
class Protocol extends DefaultJsonProtocol {
  implicit val jobFormat = jsonFormat5(JobEntity.apply)
  implicit val mediaFormat = jsonFormat5(MediaEntity.apply)
}
```

However I can't just cast the old `job` entities to the new `job` entities because the old entities lack the proper fields (specifically, medias & created). So instead I ended up recreating the old entities as a Scala case class:

```scala
case class OldJobLinks(s3url: String, signedUrl: String)
case class OldJobEntity(id: String, state: String, links: List[OldJobLinks]){
  // this allows us to get new Job Entities and Media Entities from an old job
  def convert: (JobEntity, List[MediaEntity]) = {
    // do some conversion here to return a tuple of things to save
  }
}

// extend the old protocol so we can go from json -> old job entity and back
class MigrationProtocol extends Protocol {
  implicit val jobLinksFormat = jsonFormat2(OldJobLinks.apply)
  implicit val oldJobFormat = jsonFormat4(OldJobEntity.apply)
}
```

Since this migration only touched one type of entity, it wasn't so annoying. But if I had to add `created` fields to 10+ types of entities, I would have to add in 10+ old case classes. At that point, I would just skip typecasting and directly use Spray to manipulate JSON.

### Preventing the migration script from migrating twice
If the migration script ever runs twice by accident, I don't want it messing with already migrated entities. There are two ways of getting around this:

1. Casting all `job` entities to either `OldJob` or `Job` types. If it matches the `Job`, and not `OldJob`, don't migrate.
2. Add optional fields to the `OldJob` type so that `Job` types can also be casted as an `OldJob`, and check for those optional fields to distinguish between the two.

Solution 1 would look something like this:
```scala
couchbase.compoundIndexQueryByRangeToEntity[OldJobEntity]
  ("base", "doctype",
    Some(Seq("job", "", "")),
    Some(Seq("job", "ZZZ", "ZZZ"))) // query to get all docs of type "job" and try to cast as old job entity
  .map{doc => doc.entity} // get the old job from the document stream
  .grouped(MaxInt) // group all the entities together in a list
  .runWith(Sink.head) // get the list of job entities  
  .onFailure{
    case ex: DeserializationException => // try to rerun the same query with casting to JobEntity
    case ex: Throwable => throw ex
  }  
```

And solution 2 looks like:
```scala
case class OldJobEntity(id: String, state: String, doctype: String // both old and new jobs have these fields
  links: Option[List[OldJobLinks]], // optional because new JobEntities do not have links
  medias: Option[List[MediaEntity]], // optional because old jobs didn't have this
  created: Option[String] // old jobs didn't have this
  ){
    // convert now returns an option
    // if it has something, we need to convert
    // otherwise if it returns none, the entity has already been migrated
    def convert: Option[(JobEntity, List[MediaEntity])] = {
      if (this.links.nonEmpty) {
        // this is an old job
        // do some conversion here to return a tuple of things to save
      } else {
        None // this is already a new job, no need to convert
      }
    }
  }
```

I implemented solution 2, but if I had to do it again, I would probably go with solution 1.  

### Verification
Fortunately, the database was still small enough to allow me to run it locally. So to verify the migration, I ended up pulling down the couchbase data from the dev/staging/production servers and running it locally to make sure nothing went wrong.

```sh
// ssh into dev servers
ssh ubuntu@server

// use the couchbase backup util to backup the database
cbbackup http://localhost:8091 ~/dir/to/save/backups -u Administrator -p password

// zip it up
zip -r ~/dir/to/save/backups/backup.zip ~/dir/to/save/backups/backupFile

// exit out of dev server

// scp it locally
scp -i ~/.ssh/key.pem ubuntu@server:/home/ubuntu/dir/to/save/backups/backup.zip .

// restore from a backup
cbrestore ~/dir/to/save/backups/ http://Administrator:password@localhost:8091 --bucket-source=source-bucket --bucket-destination=destination-bucket
```

The datasets that actually got migrated were small enough for me to manually check some edge cases & do some simple counts on number of old entities vs new ones.

In the future when thousands of documents are being migrated, I'm not quite sure what I'll do for verification, but I have a few ideas:

1. load up the entire migrated database as a test set for the unit tests
2. write some scripts to check for key numbers, such as X number of new `media` entities being created
3. push all old entity fields into a stack and pop off the fields as they are found in the new entity fields
