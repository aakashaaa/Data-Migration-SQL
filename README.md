# Employee Migration Script

This repository contains a **T-SQL (SQL Server)** script that demonstrates how to create a new table, index it, and populate it with employee data from the `HumanResources` schema (such as in the **AdventureWorks sample database**).

The script is written with **best practices in mind**—using transactions, error handling, and indexing—to ensure **data integrity, reliability, and performance**.

---

## 📖 What Does the Script Do?

1. **Drop existing table (if any)**
   Ensures a clean slate by removing the `EmployeeMigration` table if it already exists.

   ```sql
   drop table if exists employeemigration;
   ```

2. **Create a new table**
   Defines a new table `EmployeeMigration` with the following columns:

   * `EmployeeID` → unique employee identifier
   * `NationalIDNumber` → government-issued ID number
   * `JobTitle` → employee’s role (e.g., Sales Manager, HR Specialist)
   * `Department` → department name (e.g., Sales, HR, IT)
   * `Shift` → working shift (Day, Evening, Night)
   * `HireDate` → date the employee was hired
   * `ModifiedDate` → last time employee’s data was updated

   ```sql
   create table employeemigration(
       employeeid int not null,
       nationalidnumber nvarchar(40) not null,
       jobtitle nvarchar(200),
       department nvarchar(50),
       shift nvarchar(20),
       hiredate datetime,
       modifieddate datetime
   );
   ```

3. **Add an index**
   Creates a **non-clustered index** on `JobTitle` to speed up queries when searching or filtering by job role.

   ```sql
   create nonclustered index ix_jobtitle on employeemigration (jobtitle);
   ```

4. **Insert data with transaction safety**

   * Data is pulled from the `HumanResources` schema (`Employee`, `EmployeeDepartmentHistory`, `Department`, and `Shift` tables).
   * Only employees **hired on or after January 1, 2008** are migrated.
   * Data is inserted inside a **transaction**. If something fails, it rolls back (undoing changes) instead of leaving the database in an inconsistent state.

   ```sql
   begin transaction;
   begin try
       insert into employeemigration (...)
       select ...
       from humanresources.employee e
       inner join humanresources.employeedepartmenthistory edh on e.businessentityid = edh.businessentityid
       inner join humanresources.department d on edh.departmentid = d.departmentid
       inner join humanresources.shift s on edh.shiftid = s.shiftid
       where e.hiredate >= '2008-01-01';
       commit transaction;
   end try
   begin catch
       rollback transaction;
       throw;
   end catch;
   ```

5. **Query the results**
   At the end, you can verify data was migrated correctly:

   ```sql
   select * from employeemigration;
   ```

---

## 🗂️ Why This Script Is Useful

* **Data Migration Example** → Demonstrates how to safely migrate data between schemas/tables.
* **Transaction Handling** → Shows best practices with `TRY...CATCH`, ensuring reliability.
* **Indexing for Performance** → Improves query performance for reporting and analytics.
* **AdventureWorks Demo** → Can be run directly on Microsoft’s free sample database, making it easy to test.

---

## ⚡ How to Run

1. Install SQL Server (2016 or later recommended).
2. Download and restore the **AdventureWorks sample database** (if not already installed).
3. Open **SQL Server Management Studio (SSMS)** or **Azure Data Studio**.
4. Connect to your database instance.
5. Copy-paste the script into a new query window.
6. Execute the script.

✅ If successful, a new table `EmployeeMigration` will be created and populated with employees hired after **2008-01-01**.

---

## 📊 Example Output

| EmployeeID | NationalIDNumber | JobTitle        | Department | Shift   | HireDate   | ModifiedDate |
| ---------- | ---------------- | --------------- | ---------- | ------- | ---------- | ------------ |
| 274        | 509647174        | Marketing Spec. | Marketing  | Day     | 2009-01-14 | 2013-03-25   |
| 285        | 14417807         | HR Manager      | Human Res. | Evening | 2010-05-20 | 2014-06-18   |
| 301        | 658797903        | Sales Rep.      | Sales      | Night   | 2012-07-02 | 2016-08-30   |

---

## ✅ Requirements

* SQL Server **2016+** (for `DROP TABLE IF EXISTS`).
* AdventureWorks or a database with the following tables:

  * `HumanResources.Employee`
  * `HumanResources.EmployeeDepartmentHistory`
  * `HumanResources.Department`
  * `HumanResources.Shift`

---

## 🔒 Error Handling

* **Commit** → If all inserts succeed.
* **Rollback** → If any error occurs.
* **Throw** → Reraises the error for debugging.

This guarantees **no partial migrations**—the database remains consistent.

---

## 📌 License

This project is provided under the **MIT License**. You can freely use, modify, and distribute it.

---
