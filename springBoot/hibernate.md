1. What is a SessionFactory?
SessionFactory provides an instance of Session. It is a factory class that gives the Session objects based on the configuration parameters in order to establish the connection to the database.
As a good practice, the application generally has a single instance of SessionFactory. The internal state of a SessionFactory which includes metadata about ORM is immutable, i.e once the instance is created, it cannot be changed.

This also provides the facility to get information like statistics and metadata related to a class, query executions, etc. It also holds second-level cache data if enabled.

2. What is a Session in Hibernate?
A session is an object that maintains the connection between Java object application and database. Session also has methods for storing, retrieving, modifying or deleting data from database using methods like persist(), load(), get(), update(), delete(), etc. Additionally, It has factory methods to return Query, Criteria, and Transaction objects.

3. What are some of the important interfaces of Hibernate framework?
Hibernate core interfaces are:

Configuration
SessionFactory
Session
Criteria
Query
Transaction
12. What is the difference between first level cache and second level cache?
Hibernate has 2 cache types. First level and second level cache for which the difference is given below:

# Hibernate Cache Comparison

| First Level Cache | Second Level Cache |
|---|---|
| Local to a `Session` object and cannot be shared between multiple sessions. | Maintained at the `SessionFactory` level and shared among all sessions. |
| Enabled by default and cannot be disabled. | Disabled by default and can be enabled through configuration. |
| Exists only while the session is open; destroyed when the session is closed. | Exists across the application lifecycle; recreated when the application restarts. |
| On `get()`, Hibernate checks first-level cache first. If not found, it checks second-level cache (if configured), then database. If row is missing, returns `null`. | On `get()`, Hibernate checks first-level cache first. If not found, it checks second-level cache (if configured), then database. If row is missing, returns `null`. |

