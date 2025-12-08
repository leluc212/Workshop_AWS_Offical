---
title: "Blog 3"
date: "2025-09-08"
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---


# AWS Database Blog  
## Getting Started with Apache OFBiz and Amazon Aurora DSQL  
**Author:** James Morle | **Date:** March 26, 2025 | **Categories:** Advanced (300), Amazon Aurora, Best Practices, DSQL, Technical How-to  

---

When designing Amazon Aurora DSQL, we spent a lot of time finding the right balance between innovation and improvements versus ease of use. We did not want to change what was familiar to existing PostgreSQL users unless there was a truly compelling reason. Two areas in this group are optimistic concurrency control (OCC) behavior and the authentication model, where system behavior might feel different to application developers familiar with PostgreSQL.

In this article, we present a practical example of migrating an existing application originally running on PostgreSQL to operate with Aurora DSQL. Along with adjustments to adapt to the mentioned aspects, we also handle some data type incompatibilities and work around a few current limitations in Aurora DSQL.

---

## The Application

To make this example as representative as possible, we chose an open-source application with a broad functional scope and significant complexity, reflecting real-world applications. That application is Apache OFBiz, and the source code can be downloaded from the GitHub repository. The version used in this proof of concept is 18.12.16.

The OFBiz application is a full-featured ERP and CRM suite, serving as a typical target for our goals here:  
- Complex schema: 837 tables, 4161 indexes  
- Abstract entity engine (Java Persistence, Java Transactions, custom ORM)  
- PostgreSQL support  
- Built-in demo data and data loading functionality  
- Mostly relational model, aligned with Aurora DSQL’s initial compatibility focus  

The purpose of this article is to combine a description of the internal testing process with some specific code examples on how to approach Aurora DSQL authentication in a connection pool–based Java application. While we provide as much context as possible, this is not intended to be a full script or complete codebase for replicating the entire process.

---

## Aurora DSQL

Aurora DSQL is a PostgreSQL-compatible database with some semantic differences aimed at improving scalability and security compared to community PostgreSQL. Some aspects, especially broad functional compatibility with PostgreSQL features, will evolve rapidly over time, but a few core elements define the system. These are the focus of the changes made in the OFBiz application and presented in this article. These include:

- Aurora DSQL authentication model — How connections are established and maintained.  
- Transaction size limits — Aurora DSQL does not support unlimited transaction sizes.  
- Isolation level — Aurora DSQL only supports snapshot isolation, equivalent to repeatable read in PostgreSQL.  
- Optimistic concurrency control — Aurora DSQL does not lock resources during transaction construction; it guarantees consistency at commit time. This means some commits may fail and require retries.  
- Data type limitations in keys — Currently, there are some limits on data types (and sizes) supported in primary and foreign keys. These limits will decrease over time as Aurora DSQL adds new supported types. The workarounds presented reflect the limits present in the Aurora DSQL Preview stage.

---

## Creating a Data Source Configuration for Aurora DSQL

The first step was to define a new data source configuration for Aurora DSQL. We added a new data source named `localdsql` in the `framework/entity/config/entityengine.xml` file and set it as the active data source. Parameters were copied from the existing `localpostgres` data source, with the following key differences underlined:

```xml
<group-map group-name="org.apache.ofbiz" datasource-name="localdsql"/>
...
</datasource>
<datasource name="localdsql"
    helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO"
    schema-name="ofbiz"
    field-type-name="postgres"
    check-on-start="true"
    add-missing-on-start="true"
    use-fk-initially-deferred="false"
    alias-view-columns="false"
    join-style="ansi"
    use-binary-type-for-blob="true"
    use-order-by-nulls="true"
    result-fetch-size="50">
    <read-data reader-name="tenant"/>
    <read-data reader-name="seed"/>
    <read-data reader-name="seed-initial"/>
    <group-map group-name="org.apache.ofbiz" datasource-name="localdsql"/>
...
</datasource>
<datasource name="localdsql"
    helper-class="org.apache.ofbiz.entity.datasource.GenericHelperDAO"
    schema-name="ofbiz"
    field-type-name="postgres"
    check-on-start="true"
    add-missing-on-start="true"
    use-fk-initially-deferred="false"
    alias-view-columns="false"
    join-style="ansi"
    use-binary-type-for-blob="true"
    use-order-by-nulls="true"
    result-fetch-size="50">
    <read-data reader-name="tenant"/>
    <read-data reader-name="seed"/>
    <read-data reader-name="seed-initial"/>
</datasource>

```
In the example configuration above, we hid the JDBC URI endpoint — if you perform this on your own cluster, don’t forget to update the <Aurora DSQL_URL> value to reflect your Aurora DSQL cluster endpoint.

We set the jdbc-password value to the special constant IAM_AUTH. OFBiz by default assumes standard password authentication for database connections, but Aurora DSQL requires AWS Identity and Access Management (IAM)–based authentication for a more integrated and secure environment. We add support for this mechanism in the code in the next step.

We also change the isolation level to RepeatableRead (the equivalent PostgreSQL behavior for Aurora DSQL’s snapshot isolation) and increase the minimum pool size (connections in Aurora DSQL are cheap to maintain, but encrypted connection startup is relatively slow). Aurora DSQL only supports encrypted connections, so we add this parameter to the JDBC URI. We also use the latest PG JDBC driver version (42.7.4 at writing) to ensure certified compatibility with Aurora DSQL.

### Implementing Connection Logic for Aurora DSQL

There are three key considerations when building connection logic for Aurora DSQL:
- The actual password is a dynamically generated authentication token, created on demand when IAM authentication is verified.
- The authentication token has a limited validity period — they are valid only for a specified time window at creation. This period can be up to one week, suitable for development tools, but for production applications, we recommend a much shorter duration to reduce unauthorized access risks. We chose a 3-hour validity here.
- Connections also have a lifetime, currently one hour. The interplay between token validity and connection lifetime can confuse new Aurora DSQL users, so remember the token is only applied when a connection is created (token must be valid at creation), and connection lifetime affects only existing connections (independent of token validity after creation).

To support this, we edited the file
framework/entity/src/main/java/org/apache/ofbiz/entity/config/model/EntityConfig.java to add a dedicated authentication flow for Aurora DSQL, adding the following logic:

```java
public static String getJdbcPassword(InlineJdbc inlineJdbcElement) throws GenericEntityConfException {
    String jdbcPassword = inlineJdbcElement.getJdbcPassword();
+   if (!jdbcPassword.isEmpty() && ! jdbcPassword.equals("IAM_AUTH")) {
        return jdbcPassword;
    }
+   if (jdbcPassword.equals("IAM_AUTH")) {
+        return DSQLAuth.generateToken(inlineJdbcElement.getJdbcUri().split("/")[2], 
+            Region.US_EAST_1, 
```

Then, we edited the file
`framework/entity/src/main/java/org/apache/ofbiz/entity/connection/DBCPConnectionFactory.java`
to use a dedicated ConnectionFactory for Aurora DSQL when the data source is localdsql. We also set the maximum connection lifetime in the pool slightly shorter than Aurora DSQL’s maximum supported connection lifetime.

```java
      // create the connection factory
+        org.apache.commons.dbcp2.ConnectionFactory cf = null;
+        if (cacheKey.equals("localdsql")) {
+           cf = new DSQLConnectionFactory(jdbcDriver, jdbcUri, cfProps);
+        } else {
+           cf = new DriverConnectionFactory(jdbcDriver, jdbcUri, cfProps);
+        }

         // wrap it with a LocalXAConnectionFactory
         XAConnectionFactory xacf = new LocalXAConnectionFactory(txMgr, cf);

         // create the pool object factory
         PoolableConnectionFactory factory = new PoolableManagedConnectionFactory(xacf, null);
+        ((PoolableManagedConnectionFactory)factory).setMaxConnLifetimeMillis((long)55*60*1000);
         factory.setValidationQuery(jdbcElement.getPoolJdbcTestStmt());
```

Finally, we added core logic to manage connections, including attaching the Aurora DSQL SessionId to connection attributes for better diagnostics. When contacting AWS Support to resolve Aurora DSQL-related issues, providing the Session ID of the specific problematic connection is very helpful.
We implemented the following concrete classes:

* framework/entity/src/main/java/org/apache/ofbiz/entity/connection/DSQLAuth.java

* framework/entity/src/main/java/org/apache/ofbiz/entity/connection/DSQLConnectionFactory.java

* framework/entity/src/main/java/org/apache/ofbiz/entity/connection/DSQLConnection.java

### Schema Adjustments

We made minimal schema changes to comply with current Aurora DSQL limits. Most of the original schema was compatible, but more adjustments could be made if demo data retention were not required (this will be discussed later).
The limitations we worked around relate to supported data types in keys:

NUMERIC type is currently unsupported in keys:

We edited applications/datamodel/entitydef/product-entitymodel.xml, changing the data type of minimumOrderQuantity from fixed-point (NUMERIC(18,6)) to floating-point (float). These two types are not logically identical, but demo data uses decimal. The logic still works correctly here (only “greater than” comparisons used).

We edited framework/entity/fieldtype/fieldtypepostgres.xml to change the <field-type-def type="numeric" sql-type="NUMERIC(20,0)" java-type="Long"/> definition to <field-type-def type="numeric" sql-type="bigint" java-type="Long"/>. Using bigint here is actually more appropriate than NUMERIC(20,0).

VARCHAR is supported in keys, but the maximum size is smaller than normal VARCHAR:

We edited framework/entity/fieldtype/fieldtypepostgres.xml to change the <field-type-def type="email" sql-type="VARCHAR(320)" java-type="String"/> definition to <field-type-def type="email" sql-type="VARCHAR(220)" java-type="String"/>.

After these changes, the schema could be successfully created during data loading (detailed in the next section). We used the -l drop-constraints option in the OFBiz loader utility to ensure no tables were created using foreign key constraints, which are currently unsupported in Aurora DSQL.

As mentioned earlier, we could optimize the schema further. A notable change we avoided was converting ID field data types to UUIDs, common in ORM-based applications. Since we are using demo data, we would need to modify all keys (and references) in the demo data to apply UUID types. Instead, we kept the VARCHAR-based keys and loaded the demo data unchanged.

### Loading Demo Data

The OFBiz data loader (see OFBiz Technical Setup Guide) originally performed large numbers of inserts before each commit, causing some transactions to exceed Aurora DSQL’s transaction size limits. To fix this, we edited the storeAll() method in
`framework/entity/src/main/java/org/apache/ofbiz/entity/GenericDelegator.java`
to batch data loading into groups of 1,000 rows per commit.

```java

              numberChanged += this.store(toStore);
                 }
             }
+            // commit every 1000 rows to conform to DSQL transaction limits
+            if (numberChanged%1000 == 0) {
+                if (Debug.verboseOn()) Debug.logVerbose("Committing at 1000 row threshold. beganTransaction=" + beganTransaction, module);
+                TransactionUtil.commit();
+                beganTransaction = TransactionUtil.begin();
+            }
         }
         TransactionUtil.commit(beganTransaction);
         return numberChanged;
```

**Note:** Aurora DSQL’s actual transaction size limit is larger than 1,000 rows, but we chose this smaller value to observe batching behavior more easily after the change.
We also fixed another interesting bug during lookup. Part of the loading logic calls the findKey() function in
`framework/entity/src/main/java/org/apache/ofbiz/entity/util/EntityCrypto.java`
to check for the existence of an encryption key in the database. If the key is missing at the expected location, loading stops with an error. When testing with Aurora DSQL, this invariant always triggered. After investigation, we found a general logic bug causing behavior differences between read committed and repeatable read isolation levels. The findKey() method was called without starting a new transaction, so it only saw data as of the current transaction’s snapshot. Since the snapshot was from before the corresponding INSERT committed, findKey() could not see the newly inserted row. Under read committed isolation, the call works fine because the session sees committed data after the transaction start.

To fix this, we start a new transaction whenever findKey() is called:

```java

GenericValue keyValue = null;
     try {
+        try {
+            keyValue = TransactionUtil.doNewTransaction(() -> {
+                return EntityQuery.use(delegator).from("EntityKeyStore").where("keyName", hashedKeyName).queryOne();
+            }, "checking encrypted key", 0, true);
+        } catch (GenericEntityException e) {
+            throw new EntityCryptoException(e);
+        }
     } catch (GenericEntityException e) {
         throw new EntityCryptoException(e);
     }
```

We loaded initial seed and demo data following instructions in the OFBiz Technical Setup Guide.

Handling OCC aborts in transaction command pattern
OFBiz submits transactions to the database using a command pattern — with a Callable interface — simplifying Aurora DSQL transaction handling. Other applications (outside OFBiz) that also encapsulate transactions this way can manage retries similarly because transaction boundaries are clear. For illustration, all we add here is a retry loop around the transaction execution, implemented in
`framework/entity/src/main/java/org/apache/ofbiz/entity/transaction/TransactionUtil.java` in the InTransaction.call() method.

```java

protected InTransaction(Callable<V> callable, String ifErrorMessage, int timeout, boolean printException) {
        this.callable = callable;
        this.ifErrorMessage = ifErrorMessage;
        this.timeout = timeout;
        this.printException = printException;
    }

    public V call() throws GenericEntityException {
+       boolean transactionComplete = false;
+       while (!transactionComplete) {
            boolean tx = TransactionUtil.begin(timeout);
            Throwable transactionAbortCause = null;
            try {
                try {
                    return callable.call();
                } catch (Throwable t) {
                    while (t.getCause() != null) {
                        t = t.getCause();
                    }
                    throw t;
                }
            } catch (Error | RuntimeException e) {
                transactionAbortCause = e;
                throw e;
            } catch (Throwable t) {
                transactionAbortCause = t;
+               if (t instanceof SQLException && (((SQLException) t).getSQLState().startsWith("OC")
+                       || ((SQLException) t).getSQLState().equals("40001"))) {
+                   if (Debug.verboseOn()) {
+                       Debug.logVerbose("Transaction abort, reason: " + t, module);
+                       Debug.logVerbose("Retrying on OCC conflict", module);
+                   }
+                   // Rollback to recover transaction state and loop again
+                   TransactionUtil.rollback(tx, ifErrorMessage, transactionAbortCause);
+                   continue;
+               } else {
                    throw new GenericEntityException(t);
+               }
            } finally {
                if (transactionAbortCause == null) {
                    TransactionUtil.commit(tx);
+                   transactionComplete = true;
                } else {
                    if (printException) {
                        Debug.logError(transactionAbortCause, module);
                    }
                    TransactionUtil.rollback(tx, ifErrorMessage, transactionAbortCause);
+                   transactionComplete = true;
+               }
            }
        }
+       throw new GenericEntityException("Unexpected state in transaction handler");
    }
}
```

**Note:** For production applications, we also recommend adding exponential backoff and jitter to improve stability under high contention.
Existing applications that do not model transactions this clearly may require deeper analysis to handle OCC aborts properly. We plan to write a future post on this topic after collecting more customer experience data to identify common patterns.

## Conclusion

In this article, we guided you through various changes made to a complex existing application. We made minor schema modifications to use supported data types, created new logic to properly interact with Aurora DSQL’s authentication mechanism, implemented batching commits during data loading, and adapted transaction handling to work correctly with OCC semantics.

While this proof of concept focuses more on technical feasibility than production readiness, it demonstrates that Aurora DSQL can support complex database applications like OFBiz with reasonable migration effort. This test confirms that even complex schemas can be adjusted to operate with Aurora DSQL, including improved security and scalability features, through targeted adjustments to authentication, transaction handling, and schema design.
These are also key focus areas when planning your own migrations to Aurora DSQL:

Use supported data types in schemas, including keys

Implement IAM-based authentication logic required by Aurora DSQL

Ensure transaction sizes remain within Aurora DSQL limits

Implement idempotent, retryable transactions

If you are working with existing applications on Aurora DSQL, we invite you to share your experiences and observations in the comments!

**About the Author**
James Morle is a Principal Engineer at Amazon Web Services and has been deeply involved in the design and implementation of Aurora DSQL since its inception. He has experience designing and building large-scale databases since the early 1990s.