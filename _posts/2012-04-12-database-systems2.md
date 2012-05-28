---
layout: post
title: "database systems(2)"
category: 
tags: []
---
{% include JB/setup %}

- Database needs mechanisms to guarantee:
	1) Secrecy: users should not be able to see things they are not supposed to.
	2) Integrity: users should not be able to modify things they are not supposed to.
	3) Availability: users should be able to see and modify things they are allowed to.

- Access Control in SQL
	GRANT privilege-list
	ON table
	TO user_list
	[WITH GRANT OPTION]

- GRANT UPDATE(grade) ON Enrolled TO uroehm
	only the grade column can be updated by user 'uroehm'

- GRANT SELECT ON Enrolled TO jpoon

- If a user has a privilege with the GRANT OPTION, can pass privilege on to other users (with or without passing on the GRANT OPTION)
	GRANT DELETE ON STUDENTS TO Jon WITH GRANT OPTION

- REVOKE: When a privilege is revoked from X, it is also revoked from all users who got it solely from X

- Views can be used to present necessary information, while hiding details in underlying relations.

- CREATE VIEW EnrolledStuds AS SELECT sid, uos_code FROM Enrolled

- Example: GRANTs and VIEWs

		User A: CREATE TABLE Student (sid INT, ... );
				GRANT SELECT ON Student TO B WITH GRANT OPTION;
				/* note: without GRANT OPTION, B cannot pass SELECT privilege on its view on to C */

		User B: CREATE VIEW MyStud AS
				SELECT sid FROM A.Student;
				GRANT SELECT ON MyStud TO C;
		
		User C: SELECT * FROM B.MyStud; -- works 
				SELECT * FROM A.Student; -- does not work
		
- Limitations of SQL Authorization
	1) SQL does not support authorization at a tuple level: we cannot restrict students to see only their own grades
	2) With the growth in web access to databases, database accesses come primarily from application servers: end users dont have database user ids, they are all mapped to the same database user id
	3) All end-users of an application may be mapped to a single database user

- A database should only store information that is absolutely necessary for the operation of your application

- Integrity Constraint(IC): condition that must be true for every instance of a database.
	A legal instance of a relation is one that satisfies all specified ICs.

- ICs are specified in the database schema
- ICs are checked when the database is modified
- Possible reactions if an IC is violated:
	1) Undoing of a database operation
	2) Abort of the transaction
	3) Execution of 'maintenance' operations to make db legal again

- Referential Integrity: for each tuple in the referring relation whose foreign key value a', there must be a tuple in the referred relation whose primary key value is also a'

- CREATE TABLE Enrolled
(  sid CHAR(10),
   uos CHAR(8),
   grade CHAR(2),
   PRIMARY KEY (sid,uos),
   FOREIGN KEY (sid)
      REFERENCES Student
      ON DELETE CASCADE
      ON UPDATE SET DEFAULT )

    ! Default is NO ACTION (delete/update is rejected)
	! CASCADE (also delete all tuples that refer to deleted tuple)相关联的数据全部删除
	! SET NULL / SET DEFAULT (sets foreign key value of referencing tuple)相关联的数据变成null空值

- Example:
	
		create table Themes
		(
			ThemeID int primary key,
			ThemeName varchar(100),
		)

		create table Users
		(
			UserID int primary key,
			UserName varchar(100),
			ThemeID int constraint Users_ThemeID_FK references Themes(ThemeID) 
		)

		insert into Themes (ThemeID, ThemeName) values (1,'Default')
		insert into Themes (ThemeID, ThemeName) values (2,'Winter')

		insert into Users(UserID, UserName, ThemeID) values (1,'JSmith',null)
		insert into Users(UserID, UserName, ThemeID) values (2,'Ted',1)
		insert into Users(UserID, UserName, ThemeID) values (3,'Mary',2)

		**delete from Themes where ThemeID=2**

		Msg 547, Level 16, State 0, Line 1
		The DELETE statement conflicted with the REFERENCE constraint "FK__Users__ThemeID__23F3538A". 
		The conflict occurred in database "PlayGround", table "dbo.Users", column 'ThemeID'.
		The statement has been terminated.

		-- remove the existing constraint:
		**alter table users drop constraint Users_ThemeID_FK**

		-- re-create it:
		**alter table users add constraint Users_ThemeID_FK foreign key (ThemeID) references Themes(ThemeID)on delete cascade**

		**delete from Themes where ThemeID =2**

		**select * from Users**

		UserID      UserName        ThemeID
		----------- --------------- -----------
		1           JSmith          NULL
		2           Ted             1

		(2 row(s) affected)

		-- remove the existing constraint:
		**alter table users drop constraint Users_ThemeID_FK**

		-- This time, create it with on delete set null:
		**alter table users add constraint Users_ThemeID_FK foreign key (ThemeID) references Themes(ThemeID) on delete set null**

		-- Add our data back in
		**insert into Themes (ThemeID, ThemeName) values (2,'Winter')**
		**insert into Users(UserID, UserName, ThemeID) values (3,'Mary',2)**

		-- And now delete ThemeID 2 again:
		**delete from Themes where ThemeID =2**

		-- Let's see what we've got:
		select * from Users

		UserID      UserName          ThemeID
		----------- ----------------- -----------
		1           JSmith            NULL
		2           Ted               1
		3           Mary              NULL

		(3 row(s) affected)

- CONSTRAINT maxMarks CHECK (mark between 0 and 100),
  CONSTRAINT rightLecturer CHECK ( empid = (SELECT u.lecturer
                   							FROM UnitOfStudy u
                  							WHERE u.uos_code=uos) )

- CONSTRAINT FK_sid_enrolled FOREIGN KEY (sid)
								REFERENCES Student
								ON DELETE CASCADE,
  CONSTRAINT FK_cid_enrolled FOREIGN KEY (uos)
								REFERENCES UnitOfStudy
								ON DELETE CASCADE,
  CONSTRAINT CK_grade_enrolled CHECK(grade in (‘F’,...)),
  CONSTRAINT PK_enrolled       PRIMARY KEY (sid,uos)      

- NOT DEFERRABLE: The default. It means that every time a database modification occurs, the constraint is checked immediately afterwards.

- DEFERRABLE: Gives the option to wait until a transaction is complete before checking the constraint

- ALTER TABLE table-name 'constraint-modification'
		where constraint-modification is one of:
		ADD CONSTRAINT constraint-name new-constraint 
		DROP CONSTRAINT constraint-name
		RENAME CONSTRAINT old-name TO new-name 
		MODIFY attribute-name domain-constraint

- Assertion: a predicate expressing a condition that we wish the database always to satisfy
	create assertion <assertion-name> check (<condition>)

- When an assertion is made, the system tests it for validity, and tests it again on every update that may violate it.

- This testing may introduce a significant amount of overhead; hence assertion should be used with great care.

- A trigger is a statement that is executed automatically if specified modifications occur to the DBMS

- A trigger specification consists of three parts:
	ON event IF precondition THEN action

- Triggering event can be insert, delete or update
	Triggers on update can be restricted to specific attributes
	==CREATE TRIGGER overdraft-trigger AFTER UPDATE OF balance ON account

- 

