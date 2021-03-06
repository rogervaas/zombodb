# [![ZomboDB](logo.png)](http://www.zombodb.com/) [![Build Status](https://travis-ci.org/zombodb/zombodb.svg?branch=master)](https://travis-ci.org/zombodb/zombodb/branches)

ZomboDB is a Postgres extension that enables efficient full-text searching via the use of indexes backed by Elasticsearch.  In order to achieve this, ZomboDB implements Postgres' [Access Method API](http://www.postgresql.org/docs/9.5/static/indexam.html).

## What am I Reading?

This is the `master` branch of ZomboDB.  It simply serves as a landing page to learn about ZomboDB.  Documentation is availble here and within each version branch.

The current version of ZomboDB is `v5.6.4-1.0.0_beta1`.

## Upgrading from ES 1.7 or ES 2.4?

In short, ZomboDB doesn't provide a migration process from ES 1.7/2.4 to ES 5.6.4.  You'll need to drop all of your `USING zombodb` indexes, manually delete all the indexes from your ES cluster, upgrade your ES cluster to 5.6.4, upgrade the ZomboDB Postgres extension to v5.6.4-x.x.x and then re-create all your `USING zombodb` indexes.

If you're interested in ES 5.6 support, switch to [`master-es5.6`](https://github.com/zombodb/zombodb/tree/master-es5.6).  If you're interested in ES 2.4 support, switch to [`master-es2.4`](https://github.com/zombodb/zombodb/tree/master-es2.4).  Otherwise, you must be looking for ES 1.7 support, which is on the [`master-es1.7`](https://github.com/zombodb/zombodb/tree/master-es1.7) branch.

If you just want to download the latest version for your ES+PG+Linux stack, please visit [www.zombodb.com/releases](https://www.zombodb.com/releases).


## Versioning Practices

ZomboDB will continue to support ES 1.7 under the `v3.x` versioning scheme, and ES 2.4 under the `v4.x` scheme.

Going forward, ZomboDB will follow, at least, the major version number of Elasticsearch.  As such, the ZomboDB version to support ES 5.6 is currently `v5.6.4-1.0.0`.  The same will be true of ES 6 support.

Other than for security fixes or very serious bugs Support for Elasticsearch 1.7 and 2.4 is now suspended.  Going forward, only ES v5.6.x (and soon ES 6.0) will be supported.

## Releases

Historically, release artifacts have been published here on Github.  Going forward, release artifacts will only be available on [https://www.zombodb.com/releases](https://www.zombodb.com/releases).

Release notes will continue to be published here on Github, just not downloadable binaries.

## More Information

In practical terms, a ZomboDB index appears to Postgres as no different than a standard btree index.  As such, standard SQL commands are fully supported, including `SELECT`, `BEGIN`, `COMMIT`, `ABORT`, `INSERT`, `UPDATE`, `DELETE`, `COPY`, and `VACUUM`.

Because ZomboDB implements Postgres' Access Method API, ZomboDB indexes are MVCC-safe, even as concurrent sessions mutate underlying tables.  Following Postgres' MVCC rules means that every transaction sees, at all times, a view of the backing Elasticsearch index that is always consistent with the current transaction's snapshot.

Behind the scenes, ZomboDB indexes communicate with Elasticsearch via HTTP and are automatically synchronized, in a high-performance manner, as data changes.

Index management happens using standard Postgres SQL commands such as `CREATE INDEX`, `REINDEX`, and `ALTER INDEX`.  Searching uses standard SQL `SELECT` statements with a custom operator that exposes a [full-text query language](SYNTAX.md) supporting most of Elasticsearch's query constructs.

Elasticsearch-calculated aggregations are also provided through custom functions.


## Quick Links
   - [Latest Release](https://www.zombodb.com/releases/latest)  
   - [Installation instructions](INSTALL.md)  
   - [Getting Started Tutorial](TUTORIAL.md)  
   - [Index Management](INDEX-MANAGEMENT.md), [Index Options](INDEX-OPTIONS.md), and [Type Mapping](TYPE-MAPPING.md)
   - [Query Syntax](SYNTAX.md)  
   - [SQL-level API](SQL-API.md)  
   - [v3.2.0 Release Notes](https://github.com/zombodb/zombodb/releases/tag/v3.2.0)
   - [v5.6.4-1.0.0 Release Notes](https://github.com/zombodb/zombodb/releases/tag/v5.6.4-1.0.0_beta1)

## Features

- transaction-safe, MVCC-correct full text queries and Elasticsearch aggregations
- managed & queried via standard Postgres SQL
- works with tables of any structure
   - Extremely fast [Cross-index joining](CROSS-INDEX-JOINS.md) for full text queries across relational data structures
- automatically creates Elasticsearch Mappings supporting most datatypes, including arrays
   - supports full set of Elasticsearch [language analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/1.7/analysis-lang-analyzer.html)
   - support for [custom analyzer chains](TYPE-MAPPING.md)
   - custom [per-field mappings](TYPE-MAPPING.md)
   - json/jsonb columns as nested objects for flexible schemaless sub-documents
- works with all Postgres query plans, including [sequential scans](SEQUENTIAL-SCAN-SUPPORT.md) 
- use whatever method you currently use for talking to Postgres (JDBC, DBI, libpq, etc)
- extremely fast indexing
- store document source in Elasticsearch so ZomboDB-generated indexes can by used by 3rd-party tools like Kibana
- [per-row scoring](SQL-API.md#function-zdb_scoretable_name-regclass-ctid-tid-returns-float4) with term/phrase boosting
- [record count estimation](SQL-API.md#function-zdb_estimate_counttable_name-regclass-query-text-returns-bigint)
- custom full-text query language supporting nearly all of Elasticsearch's search features, such as
  - boolean operations
  - proximity (in and out of order)
  - phrases, wildcards, fuzzy terms/phrases
  - regular expressions, inline scripts
  - range queries
  - "more like this"
  - any Elasticsearch query construct through direct JSON
- [query expansion](SYNTAX.md#query-expansion), [index linking](INDEX-OPTIONS.md), and [block routing](BLOCK-ROUTING.md) for improved cross-index join performance
- [search multiple tables at once](SQL-API.md#function-zdb_multi_searchtable_names-regclass-user_identifiers-text-field_names-query-text-returns-setof-zdb_multi_search_response)
- [high-performance hit highlighting](SQL-API.md#function-zdb_highlighttable_name-regclass-es_query-text-where_clause-text-returns-set-of-zdb_highlight_response)
- limit/offset/sorting of fulltext queries by Elasticsearch
- support for common Elasticsearch aggregations, including ability to nest
- access to all of Elasticsearch's aggregations via direct JSON
- extensive test suite

Not to suggest that these things are impossible, but there's a small set of non-features too:

- ZomboDB indexes are not WAL-logged by Postgres.  As such, ZomboDB indexes are not recoverable in the event of a Postgres server crash and will require a `REINDEX`
- interoperability with various Postgres replication schemes is unknown
- Postgres [HOT](https://github.com/postgres/postgres/blob/master/src/backend/access/heap/README.HOT) update chains are not supported (necessitates a `VACUUM FULL` if a HOT-updated row is found during index creation)

## Downloading

Please visit [www.zombodb.com/releases/](https://www.zombodb.com/releases) to download.

If you want to integrate with some kind of CI or deployment system, you can intuit the pattern for versions from the Elasticsearch plugin and Postgres extension download links, but it'll be something like:

```
https://www.zombodb.com/releases/VERSION/zombodb-es-plugin-VERSION.zip
https://www.zombodb.com/releases/VERSION/zombodb_trusty_pg95-VERSION_amd64.deb
```

For the Postgres extension binaries, you'll need to use the one that's for your Postgres + Linux distro combination -- the example above is for Postgres 9.5 on Ubuntu Trusty.

## What you need

Product       | Version 
---           | ---      
Postgres      | 9.3, 9.4, 9.5
Elasticsearch | 1.7.x, 2.4.6, or 5.6.4

For information about how to develop/build ZomboDB, see the [Development Guide](DEVELOPER.md).

## How to Use It

Usage is really quite simple.  Note that this is just a brief overview.  See the various documentation files for more detailed information.

Install the extension:

```sql
CREATE EXTENSION zombodb;
```

Create a table:

```sql
CREATE TABLE products (
    id SERIAL8 NOT NULL PRIMARY KEY,
    name text NOT NULL,
    keywords varchar(64)[],
    short_summary phrase,
    long_description fulltext, 
    price bigint,
    inventory_count integer,
    discontinued boolean default false,
    availability_date date
);
-- insert some data
```

Index it:

```sql
CREATE INDEX idx_zdb_products 
          ON products 
       USING zombodb(zdb('products', products.ctid), zdb(products))
        WITH (url='http://localhost:9200/', shards=5, replicas=1);
```

Query it:

```sql
SELECT * 
  FROM products 
 WHERE zdb('products', ctid) ==> 'keywords:(sports,box) or long_description:(wooden w/5 away) and price < 100000';
```

## Contact Information

- [www.zombodb.com](http://www.zombodb.com/)
- Google Group: [zombodb@googlegroups.com](mailto:zombodb@googlegroups.com)
- Twitter:  [@zombodb](https://twitter.com/zombodb)
- via github Issues and Pull Requests ;)
- [https://www.zombodb.com/services/](https://www.zombodb.com/services/)  for commercial support


## History

The name is an homage to [zombo.com](http://zombo.com/) and its long history of continuous self-affirmation. 

Development began in 2013 by [Technology Concepts & Design, Inc](http://www.tcdi.com) as a closed-source effort to provide transaction safe full-text searching on top of Postgres tables.  While Postgres' "tsearch" features are useful, they're not necessarily adequate for 200 column-wide tables with 100M rows, each containing large text content.

Initially designed on-top of Postgres' Foreign Data Wrapper API, ZomboDB quickly evolved into an index type (Access Method) so that queries are MVCC-safe and standard SQL can be used to query and manage indexes.

Elasticsearch was chosen as the backing search index because of its horizontal scaling abilities, performance, and general ease of use.

ZomboDB was open-sourced in July 2015 and has been used in numerous production systems of various sizes and complexity.

## Credit and Thanks

Credit goes to Technology Concepts & Design, Inc, its management, and its development and quality assurance teams not only for their work during the early development days but also for their on-going support now that ZomboDB is open-source.


## License

Portions Copyright 2013-2015 Technology Concepts & Design, Inc.  
Portions Copyright 2015-2017 ZomboDB, LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
