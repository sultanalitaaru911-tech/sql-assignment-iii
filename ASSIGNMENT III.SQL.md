ASSIGNMENT III: SQL TABLE MANIPULATION 

Project: Library Management System 

DBMS: PostgreSQL 

# BEFORE RUNNING: 

1. Replace yourname and studentid in the database name below. 

2. Run CREATE DATABASE as a separate statement while connected to an existing database. 

PostgreSQL does not allow CREATE DATABASE inside a transaction. 

3. In pgAdmin, connect to the newly created database. 

4. Run the remaining script from Section 2 onward. 

5. Run statements in order. Some UPDATE/ALTER/INSERT tasks change the database state. 

- */ 

- -- ========================================================= 

# -- SECTION 1: DATABASE CREATION 

- -- Replace the placeholders with your own name and student ID. 

- -- Run this statement separately, then connect to that database. 

- -- ========================================================= 

CREATE DATABASE yourname_studentid_library_management; 

-- ========================================================= 

# -- SECTION 2: TABLE CREATION AND CONSTRAINTS 

- -- Run this section only after connecting to the new database. 

- -- Parent tables are created before the child table. 

- -- ========================================================= 

CREATE TABLE members ( 

member_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, 

full_name VARCHAR(100) NOT NULL, 

email VARCHAR(100) NOT NULL UNIQUE, 

phone VARCHAR(20) 

); 

CREATE TABLE books ( 

book_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, 

title VARCHAR(150) NOT NULL, 

author VARCHAR(100) NOT NULL, 

category VARCHAR(50), 

available BOOLEAN NOT NULL DEFAULT TRUE 

); 

CREATE TABLE loans ( 

loan_id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY, 

member_id INTEGER NOT NULL, 

book_id INTEGER NOT NULL, 

loan_date DATE NOT NULL DEFAULT CURRENT_DATE, 

return_date DATE, 

CONSTRAINT fk_loans_members 

FOREIGN KEY (member_id) REFERENCES members(member_id), CONSTRAINT fk_loans_books 

FOREIGN KEY (book_id) REFERENCES books(book_id) 

); 

-- ========================================================= 

- -- SECTION 3: INITIAL DATA INSERTION 

- -- At least five initial records in every table. 

- -- Do not manually insert identity primary-key values. 

- -- ========================================================= 

-- Initial members: 5 records 

INSERT INTO members (full_name, email, phone) VALUES ('John Smith', 'john.smith@example.com', '0781000001'), ('Mary Jones', 'mary.jones@example.com', '0781000002'), 

('Peter James', 'peter.james@example.com', '0781000003'), 

('Sarah Brown', 'sarah.brown@example.com', '0781000004'), 

('David Wilson', 'david.wilson@example.com', '0781000005'); 

-- Initial books: 5 records 

INSERT INTO books (title, author, category, available) VALUES 

('Database Systems', 'Thomas Connolly', 'Technology', TRUE), 

('Introduction to SQL', 'John Smith', 'Technology', TRUE), 

('Computer Networks', 'Andrew Tanenbaum', 'Technology', TRUE), 

('Clean Code', 'Robert C. Martin', 'Programming', TRUE), 

('Operating Systems', 'Abraham Silberschatz', 'Technology', TRUE); 

-- Initial loans: 5 records. 

-- Parent IDs are looked up by unique member email and book title. 

INSERT INTO loans (member_id, book_id, loan_date, return_date) 

VALUES 

( 

(SELECT member_id FROM members WHERE email = 'john.smith@example.com'), (SELECT book_id FROM books WHERE title = 'Database Systems'), 

DATE '2026-10-20', NULL 

), 

( 

(SELECT member_id FROM members WHERE email = 'mary.jones@example.com'), (SELECT book_id FROM books WHERE title = 'Introduction to SQL'), 

DATE '2026-10-21', NULL 

), 

( 

(SELECT member_id FROM members WHERE email = 'peter.james@example.com'), (SELECT book_id FROM books WHERE title = 'Computer Networks'), DATE '2026-10-22', NULL 

), ( 

(SELECT member_id FROM members WHERE email = 'sarah.brown@example.com'), (SELECT book_id FROM books WHERE title = 'Clean Code'), 

DATE '2026-10-23', NULL 

), ( 

(SELECT member_id FROM members WHERE email = 'david.wilson@example.com'), (SELECT book_id FROM books WHERE title = 'Operating Systems'), DATE '2026-10-24', NULL 

); 

-- Verify the initial record counts (each should be at least 5). SELECT COUNT(*) AS member_count FROM members; SELECT COUNT(*) AS book_count FROM books; SELECT COUNT(*) AS loan_count FROM loans; 

-- Optional: inspect the initial tables. SELECT * FROM members ORDER BY member_id; SELECT * FROM books ORDER BY book_id; SELECT * FROM loans ORDER BY loan_id; 

-- ========================================================= 

-- SECTION 4: SELECT TASKS 

-- ========================================================= 

-- Q1.1: Basic Data Retrieval 

-- Objective: Show selected member columns only. 

SELECT full_name, email 

FROM members; 

-- Q1.2: Conditional Data Retrieval -- Objective: Show books in the Technology category. SELECT title, author, category 

FROM books 

WHERE category = 'Technology'; 

-- Q1.3: Sorted Data Retrieval -- Objective: List all books alphabetically by title. 

SELECT title, author, category 

FROM books 

ORDER BY title ASC; 

-- ========================================================= 

-- SECTION 5: JOIN TASKS 

-- ========================================================= 

-- Q2.1: INNER JOIN 

-- Objective: Show loans that match an existing member. 

SELECT l.loan_id, m.full_name, l.loan_date, l.return_date 

FROM loans AS l 

INNER JOIN members AS m 

ON l.member_id = m.member_id 

ORDER BY l.loan_id; 

-- Q2.2: LEFT JOIN 

-- Objective: Show every member, including members with no loan. 

SELECT m.member_id, m.full_name, l.loan_id, l.loan_date 

FROM members AS m 

LEFT JOIN loans AS l 

ON m.member_id = l.member_id 

ORDER BY m.member_id, l.loan_id; 

-- Q2.3: Multi-Table JOIN -- Objective: Produce a report showing member, book, and loan details. SELECT l.loan_id, m.full_name, b.title AS book_title, l.loan_date, l.return_date FROM loans AS l INNER JOIN members AS m ON l.member_id = m.member_id 

INNER JOIN books AS b ON l.book_id = b.book_id 

ORDER BY l.loan_date; 

-- ========================================================= 

-- SECTION 6: UPDATE TASKS AND VERIFICATION 

-- Run the preview SELECT before each UPDATE. 

-- ========================================================= 

-- Q3.1: Update One Record -- Objective: Change one member's phone number. 

SELECT member_id, full_name, email, phone 

FROM members 

WHERE email = 'john.smith@example.com'; 

UPDATE members 

SET phone = '0781999001' 

WHERE email = 'john.smith@example.com'; 

-- Verify Q3.1 

SELECT member_id, full_name, email, phone 

FROM members 

WHERE email = 'john.smith@example.com'; 

-- Q3.2: Update Multiple Records -- Objective: Mark all Technology books as available. 

SELECT book_id, title, category, available 

FROM books 

WHERE category = 'Technology'; 

UPDATE books 

SET available = TRUE 

WHERE category = 'Technology'; 

-- Verify Q3.2 SELECT book_id, title, category, available 

FROM books 

WHERE category = 'Technology' 

ORDER BY book_id; 

-- Q3.3: Conditional Update 

- -- Objective: Record the return date for John's Database Systems loan, 

-- only if it has not already been returned. 

SELECT l.loan_id, m.full_name, b.title, l.return_date 

FROM loans AS l 

JOIN members AS m ON m.member_id = l.member_id 

JOIN books AS b ON b.book_id = l.book_id 

WHERE m.email = 'john.smith@example.com' 

AND b.title = 'Database Systems' 

AND l.return_date IS NULL; 

UPDATE loans AS l 

SET return_date = DATE '2026-10-26' 

FROM members AS m, books AS b 

WHERE l.member_id = m.member_id 

AND l.book_id = b.book_id 

AND m.email = 'john.smith@example.com' 

AND b.title = 'Database Systems' 

AND l.return_date IS NULL; 

-- Verify Q3.3 

SELECT l.loan_id, m.full_name, b.title, l.loan_date, l.return_date 

FROM loans AS l 

JOIN members AS m ON m.member_id = l.member_id 

JOIN books AS b ON b.book_id = l.book_id 

WHERE m.email = 'john.smith@example.com' 

AND b.title = 'Database Systems'; 

-- ========================================================= 

# -- SECTION 7: ALTER TABLE TASKS AND VERIFICATION 

-- These changes intentionally alter the schema. 

-- ========================================================= 

-- Q4.1: Rename a Column -- Objective: Rename category to book_category. ALTER TABLE books 

RENAME COLUMN category TO book_category; 

-- Verify Q4.1: existing category data should still be present. SELECT book_id, title, book_category FROM books ORDER BY book_id; 

-- Q4.2: Change a Column's Data Type -- Objective: Increase the phone column's allowed character length. ALTER TABLE members ALTER COLUMN phone TYPE VARCHAR(30); 

-- Verify Q4.2: inspect the structure in pgAdmin's Columns view, -- and verify existing values remain readable. SELECT member_id, full_name, phone 

FROM members 

ORDER BY member_id; 

-- Q4.3: Add a Column -- Objective: Add an ISBN field to store a book's standard identifier. ALTER TABLE books 

ADD COLUMN isbn VARCHAR(20); 

-- Verify Q4.3 SELECT book_id, title, isbn 

FROM books 

ORDER BY book_id; 

-- ========================================================= 

-- SECTION 8: ADDITIONAL INSERT TASKS AND VERIFICATION 

-- ========================================================= 

-- Q5.1: Insert a Single Record -- Objective: Add one member without manually assigning member_id. 

INSERT INTO members (full_name, email, phone) 

VALUES ('Grace Uwimana', 'grace.uwimana@example.com', '0781999006'); 

-- Verify Q5.1 

SELECT member_id, full_name, email, phone 

FROM members 

WHERE email = 'grace.uwimana@example.com'; 

-- Q5.2: Insert Multiple Records 

-- Objective: Add several books without manually assigning book_id. 

-- book_category is used because Q4.1 renamed the column. 

INSERT INTO books (title, author, book_category, available, isbn) VALUES 

('The Pragmatic Programmer', 'Andrew Hunt and David Thomas', 'Programming', TRUE, '9780135957059'), 

('Learning PostgreSQL', 'Richard Blum', 'Technology', TRUE, '9781783989188'), 

('Algorithms Unlocked', 'Thomas H. Cormen', 'Technology', TRUE, '9780262518802'); 

-- Verify Q5.2 

SELECT book_id, title, author, book_category, available, isbn 

FROM books 

WHERE title IN ( 'The Pragmatic Programmer', 'Learning PostgreSQL', 'Algorithms Unlocked' ) ORDER BY title; 

-- Q5.3: Insert a Related Record -- Objective: Add a loan for an existing member and existing book. -- The parent IDs are looked up instead of manually assigned. INSERT INTO loans (member_id, book_id, loan_date, return_date) VALUES ( (SELECT member_id FROM members WHERE email = 'grace.uwimana@example.com'), (SELECT book_id FROM books WHERE title = 'The Pragmatic Programmer'), DATE '2026-10-26', 

NULL ); -- Verify Q5.3 with a JOIN. SELECT l.loan_id, m.full_name, b.title AS book_title, l.loan_date, l.return_date FROM loans AS l JOIN members AS m ON m.member_id = l.member_id JOIN books AS b ON b.book_id = l.book_id WHERE m.email = 'grace.uwimana@example.com' AND b.title = 'The Pragmatic Programmer'; 

-- ========================================================= 

-- FINAL VERIFICATION 

-- Confirm the final row counts and inspect the final data. 

-- ========================================================= 

SELECT COUNT(*) AS final_member_count FROM members; SELECT COUNT(*) AS final_book_count FROM books; SELECT COUNT(*) AS final_loan_count FROM loans; 

SELECT * FROM members ORDER BY member_id; SELECT * FROM books ORDER BY book_id; SELECT * FROM loans ORDER BY loan_id; 

