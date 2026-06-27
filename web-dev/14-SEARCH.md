# 14 Search

This guide covers search architectures: Relational search (ILIKE, text indexes), Elasticsearch (inverted indexes, boolean filters, analyzers), database syncing models (CDC, message queues), and Algolia hosted search.

## Relational Database Search

### Definition
Relational search queries database tables using SQL matching patterns (LIKE, ILIKE, regular expressions). While effective for simple equality lookups, leading wildcard queries (e.g. LIKE '%term%') bypass index trees and force full table scans.

### Real-world Analogy
Imagine searching a directory book of residents. Searching: LIKE 'Smith%' is fast because names are ordered: you flip directly to the "S" section. Searching: LIKE '%smith%' forces you to read every single line of every single page of the phone book to find if their middle name contains "smith".

### Code Example
```sql
-- Search query that bypasses B-tree indexes due to leading wildcard
-- SELECT * FROM posts WHERE title LIKE '%database%';

-- Optimized case-insensitive search using LOWER and B-tree index
-- (Requires functional index: CREATE INDEX ON posts (LOWER(title)))
SELECT * FROM posts WHERE LOWER(title) = LOWER('Database');
```

### Common Interview Questions
- Why do leading wildcards in LIKE statements prevent B-tree indexes from being used?
- How do you configure functional indexes to speed up case-insensitive relational searches?
- When should you transition from relational search to dedicated search engines?

### Reference Links
- [PostgreSQL Docs: Pattern Matching](https://www.postgresql.org/docs/current/functions-matching.html)

## Elasticsearch Fundamentals

### Definition
Elasticsearch is a distributed, RESTful search and analytics engine based on Apache Lucene. It stores data as JSON documents and builds an Inverted Index, which maps words to the specific documents containing them to facilitate high-speed text searches.

### Real-world Analogy
Instead of reading 100 books page-by-page to find the term "event loop", you flip to the back of each book to inspect its pre-compiled Index page. The index lists: "event loop -> Book 2 Page 40, Book 5 Page 12", letting you find the exact books instantly.

### Code Example
```json
// Sample Elasticsearch Document Index Request (POST /products/_doc/1)
{
  "name": "Mechanical Keyboard",
  "category": "electronics",
  "tags": ["gaming", "hardware"],
  "price": 99.99
}
```

### Common Interview Questions
- Explain what an inverted index is and how it maps document terms.
- Describe the Elasticsearch cluster components: nodes, shards, and replica shards.
- How does Elasticsearch update documents (e.g. read-modify-write cycle under the hood)?

### Reference Links
- [Elasticsearch Docs: Introduction](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html)

## Elasticsearch Queries

### Definition
Elasticsearch queries search and filter indexed documents. Queries include Match queries (full-text analysis), Term queries (exact values, non-analyzed), and Boolean queries (combining criteria using must, should, must_not, and filter).

### Real-world Analogy
Imagine shopping on an electronics site. A Term query is checking if the brand tag is exactly "Dell". A Match query is searching for "curved monitor". A Boolean filter combines them: "The brand MUST be Dell, the description SHOULD contain 'curved', and the price MUST NOT exceed $500".

### Code Example
```json
// POST /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "description": "mechanical keyboard" } }
      ],
      "filter": [
        { "term": { "status": "active" } },
        { "range": { "price": { "lte": 150.00 } } }
      ]
    }
  }
}
```

### Common Interview Questions
- What is the difference between a Match query and a Term query?
- How do filters differ from queries in terms of relevance scoring and caching?
- What are dynamic search templates and when are they used?

### Reference Links
- [Elasticsearch Docs: Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

## Elasticsearch Analysis

### Definition
Elasticsearch Analysis transforms raw text into indexed tokens using Analyzers. An analyzer contains Character Filters (strip HTML/regex), a Tokenizer (splits text into words), and Token Filters (lowercases, removes stop words, stems words).

### Real-world Analogy
Imagine a juice processor. Character filters peel the fruit skin (strip HTML). The Tokenizer cuts the fruit into cubes (splits words). Token filters squeeze the cubes, remove seeds (stop words like "and/the"), and output fresh juice tokens.

### Code Example
```json
// Configuring a custom analyzer in index settings
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_html_stemmer": {
          "tokenizer": "standard",
          "char_filter": ["html_strip"],
          "filter": ["lowercase", "stop", "stemmer"]
        }
      }
    }
  }
}
```

### Common Interview Questions
- What is stemming and why is it critical for full-text search engines?
- What are stop words and why are they removed during indexing?
- How do you test an analyzer configuration using the _analyze API?

### Reference Links
- [Elasticsearch Docs: Text Analysis](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html)

## Elasticsearch Sorting and Pagination

### Definition
Sorting and Pagination manage query results. While from/size works for shallow offsets, deep pagination triggers resource limitations. Elasticsearch uses search_after parameters (using sorting states) or the Scroll API for pagination.

### Real-world Analogy
Imagine sorting 10,000 index cards. If you ask for the 9,000th card, the librarian must gather cards 1 to 9,000, sort them in their hands, take the last 10, and drop the rest. Using search_after is like leaving a bookmark: you start sorting from the bookmark card immediately.

### Code Example
```json
// Pagination search using search_after (POST /products/_search)
{
  "size": 10,
  "query": { "match": { "category": "electronics" } },
  "sort": [
    { "price": "asc" },
    { "_id": "asc" }
  ],
  "search_after": [49.99, "doc_101"] // Point cursor to last returned values
}
```

### Common Interview Questions
- Why does using deep from/size values (e.g. skipping 10,000+ items) degrade cluster memory?
- Explain how the search_after cursor parameter works.
- When should you use the Scroll API over search_after?

### Reference Links
- [Elasticsearch Docs: Paginate Search Results](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)

## Syncing Database with Elasticsearch

### Definition
Syncing database records with search indexes maintains data consistency. Syncing models include Dual Writing (API writes to both DB and ES), Message Queue patterns, Logstash polling, and CDC (Change Data Capture) via tools like Debezium.

### Real-world Analogy
Dual writing is like an assistant writing details in the office ledger and emailing the newsletter team at the same time: if their computer crashes midway, the ledger has the entry but the newsletter team never receives it. CDC is having a camera record the ledger page: every time the ink dries, the camera automatically scans the page and sends the copy.

### Code Example
```javascript
// Dual writing pattern (unreliable under failure)
async function createProduct(productData) {
  const product = await db.products.save(productData); // DB Save
  try {
    await elasticsearch.index({
      index: "products",
      id: product.id,
      body: productData
    }); // ES Index
  } catch (err) {
    console.error("Index sync failed: ", err);
    // Data is now out of sync!
  }
}
```

### Common Interview Questions
- Why is the Dual Writing pattern vulnerable to data inconsistency?
- Explain how Change Data Capture (CDC) with Debezium and Kafka works.
- How do you handle database deletions when syncing with Elasticsearch?

### Reference Links
- [Elastic: CDC with Debezium](https://www.elastic.co/blog/cdc-debezium-kafka-elasticsearch)

## Algolia

### Definition
Algolia is a fully managed, hosted search-as-a-service platform. It uses proprietary search engines optimized for search-as-you-type autocomplete, indexing documents via API integrations and managing query matching rules.

### Real-world Analogy
Algolia is like hiring a professional search concierge. Instead of setting up and tuning your own indexing machines in your basement (Elasticsearch cluster), you ship your catalogs to the concierge. They organize the records, and provide an instant search UI bar for visitors.

### Code Example
```javascript
const algoliasearch = require("algoliasearch");

const client = algoliasearch("YOUR_APP_ID", "YOUR_API_KEY");
const index = client.initIndex("products");

// Indexing objects in Algolia
const objects = [{ objectID: "1", name: "Mechanical Keyboard", price: 99 }];
index.saveObjects(objects).then(({ objectIDs }) => {
  console.log("Indexed in Algolia: " + objectIDs);
});
```

### Common Interview Questions
- Compare Algolia and self-hosted Elasticsearch regarding maintenance and pricing.
- What are search facets and how do you configure them in Algolia?
- How does Algolia resolve matching rules and typotolerance?

### Reference Links
- [Algolia Documentation](https://www.algolia.com/doc/)

## Scenario-based Interview Questions for Search

### Scenario 1
An e-commerce site search bar queries a SQL database using LIKE '%user_input%'. When traffic grows, the database CPU spikes to 100%, and search queries freeze. How do you resolve this?

*Expected Approach:*
1. Explain that LIKE queries with leading wildcards force full table sequential scans, bypassing B-tree indexes, which scales terribly as data grows.
2. If budget/operational scale is small, suggest migrating to PostgreSQL's native Full-Text Search using tsvector columns indexed with GIN.
3. For large catalogs with complex requirements, suggest syncing the database to an Elasticsearch cluster (using CDC or message queues) and querying the cluster instead.

### Scenario 2
Your system uses Elasticsearch. In production, users complain that recently updated products take up to 2 seconds to appear in search query results, even though the database updates immediately. Why is this happening and how do you handle it?

*Expected Approach:*
1. Explain that Elasticsearch is near-real-time: indexed documents are held in memory buffers and only made searchable after a Segment Refresh operation (which defaults to every 1 second).
2. If immediate consistency is required for critical paths, suggest sending the refresh=wait_for parameter with index/update requests to block the API response until the segment is refreshed.
3. Note that forcing synchronous refreshes degrades cluster write throughput and should be used sparingly.
