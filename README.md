#Practical no. 1 

-- Create Database
CREATE DATABASE CollegeDB;

-- Use Database
USE CollegeDB;

-- Create Instructor Table
CREATE TABLE Instructor (
    InstructorID INT PRIMARY KEY,
    Name VARCHAR(50),
    Email VARCHAR(50),
    Department VARCHAR(50)
);

-- Create Student Table
CREATE TABLE Student (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(50),
    Email VARCHAR(50),
    Age INT,
    Address VARCHAR(100)
);

-- Create Course Table
CREATE TABLE Course (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(50),
    Credits INT,
    InstructorID INT,
    FOREIGN KEY (InstructorID) 
    REFERENCES Instructor(InstructorID)
);

-- Create Enrollment Table
CREATE TABLE Enrollment (
    EnrollmentID INT PRIMARY KEY,
    StudentID INT,
    CourseID INT,
    EnrollmentDate DATE,
    FOREIGN KEY (StudentID) 
    REFERENCES Student(StudentID),
    FOREIGN KEY (CourseID) 
    REFERENCES Course(CourseID)
);

-- Insert Instructor Data
INSERT INTO Instructor VALUES
(1,'Dr. Sharma','sharma@gmail.com','Computer Science'),
(2,'Dr. Patil','patil@gmail.com','Information Technology');

-- Insert Student Data
INSERT INTO Student VALUES
(101,'Rahul','rahul@gmail.com',20,'Pune'),
(102,'Sneha','sneha@gmail.com',21,'Mumbai'),
(103,'Amit','amit@gmail.com',19,'Nashik');

-- Insert Course Data
INSERT INTO Course VALUES
(201,'Database Management',4,1),
(202,'Data Structures',3,2);

-- Insert Enrollment Data
INSERT INTO Enrollment VALUES
(1,101,201,'2026-03-10'),
(2,102,202,'2026-03-10'),
(3,103,201,'2026-03-10');

-- Display Tables
SELECT * FROM Instructor;
SELECT * FROM Student;
SELECT * FROM Course;
SELECT * FROM Enrollment;


#Practical No. 2 — Views and Indexes


CREATE DATABASE CollegeDB;

-- View: Student Course Details
CREATE VIEW StudentCourseView AS
SELECT 
    s.StudentID,
    s.Name AS StudentName,
    c.CourseName,
    e.EnrollmentDate
FROM Student s
JOIN Enrollment e ON s.StudentID = e.StudentID
JOIN Course c ON e.CourseID = c.CourseID;

-- View: Course Instructor Details
CREATE VIEW CourseInstructorView AS
SELECT 
    c.CourseID,
    c.CourseName,
    i.Name AS InstructorName,
    i.Department
FROM Course c
JOIN Instructor i ON c.InstructorID = i.InstructorID;

SELECT * FROM StudentCourseView;
SELECT * FROM CourseInstructorView;

-- Update student name using view
UPDATE StudentCourseView
SET StudentName = 'Rahul Sharma'
WHERE StudentID = 101;

-- Delete a record using view
DELETE FROM StudentCourseView
WHERE StudentID = 103;

-- Index on Student Name
CREATE INDEX idx_student_name
ON Student(Name);

-- Index on Course Name
CREATE INDEX idx_course_name
ON Course(CourseName);

-- Index on Enrollment StudentID
CREATE INDEX idx_enrollment_student
ON Enrollment(StudentID);

SELECT * FROM Student WHERE Name = 'Rahul Sharma';

SELECT * FROM Course
WHERE CourseName = 'Database Management';

SHOW INDEX FROM Student;
SHOW INDEX FROM Course;
SHOW INDEX FROM Enrollment;


#Practical No. 3 — SQL Queries and Subqueries

USE CollegeDB;

-- Students above age 20
SELECT * FROM Student
WHERE Age > 20;

-- Students taught by Dr. Sharma
SELECT s.StudentID, s.Name, c.CourseName
FROM Student s
JOIN Enrollment e ON s.StudentID = e.StudentID
JOIN Course c ON e.CourseID = c.CourseID
JOIN Instructor i ON c.InstructorID = i.InstructorID
WHERE i.Name = 'Dr. Sharma';

-- Students sorted alphabetically
SELECT * FROM Student
ORDER BY Name ASC;

-- Courses sorted by credits
SELECT * FROM Course
ORDER BY Credits DESC;

-- Count of students per course
SELECT c.CourseName,
COUNT(e.StudentID) AS StudentCount
FROM Course c
LEFT JOIN Enrollment e
ON c.CourseID = e.CourseID
GROUP BY c.CourseName;

-- Average age of students
SELECT AVG(Age) AS AverageAge
FROM Student;

-- Students enrolled in Database Management
SELECT Name FROM Student
WHERE StudentID IN (
    SELECT StudentID FROM Enrollment
    WHERE CourseID = (
        SELECT CourseID FROM Course
        WHERE CourseName = 'Database Management'
    )
);

-- Students with age greater than average age
SELECT Name, Age FROM Student
WHERE Age > (
    SELECT AVG(Age) FROM Student
);

-- Students with above-average age in high-credit courses
SELECT s.Name, s.Age, c.CourseName
FROM Student s
JOIN Enrollment e ON s.StudentID = e.StudentID
JOIN Course c ON e.CourseID = c.CourseID
WHERE s.Age > (
    SELECT AVG(Age) FROM Student
)
AND c.Credits > 3
ORDER BY s.Name ASC;

#Practical No. 4 — Joins

CREATE DATABASE IF NOT EXISTS OrderDB;

USE OrderDB;

-- Customer Table
CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(50),
    Email VARCHAR(50),
    City VARCHAR(50)
);

-- Orders Table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    Amount DECIMAL(10,2),
    FOREIGN KEY (CustomerID)
    REFERENCES Customer(CustomerID)
);

INSERT INTO Customer VALUES
(1,'Rahul','rahul@gmail.com','Pune'),
(2,'Sneha','sneha@gmail.com','Mumbai'),
(3,'Amit','amit@gmail.com','Nashik'),
(4,'Priya','priya@gmail.com','Delhi');

INSERT INTO Orders VALUES
(101,1,'2026-03-01',500),
(102,2,'2026-03-02',700),
(103,1,'2026-03-05',300),
(104,NULL,'2026-03-06',400);

-- INNER JOIN
SELECT c.CustomerID, c.Name, o.OrderID, o.Amount
FROM Customer c
INNER JOIN Orders o
ON c.CustomerID = o.CustomerID;

-- LEFT JOIN
SELECT c.CustomerID, c.Name, o.OrderID, o.Amount
FROM Customer c
LEFT JOIN Orders o
ON c.CustomerID = o.CustomerID;

-- RIGHT JOIN
SELECT c.CustomerID, c.Name, o.OrderID, o.Amount
FROM Customer c
RIGHT JOIN Orders o
ON c.CustomerID = o.CustomerID;

-- FULL OUTER JOIN using UNION
SELECT c.CustomerID, c.Name, o.OrderID, o.Amount
FROM Customer c
LEFT JOIN Orders o
ON c.CustomerID = o.CustomerID

UNION

SELECT c.CustomerID, c.Name, o.OrderID, o.Amount
FROM Customer c
RIGHT JOIN Orders o
ON c.CustomerID = o.CustomerID;


#Practical No. 6 — Triggers and Cursor

CREATE DATABASE IF NOT EXISTS EmployeeDB;

USE EmployeeDB;

CREATE TABLE Employee (
    EmpID INT PRIMARY KEY,
    Name VARCHAR(50),
    Department VARCHAR(50),
    Salary DECIMAL(10,2)
);

CREATE TABLE Employee_Audit (
    AuditID INT AUTO_INCREMENT PRIMARY KEY,
    EmpID INT,
    ActionType VARCHAR(10),
    ActionDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER $$

-- INSERT Trigger
CREATE TRIGGER emp_insert
AFTER INSERT ON Employee
FOR EACH ROW
BEGIN
    INSERT INTO Employee_Audit(EmpID, ActionType)
    VALUES (NEW.EmpID, 'INSERT');
END $$

-- UPDATE Trigger
CREATE TRIGGER emp_update
AFTER UPDATE ON Employee
FOR EACH ROW
BEGIN
    INSERT INTO Employee_Audit(EmpID, ActionType)
    VALUES (NEW.EmpID, 'UPDATE');
END $$

-- DELETE Trigger
CREATE TRIGGER emp_delete
AFTER DELETE ON Employee
FOR EACH ROW
BEGIN
    INSERT INTO Employee_Audit(EmpID, ActionType)
    VALUES (OLD.EmpID, 'DELETE');
END $$

DELIMITER ;

INSERT INTO Employee VALUES
(1,'Rahul','IT',50000),
(2,'Sneha','HR',40000),
(3,'Amit','IT',45000);

-- Insert
INSERT INTO Employee VALUES
(4,'Priya','Finance',42000);

-- Update
UPDATE Employee
SET Salary = 55000
WHERE EmpID = 1;

-- Delete
DELETE FROM Employee
WHERE EmpID = 2;

SELECT * FROM Employee_Audit;

DELIMITER $$

CREATE PROCEDURE IncreaseSalary()
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE emp_id INT;
    DECLARE dept VARCHAR(50);

    DECLARE emp_cursor CURSOR FOR
    SELECT EmpID, Department FROM Employee;

    DECLARE CONTINUE HANDLER
    FOR NOT FOUND SET done = 1;

    OPEN emp_cursor;

    read_loop: LOOP

        FETCH emp_cursor INTO emp_id, dept;

        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Increase salary by 10% for IT department
        IF dept = 'IT' THEN
            UPDATE Employee
            SET Salary = Salary + (Salary * 0.10)
            WHERE EmpID = emp_id;
        END IF;

    END LOOP;

    CLOSE emp_cursor;

END $$

DELIMITER ;

CALL IncreaseSalary();

SELECT * FROM Employee;


#practical no 7



// Create Collection
db.createCollection("MANISHA");

// Insert Data
db.MANISHA.insertMany([
{
    NAME: "MANISHA",
    CITY: "JUNNAR",
    AGE: 20
},
{
    NAME: "NEHA",
    CITY: "PUNE",
    AGE: 21
},
{
    NAME: "AKANKSHA",
    CITY: "PUNE",
    AGE: 22
},
{
    NAME: "SONAL",
    CITY: "MUMBAI",
    AGE: 25
}
]);

// Finding Data
db.MANISHA.find({ CITY: "MUMBAI" });

// Update Data
db.MANISHA.updateOne(
    { NAME: "SONAL" },
    { $set: { AGE: 26 } }
);

// Delete Data
db.MANISHA.deleteOne({ NAME: "AKANKSHA" });

// Aggregation on Average Age
db.MANISHA.aggregate([
{
    $group: {
        _id: null,
        averageAGE: { $avg: "$AGE" }
    }
}
]);

// Creating Index
db.MANISHA.createIndex({ AGE: 1 });




#practical no 8


// Insert Data
db.posts.insertMany([
{
    user: "amit",
    likes: 120,
    content: "Learning MongoDB #mongodb #database",
    hashtags: ["mongodb", "database"],
    date: "2025-03-01"
},
{
    user: "neha",
    content: "AI is future #AI #ML",
    likes: 200,
    hashtags: ["AI", "ML"],
    date: "2025-03-02"
},
{
    user: "rahul",
    content: "Big data #bigdata #AI",
    likes: 150,
    hashtags: ["bigdata", "AI"],
    date: "2025-03-03"
},
{
    user: "sneha",
    content: "Web trends #web #javascript",
    likes: 90,
    hashtags: ["web", "javascript"],
    date: "2025-03-04"
},
{
    user: "karan",
    content: "Machine Learning #ML #AI",
    likes: 250,
    hashtags: ["ML", "AI"],
    date: "2025-03-05"
}
]);

// Aggregation on Top Trending Hashtags
db.posts.aggregate([
{
    $unwind: "$hashtags"
},
{
    $group: {
        _id: "$hashtags",
        count: { $sum: 1 }
    }
},
{
    $sort: {
        count: -1
    }
},
{
    $limit: 5
}
]);

// Total Likes
db.posts.aggregate([
{
    $group: {
        _id: null,
        totalLikes: { $sum: "$likes" }
    }
}
]);

// Average Likes
db.posts.aggregate([
{
    $group: {
        _id: null,
        averageLikes: { $avg: "$likes" }
    }
}
]);


#practical no 9


CREATE DATABASE college_db;

USE college_db;

CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50)
);

INSERT INTO students VALUES
(1, 'Amit', 'Computer'),
(2, 'Neha', 'Management'),
(3, 'Rahul', 'Engineering');

SELECT * FROM students;

DROP DATABASE college_db;

SHOW DATABASES;

USE college_db;

SHOW TABLES;

SELECT * FROM students;
