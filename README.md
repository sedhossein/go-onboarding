# Go 10 Week Backend Eng Onboarding

This 10 week Go onboarding program covers a few different topics:

- Go fundamentals & performance
- Databases, scaling & Redis. Common patterns for scalability
- Testing best practices
- Review, measurement, errors and the whole code lifecycle
- Raft & WebRTC

During this onboarding we work in a group and pick up a real project, the best way to learn is to combine the materials with actual work. Welcome, and let’s get started.

**Onboarding Table of Contents**

<aside>
    💡 Some parts of this document refer to internal documentation requiring authentication

</aside>

### **Week 1 - Go Basics**

You’ll be surprised how easy it is to pickup the basics of Go. Go as a language aims to have less features, and instead focuses on simplicity, performance and compilation speed. The power of Go comes from having performance that’s close to C++/Rust, with developer productivity that’s close to Python/Ruby.

**Editor & Setup**

- Install Goland. https://www.jetbrains.com/go/
- This Reddit is awesome: https://www.reddit.com/r/golang/
- And this newsletter: https://golangweekly.com/

**Learning the basics of Go**

- The best starting point to learn Go is the Tour: https://go.dev/tour/welcome/1
- After that be sure to scan over the learn x in y for Go: https://learnxinyminutes.com/docs/go/
- With a bit more time you can also go over these: https://exercism.org/tracks/go
- For those that prefer a book: Go in action https://www.manning.com/books/go-in-action

**Differences from other languages**

If you’re coming from a different language most concepts in Go will feel familiar. Some things are a little different though so take a moment to understand those:

- Errors are returned and wrapped (for correct stack traces)
- Channels & Go routines: https://go.dev/tour/concurrency/1
- Defer: https://go.dev/tour/flowcontrol/12
- Go doesn’t have overloading: https://go.dev/doc/faq#overloading
- Interfaces are implemented implicitly https://go.dev/tour/methods/10
- Function parameters are a common pattern https://www.alexedwards.net/blog/demystifying-function-parameters-in-go

Other Gotchas: https://divan.dev/posts/avoid_gotchas/

### **Week 2 - Testing with Go**

Learning the syntax of Go is relatively easy. The most common reasons why we see backend engineers struggle are around testing, understanding databases or understanding the requirements/ business needs. That’s why week 2 focuses on testing.

**It’s easy to get testing wrong:**

- Not writing tests. If you do this, it eventually becomes very hard to fix issues in your codebase.
- Mock everything. Some Mocking is essential, however for some devs that turns into an obsession with mocks, and if you do that you end up testing nothing.
- Test everything. You can overdo it. If you add tests everywhere it can become hard to refactor your code and update tests.
- Not having integration tests. This is related to mocking too much. I’ve seen large test suites that test lots of things without actually guaranteeing that the software does what it’s intended to do. You always need some integration tests

**Resources to learn testing**

- Study interfaces: https://gobyexample.com/interfaces. It’s important to understand interfaces since you need to use interfaces + mocks to make code testable in go
- https://www.calhoun.io/how-to-test-with-go/
- https://github.com/stretchr/testify we use this library for assertions, test suites and mocks
- https://ieftimov.com/posts/testing-in-go-go-test/
- https://quii.gitbook.io/learn-go-with-tests/
- https://www.youtube.com/watch?v=yszygk1cpEc

**Mini practice session**

- Create a little CLI command that searches google (no API just open the URL) for a given keyword and returns a boolean if “getstream.io” is in the response body
- Abstract the google searching with a SearchEngine interface
- Create a DummySearchEngine Mock with testify
- Write a test that verifies you’re correctly detecting if [getstream.io](http://getstream.io) is present

### Week 3 - SQL & Database Performance

Now that we’ve covered the basics of Go & testing it’s time to talk about databases. In particular you want to understand DB performance in more detail since the traffic on Stream’s API is quite high. Join the #learn_db slack channel.

**Database Basics**

- Study Bun (we use both go pg and bun, so eventually you’ll also need to review the older go pg package):
- Defining Models: https://bun.uptrace.dev/guide/models.html
- Inserting Data: https://bun.uptrace.dev/guide/query-insert.html
- Different relations: https://bun.uptrace.dev/guide/relations.html
- Select: https://bun.uptrace.dev/guide/query-select.html

**Indexes & Performance**

- Understand unique, index etc: https://use-the-index-luke.com/
- Issues with limit/offset pagination: https://www.citusdata.com/blog/2016/03/30/five-ways-to-paginate/
- Deadlocks:
- https://www.cybertec-postgresql.com/en/postgresql-understanding-deadlocks/
- https://morningcoffee.io/deadlocks-in-postgresql.html
- PG stat statements/ Statements: https://www.cockroachlabs.com/docs/stable/ui-statements-page

**Architecture Tips**

- APIs tend to have more reads than writes
- Denormalizing data can often help with performance. IE you can have a “reaction count” field on a message instead of having the DB fetch all messages
- A database doesn’t scale as well as a caching layer. So if you have a lot of repeated reads on something you often want to have a cache in front of the database
- A common mistake is doing N queries. Say that you have a list of blogposts and you do 1 query per blogpost to fetch the Author. In that you’re doing N queries. You should avoid this and fetch the data in 1 or 2 queries
- The second most common mistake with databases is not getting the indexes right. Be sure to check index usage with EXPLAIN
- https://www.cybertec-postgresql.com/en/how-to-interpret-postgresql-explain-analyze-output/?utm_source=dbplatz

**Other Tips**

- Tracing is super useful to visualize what happens when API calls are performed. We run Jaeger locally so that it is easy to have an easy way to see what is going on with cache/db
- When building code that relies on a cache layer, it is always a good idea to write specific tests that ensure caching is working correctly
- There are tons of things you can get wrong when dealing with a database, here’s a few interesting reads
- **Read-modify-write anti-pattern** https://www.2ndquadrant.com/en/blog/postgresql-anti-patterns-read-modify-write-cycles/
- **SQL injection is real**: always use `?` placeholders in your SQL queries and never interpolate arbitrary strings in your SQL statements (if you really must, then make sure to escape it)
- Good books:
- https://www.amazon.com/Learn-PostgreSQL-manage-scalable-databases/dp/1837635641/
- https://theartofpostgresql.com/
- PG howtos https://gitlab.com/postgres-ai/postgresql-consulting/postgres-howtos

### Week 4 - OpenAPI & JSON

**HTTP**

Make sure to first play with some basic HTTP/REST/JSON stuff. Our APIs do a lot of custom stuff but behind the scenes we still the standard library (net/http).

**Things to study to get ready**

Here’s one that covers many common things related to HTTP client/servers. https://www.digitalocean.com/community/tutorials/how-to-make-an-http-server-in-go

- Building a simple HTTP server
- HTTP handler functions and middleware
- Build simple client/server program that handles input coming from body, query params and path params

**How to test an HTTP server and its controllers**

Golang comes with default testing facilities to test request/response objects. We abstract most of this stuff in our API but it still good to have a good understanding on what happens behind the scenes. Here’s a good article that explains the basics around testing HTTP req/resp

https://quii.gitbook.io/learn-go-with-tests/build-an-application/http-server

- Testing HTTP servers and handlers
- Input validation

Note: in most cases HTTP controllers and HTTP servers do not do much more than exposing some complex business logic. At Stream most of the logic is not inside the controller but inside the `state` package. Controllers mostly handle authentication and permission on top of that.

Testing HTTP controllers really often requires mocking/stubbing, make sure to spend a bit of time reading about Go’s interfaces and mocking.

- Mock library we use on our projects https://github.com/matryer/moq
- https://quii.gitbook.io/learn-go-with-tests/go-fundamentals/mocking

```go
type GranadeLauncher interface {
Reload() error
Aim(x, y, z float64) error
Shoot() (bool, error)
}

type M75 struct {
// here the real implementation throws real granades, very expensive to test
}

type MyMock struct {
GranadeLauncher
}

func (MyMock) Reload() {}
```

> **Tip**: sometimes you want to create a quick implementation of an interface but do not want to get all methods implemented. You can embed the interface type into your struct. Doing so gives you a real-implementation. NOTE: calling `Aim` or `Shoot` on this interface will raise a nil pointer exception ;)
>

**JSON & Golang**

Compared to Node.js or Python, Golang exposes a lower-level API when it comes to HTTP. Make sure to understand the basics related to reading the body for a request and a response.

Golang supports JSON out-of-the-box, you can encode/decode (marshall/unmarshall) types from/to JSON/Go very easily.

Make sure to study how JSON works in Golang:

- Field tags (annotations)
- Encoding and Decoding payloads
- Zero-values + pointer types + JSON encode/decode

**Request validation**

- Query parameters
- Path
- Body

**OpenAPI**

- Quick introduction
- Go + OpenAPI for public APIs

**Goland Editor Tips**

- https://www.youtube.com/watch?v=N0jvAea46YM
- https://www.youtube.com/watch?v=FdS75SJN-wI

### Week 5 - Redis

https://redis.io/

https://github.com/redis/go-redis

**Use Cases**

- Caching
- Rate limits
- Counters

This interactive tutorial is a nice start: https://try.redis.io/

Here’s a super basic tutorial: https://betterstack.com/community/guides/scaling-go/redis-caching-golang/#final-thoughts

B**asics**

- String
- Hash
- Sorted Set
- PubSub

Client side caching:

https://redis.io/docs/manual/client-side-caching/

Pipelines

https://redis.io/docs/manual/pipelining/

LUA scripting

- https://redis.io/docs/interact/programmability/eval-intro/
- Study readstate/state/redis/v2/lua

Understand locks & semaphores: https://pkg.go.dev/golang.org/x/sync/semaphore and proceed with how Redis enables you build distributes versions of this. See section 6 of this book

https://redis.com/ebook/redis-in-action/

Understand rate limiting: https://medium.com/@SaiRahulAkarapu/rate-limiting-algorithms-using-redis-eb4427b47e33

Assignment

- Create a sorted set of followers
- Store the sorted set in Redis
- Keep both redis & local memory in sync with the client side caching approach discussed above

Cache stampede: https://zahere.com/cache-stampede-a-problem-the-industry-fights-every-day

### **Week 6 - Go Advanced Syntax**

**Go routines**

- The basic https://go.dev/tour/concurrency/1

**Context**

- context.Context https://medium.com/@jamal.kaksouri/the-complete-guide-to-context-in-golang-efficient-concurrency-management-43d722f6eaea
- More context best practices: https://zenhorace.dev/blog/context-control-go/
- Panic and recovery: https://gobyexample.com/recover

**Functions & Constructors**

- Why constructors use the weird WithOption style setup: https://golang.cafe/blog/golang-functional-options-pattern.html
- Function parameters: https://www.alexedwards.net/blog/demystifying-function-parameters-in-go
- Variadic functions: https://gobyexample.com/variadic-functions

**Sync package**

- sync.Mutex and sync.RWMutex
- deadlocks & data races: https://www.craig-wood.com/nick/articles/deadlocks-in-go/
- wait groups: https://gobyexample.com/waitgroups
- error groups: https://medium.com/@yardenlaif/go-sync-or-go-home-errgroup-f91a0ee72d3f
- sync.Once: https://leangaurav.medium.com/golang-channels-vs-sync-once-for-one-time-execution-of-code-fafc81d2f54d
- Arena: https://uptrace.dev/blog/golang-memory-arena.html

**Maps, Slices and Interfaces**

- Type switch https://go.dev/tour/methods/16
- Pattern: embedded interface https://eli.thegreenplace.net/2020/embedding-in-go-part-3-interfaces-in-structs/

**Structs:** Embedding structs https://gobyexample.com/struct-embedding

**Generics**: https://go.dev/doc/tutorial/generics

**Design Patterns in Go**

- https://refactoring.guru/design-patterns/go

### Week 7 - Tracing & Performance

**Tracing & Errors**

The first step of fixing performance is understanding what’s slow. Often that means instrumenting your code with timers using Prometheus and Open telemetry.

Some components are automatically traced and measured for you, for instance:

- API controllers latency and errors
- Redis calls with their latency (open telemetry)
- SQL queries (open telemetry)

**Benchmarks:**

- Basics: https://gobyexample.com/testing-and-benchmarking
- Benchmarking tips: https://medium.com/hyperskill/testing-and-benchmarking-in-go-e33a54b413e

**Monitoring Errors & Performance**

At Stream we use a few different tools to monitor errors and performance:

- ELK stack for logs
- Prometheus
- Grafana
- Sentry for tracking programming errors
- Jaeger to inspect traces locally
- Grafana Tempo (production)
- Cloudwatch/SNS for AWS metrics and alerts

**Common Performance Issues**

Go is extremely fast and likely won’t be the bottleneck for performance issues. The most common causes of performance issues are:

- Doing too many queries
- Queries not using the right index
- No caching in place where you need caching
- Deadlocks either in the DB or using mutexes in Go
- Code complexity causing you to do things you don’t need to be doing

That being said, you will occasionally run into Go performance challenges. Especially on the video project where buffering can be intense.

**Go Performance Optimization**

- Profiling Go Programs: https://go.dev/blog/pprof
- Go sync.Pool: https://www.sobyte.net/post/2022-06/go-sync-pool/. sync.Pool is helpful for reducing how much time your go program spends on garbage collection
- String operations: https://medium.com/@ozoniuss/optimizing-go-string-operations-with-practical-examples-83df39b776fb
- Profile guided optimization: https://go.dev/doc/pgo
- Old tips. Somewhat dated: https://dave.cheney.net/high-performance-go-workshop/dotgo-paris.html
- Skipset should be used when we want to iterate over a set in a lock free way. (otherwise you would have to use slice + mutex which blocks when there are writes). As an example think about the participant list. You don't want to block video while a new participant joins
- The utils/lock.go uses a debug build flag to add instrumentation to locks. This helps spot deadlocks. Also see https://github.com/sasha-s/go-deadlock

**Production Issues**

- Searching logs
- Using Grafana
- Using tracing
- Using ELK stack to understand the logs
- Reproduce an issue
- Root cause
- Post mortem

### Week 8 - Pebble & Raft Consensus

Raft and RockDB/Pebble are things you don’t often need to use. In 99% of cases simple CockroachDB & Redis will be a better fit. It can be interesting for systems where you have very high throughput though.

**Pebble & Raft**

- https://github.com/cockroachdb/pebble
- https://raft.github.io/
- https://github.com/hashicorp/raft

### Week 9 - WebRTC

You can’t fully learn WebRTC in a week, but going through the webrtccourse is possible in 1 week and you’ll learn the basics.

- WebRTC course: https://webrtccourse.com/course/webrtc-codelab/module/fiddle-of-the-month/lesson/simulcast-playground/
- Anatomy of a WebRTC SDP packet: https://webrtchacks.com/sdp-anatomy/
- Example Webrtc: https://github.com/pion/example-webrtc-applications
- More examples: https://github.com/pion/webrtc/tree/master/examples
- WebRTC race conditions/ Glare: https://blog.mozilla.org/webrtc/perfect-negotiation-in-webrtc/
- WebRTC for the curious: https://webrtcforthecurious.com/docs/01-what-why-and-how/

## Week 10 - Exam time. Complete the following tasks in 1 week

**OG tag url preview service (1 day)**

- Goal: Basic testing and monitoring
- Create an API that gives you OG tags as json for a URL
- Consider redis caching for popular urls
- Consider failure scenarios and circuit breakers
- Write tests
- Think about metrics
- This has to be easy for you. If it takes you a week you’re not up to speed yet

**Latest Apple stock price lookup (1 day)**

- Goal: understand client side caching and network latency
- Start 3 servers that use redis client side caching to store the latest apple stock price
- Measure response time (should be 0-1ms to show the stock price)
- Think about how something like this could be used for realtime

**Create a realtime text editor API (1 day)**

- Goal: understand finite state machines and redis pubsub
- The API should receive text edits (not the final text, just the edits) from multiple users
- You replicate these edits via redis pub sub
- The API returns the
- Resulting text
- The deltas via pubsub
- On the other client you should be able to reconstruct the text from the deltas

**Create a commenting API (1 day)**

- Goal: database usage
- Create a threaded commenting API powered by cockroachdb
- Comments should have upvotes, downvotes, likes and a reply count
- Efficient API endpoints to show
- Comments by date
- Comments by reply count
- Comments by upvotes
- Consider both database efficiency and Redis efficiency

**Grading: Send a Github repo to Tommaso**

| Grade | Description |
| --- | --- |
| 5 | complete 2 out of 4 |
| 6 | complete 3 out of 4 |
| 7-10 | complete 4 out of 4, more points for a solid understanding per exercise |

## **Recap & Reminders**

The most common reasons for a backend engineer to get stuck are:

- Not writing tests. And the resulting complexity that you end up with if you don’t write tests. Very easy to get stuck
- Not understanding the database and Redis
- Not understanding the business/product full stack requirements. (if you’ve ever worked on SDKs you have a major advantage in this area)
- Making things more complex than they need to be

Keep that in mind and be sure to avoid those pitfalls

A note on working in a team:

- As a guideline we recommend everyone post an update to their slack channel and lattice by Friday. You want to let your manager know how things are going and keep them updated. If you don’t do this the manager will end up continuously asking you how it’s going. If your manager needs to reach out to you and ask how its going its a sign you’re not reporting up effectively
- It’s not just about your manager, also keep your team informed about what you’re working on and how things are going
- Use channels instead of DMs on slack

## Appendix 1 - Tech Lead Onboarding Tips

## Appendix 2 - Resources & Other topics

### Devops Basics

- Cloudinit
- Cloud Formation
- Puppet

### Other Storage Tech

- Fulltext search with ElasticSearch
- RabbitMQ or similar message queues

### Scaling

We’ve started on this topic, but didn’t cover everything yet.

- Partitioning data. Either at the app level or DB level: https://www.cockroachlabs.com/docs/stable/partitioning
- Replication using leader/follower (previously master/slave)
- App level challenges around replication delay
- Redis Clustering. https://redis.io/docs/management/scaling/
- Task Queuing

### Postgresql Performance

1. https://gitlab.com/postgres-ai/postgresql-consulting/postgres-howtos
2. https://github.com/NikolayS/postgres_dba/tree/master/sql
3. https://www.highgo.ca/blog/
4. https://youtu.be/gtXNWdW42SQ
5. https://youtu.be/fHlIJg4x13g
6. https://twitter.com/jer_s
7. https://www.youtube.com/watch?v=ysG3x2QOu0c&list=PLhr1KZpdzukeAZuX0pNZ1lLDkaZWBOhQT

### Postgresql misc

- [PostgreSQL In-Depth Training](https://www.youtube.com/watch?v=ysG3x2QOu0c&list=PLhr1KZpdzukeAZuX0pNZ1lLDkaZWBOhQT)
- [Postgres HowTo articles](https://gitlab.com/postgres-ai/postgresql-consulting/postgres-howtos)
- [PostgreSQL blog posts](https://www.highgo.ca/blog/)
- https://zahere.com/cache-stampede-a-problem-the-industry-fights-every-day
- https://postgres.ai/blog/20220525-common-db-schema-change-mistakes
- Useful Scripts: https://github.com/NikolayS/postgres_dba/tree/master/sql

**Cohorts**

**Resources**

‣