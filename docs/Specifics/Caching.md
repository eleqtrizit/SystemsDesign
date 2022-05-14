# Cache
![[Pasted image 20220417112844.png]]


Caching improves page load times and can reduce the load on your servers and databases. In this model, the dispatcher will first lookup if the request has been made before and try to find the previous result to return, in order to save the actual execution.

Databases often benefit from a uniform distribution of reads and writes across its partitions. Popular items can skew the distribution, causing bottlenecks. Putting a cache in front of a database can help absorb uneven loads and spikes in traffic.

### [](https://github.com/donnemartin/system-design-primer#client-caching)Client caching
Caches can be located on the client side (OS or browser), [server side](https://github.com/donnemartin/system-design-primer#reverse-proxy-web-server), or in a distinct cache layer.

### [](https://github.com/donnemartin/system-design-primer#cdn-caching)CDN caching
[CDNs](https://github.com/donnemartin/system-design-primer#content-delivery-network) are considered a type of cache.

### [](https://github.com/donnemartin/system-design-primer#web-server-caching)Web server caching
[Reverse proxies](https://github.com/donnemartin/system-design-primer#reverse-proxy-web-server) and caches such as [Varnish](https://www.varnish-cache.org/) can serve static and dynamic content directly. Web servers can also cache requests, returning responses without having to contact application servers.

### [](https://github.com/donnemartin/system-design-primer#database-caching)Database caching
Your database usually includes some level of caching in a default configuration, optimized for a generic use case. Tweaking these settings for specific usage patterns can further boost performance.

### Application caching
In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage. Since the data is held in RAM, it is much faster than typical databases where data is stored on disk. RAM is more limited than disk, so [cache invalidation](https://en.wikipedia.org/wiki/Cache_algorithms) algorithms such as [least recently used (LRU)](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)) can help invalidate 'cold' entries and keep 'hot' data in RAM.


### Caching at the database query level
Whenever you query the database, hash the query as a key and store the result to the cache. This approach suffers from expiration issues:
-   Hard to delete a cached result with complex queries
-   If one piece of data changes such as a table cell, you need to delete all cached queries that might include the changed cell

### Caching at the object level
See your data as an object, similar to what you do with your application code. Have your application assemble the dataset from the database into a class instance or a data structure(s):
-   Remove the object from cache if its underlying data has changed
-   Allows for asynchronous processing: workers assemble objects by consuming the latest cached object

Suggestions of what to cache:
-   User sessions
-   Fully rendered web pages
-   Activity streams
-   User graph data

## Cache Updates
Eviction policies:
	-  Cache aside
		- client makes a write, then invalidates the related cache
		- not guaranteed to be consistent
			- DB could be updated by outside process
			- query to invalidate could fail
	-  Write through
		- In write-through, data is **simultaneously updated to cache and memory**. This process is simpler and more reliable. This is used when there are no frequent writes to the cache(The number of write operations is less).
	- Write Behind/Back/Deferred
		- The data is updated only in the cache and updated into the memory at a later time.
		- Implies database cannot fail
		- Have to watch out for out-of-order updates
	- Refresh ahead
		- Automaticaly update caches before they expire
		- Helps prevent slow reads due to fetch from DB
		- Read through occurs if cache does expire
	- Read through
		- if the cache is not availabe, get from data source and cache for later use.  Else, just return the existing cache.