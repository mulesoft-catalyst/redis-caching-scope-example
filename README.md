# Caching Scope Example w/Redis

This example provides two separate caching implementations for Mule-based APIs, both using Redis. The primary 
purpose of caching is to increase data retrieval performance by reducing the need to access the underlying 
slower system layer. A cache typically stores a subset of data locally for faster retrieval, in contrast to 
databases whose data is usually complete and durable but costly in terms of performance regarding access.

### Why use Redis and _not_ the Object Store v2?
Object Store v2 is an excellent option for CloudHub-based applications, but it cannot be used with hybrid/on-premise runtime deployment models. 
For more information on Object Store v2, please review the [FAQ](https://docs.mulesoft.com/object-store/osv2-faq).

[Redis](https://redis.io/) is an open-source (BSD licensed), in-memory data structure store that is easy to install, setup, and manage.
Redis provides an excellent solution for caching across multiple applications. 
  

### How is caching using Redis implemented?
Caching is implemented in two separate ways in this Mule 4 example:

 - Redis is configured as a custom object store
 	- `./src/main/mule/caching-scope-example-impl.xml`
 - Redis is used outright as global cache
 	- `./src/main/mule/caching-scope-example-redis-impl.xml`

By configuring a custom object store to use Redis, each flow can use the normal object store scopes and connectors 
provided Anypoint Studio's Mule Palette. This implementation is preferable when moving from object store implementations since 
the only changes need to be made at the configuration level.    

Using Redis directly as global cache allows for more explicit control on how the cache mechanism functions. The Redis 
Connector provides a variety of additional operations to help manage the keys and values in the cache.   

### Is Redis the only option?
Of course not! However, Redis has become the defacto option for external caching for on-premise runtimes. Customers have also 
used Memcached and Apache Cassandra for global caching. CloudHub customers may also use Mule's [Object Store v2](https://docs.mulesoft.com/object-store/) for global caching.

### What is required to run the example?
 - Access to a Redis instance (either installed locally or managed locally).
 	- Redis connectivity properties are stored in `./src/main/resources/example-properties.yaml`
 	- DockerHub provides an excellent [Redis image](https://hub.docker.com/_/redis/). 
 - Anypoint Studio v7.4.2+

### What is included in the example?
The example is loosely based on the use case of retrieving and updating a passenger's flight plan using the ID of the associated reservation.
The reservation ID is traditionally used as the primary key in most airline reservation systems. Its uniqueness will also serve as cache key for each flight plan.  
The flight plan resource provides two operations:

 - Retrieving a flight plan (and caching the results in Redis).
 - Updating a flight plan (and invalidating the cache key in Redis).
 
 There are also two additional operations provided to simplify the management of data in Redis:
 
 - Delete a cached value using its associated key.
 - Invalidate the entire cache.

You can alternate between the two implementations. Simply update the flow references in `./src/main/mule/caching-scope-example.xml`. 

For the Redis Custom Object Store Configuration, use the following flows:
- `GET-reservation-flightPlanFlow` for flight plan retrieval.
- `PUT-reservation-flightPlanFlow` for flight plan update.
- `DEL-remove-cache-valueFlow` for removing a cache value.
- `DEL-invalidate-cacheFlow` for invalidating the entire cache.

For leveraging Redis directly as a global cache, use the following flows:
- `GET-reservation-flightPlan-redisFlow` for flight plan retrieval.
- `PUT-reservation-flightPlan-redisFlow` for flight plan update.
- `DEL-remove-cache-value-redisFlow` for removing a cache value.
- `DEL-invalidate-cache-redisFlow` for invalidating the entire cache (currently not supported by the Redis connector).

### For how long should I cache my values?
In caching, this is always the central question and unfortunately, the answer is _it depends_.

If you don't expect values to change much over an hour or day, then time-to-live (TTL) values can be set for longer periods of time.
However, if values change in near real-time, then TTL values will need to be set for smaller durations or maybe a cache is not required at all.
Most customers will also include a cache value reset once an update operation is applied to the data set. An example of this is provided as part
of the flight plan update operation.
