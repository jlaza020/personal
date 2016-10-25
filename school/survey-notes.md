## Module 4 - RDBMS and SQL, Part 1

SQL code to go recursive query without loops:

	WITH	RECURSIVE	SUP_EMP(SupSsn, EmpSsn) AS
	        (SELECT		SupervisorSsn, Ssn
	        FROM		EMPLOYEE
			WHERE		SupervisorSsn IS NULL  

			UNION ALL  

			SELECT		E.Ssn, S.SupSsn
			FROM		EMPLOYEE AS E JOIN SUP_EMP AS S ON E.Supervisor_Sssn = S.EmpSsn)
	SELECT * FROM SUP_EMP

Mandatory vs. optional clauses in SELECT statement:
  
	SELECT			// mandatory
	FROM 			// mandatory
	WHERE			// optional
	GROUP BY		// optional
	HAVING			// optional
	ORDER BY		// optional

Order in which clauses are processed by Query Optimizer:  

+ FROM
+ WHERE
+ GROUP BY
+ HAVING
+ ORDER BY
+ SELECT

Constraints (Assertions) and Triggers:

+ Constraints as Assertions (AKA CHECK constraints)
	+ built into the DB structure (can be part of CREATE TABLE)
	+ simple rules
	+ occur before data is changed
	+ more efficient than triggers
	+ causes hard error if violated
	+ does not have full power of SQL
	+ must be written using positive logic (tell what conditions *pass*, not fail)
	+ column-level constraint (reference single column)
	+ table-level constraint (compare columns within same table)
	+ ex: start-date <= end-date
+ Triggers
	+ not built into DB structure
	+ can happen before or after data is changed
	+ less efficient
	+ choose how to error out if violated
	+ has full power of SQL
	+ separate object from table, but *bound* to table
	+ fires automatically
	+ 3 parts
		+ event (usually INSERT, UPDATE, DELETE)
		+ condition: when or when to not fire trigger (option)
		+ action: SQL code that does acutal work
	+ can easily spire out of control

How to create a trigger:

	CREATE TRIGGER tr_InetLog_INSERT
	ON InetLog								// table trigger is bound to
	AFTER INSERT							// event that causes trigger to fire
	AS

	/* Code that runs after trigger fires. */
	IF EXISTS (SELECT * FROM inserted WHERE Target = 'AboutUs.htm')
	BEGIN
		UPDATE LogSummary
		SET LogSum_Count = (SELECT COUNT (*) FROM InetLog WHERE Target = 'AboutUs.htm')
		WHERE LogSum_Category = 'About Us'
	END

Views:

+ a stored query (AKA virtual table)
+ can use it anywhere a table can be used
+ can cause MAJOR performance problems if overused

How to create a view:

	CREATE VIEW <database-name>.<view-name> AS SELECT ...

Strategies to optimize views (done underneath the hood):

+ **query modification strategy**: optimizer modifies user query
+ **view materialization strategy**: result set of view is physically stored, like a real table; how to
update?
	+ immediate update strategy
	+ lazy update strategy: update when the query needs it
	+ periodic update strategy

It is possible to update views using DML if certain conditions apply:

+ view doesn't have joins
	+ possible if it does, but it is much more complicated: uses INSTEAD OF trigger to hand-code updates
to the underlying base tables - great for database refactoring
+ view contains PK of base table
+ view contains all the NOT NULLable columns that DON'T have DEFAULTs
+ view doesn't use any aggregate functions

Inline views are derived tables: SELECT statement inside another SELECT. 

Views (inline views as well?) are great for authorization mechanisms.

## Module 7 - Database Design

Normalization: The process of decomposing unsatisfactory "bad" relations by
breaking up their attributes into smaller relations

+ minimizes redundancy, makes tables independent of one another
+ minimizes anomalies

Different levels of normalization:

+ 2NF, 3NF, BCNF:
	+ based on keys and FDs of a relation schema
+ 4NF:
	+ based on keys, multi-valued dependencies: MVDs
+ 5NF:
	+ based on keys, join dependencies: JDs

Additional properties may be needed to ensure a good relational design
(lossless join, dependency preservation).

Denormalization: The process of storing the join of higher normal form
relations as a base relation - which is in a lower normal form

Different design philosophies:

+ 3NF modeling is good for putting data INTO the db, but hard to get data OUT
+ Dimensional Modeling:
	+ used for presentation layer of data warehouses
	+ breaks up relation into different types: dimension, facts, bridges, etc.
	+ each table type gets a different level of normalization
+ DataVault Modeling:
	+ based on modeling concepts found in nature
	+ pattern design not affected by business changes

A key is a minimal superkey.

A prime attribute is a member of some candidate key.

A non-prime attribute is NOT a member of ANY candidate key.

1NF:

+ attribues can only have simple, atomic values, and they have to be from that
attribute's domain
+ disallows
	+ composite values
	+ multivalued attributes
	+ nested relations: attribute values that look like a table

Full functional dependency: an FD Y->Z where removal of any attribute from Y
means the FD does not hold anymore

2NF:

+ if every non-attribute A in R is fully functional dependent on the PK
+ A 1NF table that has a single column PK is automatically in 2NF
+ Each key in a fully FD becomes the new PK of the tables generated from the decomposed one. All attributes determined by these keys are put into their corresponding tables.

3NF:

+ deals with transitive functional dependencies: a FD X->Z that can be derived
from two FDs X-> and Y->Z
+ A 3NF table that is in 2NF and NO non-prime attribute A in R is transitively
dependent on the primary key. Y can be a candidate key in this case.
+ Each non-prime attribute that determines another attribute will be the new
PK of one of the newly generated tables. The attributes that are determined by
said non-prime attribute will join their respective non-primate attributes as
nonkey attributes.

Normal forms defined informally:

+ 1NF: all attributes depend on the key
+ 2NF: all attributes depend on the whole key
+ 3NF: all attributes depend on nothing but the key


BCNF: 

+ A table is in BCNG if whenever an FD X->A, then X is a superkey of R
+ Stronger than 3NF

Normalization = decoupling.

## Module 10 - Transactions and Concurrency

### Transaction and System Concepts

**operation**: when >1 DML operations are "tied" together in the real world, and have to either succeed or
fail together as a unit.

**Recovery Manager** of DBMS keeps track of these operations:

+ BEGIN_TRANSACTION
+ READ or WRITE
+ END_TRANSACTION

DBMS keeps a log, which is a list of all the transaction operations that hit a DB. Mostly, READ operations
are not written to these log files. The DBMS writes transaction operations to the log file immediately
when they happen. The log file is append-only.

Log files are backed up separately, like "mini-backups".

**Commit point**: when all the transactions are successful and have been recorded in the log. A 
**commit record** is then written to the log, which means that all these changes can be written to the
main data files of the DB.

Log files are on disk, but their changes are first written to memory, then later to disk.

Log buffers hold the last part of the log data. They are written to disk frequently.

**DBMS-Specific Buffer Replacement Policies**: If all the memory buffers become full, the DBMS has to 
figure out which buffers to write to disk so that they can be freed up in memory.

### Transaction Schedules

**Schedule**: a list of execution of transaction operations, in order. May contain operations from
different transactions that are running at the same time.

For Recoverability and Concurrency Control, we are interested in reads, writes, commits, and rollbacks.

Operations will conflict if they have the following 3 conditions:

+ Belong to different transactions.
+ Access the same item.
+ At least one of the operations is a write.

2 operations are conflicting if changing their order alters the final outcome.

#### Characterizing Schedules Based on Recoverability

Once a transaction is committed, it should never be necessary to roll it back - this would violate the
Durability property and is a nonrecoverable schedule.

A **recoverable** schedule is one where: it only commits *after* the transactions that wrote the data 
which it is reading. For example, if transaction A reads uncommitted data written by transactions S, then
S is rolled back, A must also be rolled back. This is called **cascading rollbacks**.

### Transaction Support in SQL

A single SQL statement is always considered to be atomic.

SET TRANSACTION statements specify the Transaction Settings:

+ Access mode: READ ONLY vs. READ WRITE
+ Isolation level: Only affects READs.
	+ Dirty read: Can see uncommitted changes.
	+ Nonrepeatable read: Can see only committed changes, but if you read the same data twice, you may
	get different values.
	+ Repeatable read: Can see only committed changes. You will always get the same value no matter how
	many times you read the same value, BUT you can get new rows in a result set if some other transaction
	inserted them between your reads (phantom rows).
	+ Serializable: Can only see committed changes. You will always get the same value and NO phantom rows
	are allowed.
	+ Snapshot Isolation: When a transaction starts writing data, an old copy of the data is created and
	kept in a separate location. Disallows other transactions from seeing this new value, only allows the
	old value.

Notes:

	Desirable Properties of Transactions is the wrong video.
