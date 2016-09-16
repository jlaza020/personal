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











