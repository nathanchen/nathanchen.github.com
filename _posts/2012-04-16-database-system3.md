---
layout: post
title: "Database System(3)"
category: 
tags: []
---
{% include JB/setup %}

- When an event occurs in the real world that changes the state of the enterprise, a program is executed to change the database state in a corresponding way. Such a program is called a transaction.

- What does a transaction do?
	1) Return information from the database
	2) Update the database to reflect the occurrence of a real world event
	3) Cause the occurence of a real world event

- ACID properties
	1) Atomicity: Transaction should either complete or have no effect at all
	2) Consistency: Execution of a transaction in isolation preserves the consistency of the database
	3) Isolation: Although multiple transactions may execute concurrently, each transaction must be unaware of other concurrently executing transactions.
	4) Durability: The effect of a transaction on the database state should not be lost once the transaction has committed.

- 
		public void bookFlight ( String flight_num, Date flight_date, Integer seat_no)
		{
			try 
			{
				/* connect to the database */
				Connection conn = DriverManager.getConnection("jdbc:oracle:thin:@oracle10...”); 
				/* set AUTOCOMMIT off; next SQL statement will start a transaction */
				conn.setAutoCommit(false);
				/* execute the SQL statements within the transaction */
				PreparedStatement stmt = conn.prepareStatement(“SELECT occupied FROM Flight
				WHERE flightNum=? AND flightDate=? AND seat=?”); 
				stmt.setString(1, flight_num); 
				stmt.setDate(2, flight_date);
				stmt.setInteger(3, seat_no);
				ResultSet rset = stmt.executeQuery();
				if ( !rset.empty() && rset.next().getInteger()==0 ) 
				{
					stmt = conn.prepareStatement(“UPDATE Flight SET occupied=TRUE
					WHERE flightNum=? AND flightDate=? AND seat=?”); 
					stmt.setString(1, flight_num); stmt.setDate(2, flight_date); 
					stmt.setInteger(3, seat_no);
					stmt.executeUpdate(); 
				}
				/* COMMIT the transaction */
				conn.commit();
				stmt.close();
				conn.close(); 
			}
			catch (SQLException sqle) 
			{
			/* error handling */
			} 
		}

- A transaction is consistent, assuming the database is in a consistente state initially, when the transaction completes:
	1) All static integrity constraints are satisfied
	2) New state satisfies specifications of transaction
	3) No dynamic constraints have been violated

- Integrity constraints may be declared:
	1) NOT DEFERRABLE: The default. It means that every time a database modification occurs, the constraint is checked immediately afterwards.
	2) DEFERRABLE: Gives the option to wait until a transaction is complete before checking the constraint.

- Serial Execution: transactions execute in sequence. Each one starts after the previous one completes. The execution of each transaction is isolated from all others.

- Serial execution is inadequate from a performance perspective.

- Concurrent execution offers performance benefits:
	1) A computer system has multiple resources capable of executing independently, but
	2) A transaction typically uses only one resources at a time

- 	!  Reading Uncommitted Data (WR conflicts, “dirty reads”): 
	T1: R(A),W(A), 					R(B),W(B),Abort
  	T2:           R(A),W(A),Commit

	!  Unrepeatable Reads (RW conflicts):
	T1: R(A), 						R(A),W(A),Commit
	T2:       R(A),W(A),Commit
	!  Overwriting Uncommitted Data (WW conflicts,„lost updates“): 
	T1: W(A), 						W(B),Commit
	T2:         W(A),W(B),Commit
￼￼￼
- Concurrency Control: The protocol that manages simultaneous operations against a database so that serializability is assured.

- 









