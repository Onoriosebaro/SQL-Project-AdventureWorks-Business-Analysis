
# SQL Project: AdventureWorks Business Analysis

## Project Overview

Hey there! Welcome to my SQL project. I got to dive into the classic AdventureWorks database to solve some common business problems. It was a great chance to show off my SQL skills! My main goal was to write clean, efficient queries to dig up specific info, run some calculations, and organize data in useful ways.

### Skills Used
- **Complex Joins** (INNER, LEFT)
- **Conditional Logic** (CASE statements)
- **Data Filtering and Sorting**
- **Aggregate Functions** (COUNT, SUM)
- **Date and String Manipulation** (YEAR, LEFT, CONCAT)
- **Common Table Expressions** (CTE) and Window Functions (ROW_NUMBER)
- **View Creation** (CREATE VIEW)

## The Challenges & My Solutions

Here are the challenges I was given and the SQL code I used to figure them out.

1. **Finding the Right Products**  
   **The Ask:** The team wanted a product list, but with a few rules: nothing Red, Silver/Black, or White. The price had to be between £75 and £750. Oh, and I had to rename a column to 'Price' and sort the list from most to least expensive.  
   **SQL Solution:**
   ```sql
   SELECT
       Name,
       Color,
       ListPrice,
       StandardCost AS Price
   FROM
       Production.Product
   WHERE
       Color IS NOT NULL
       AND Color NOT IN ('Red', 'Silver/Black', 'White')
       AND ListPrice BETWEEN 75 AND 750
   ORDER BY
       ListPrice DESC;
   ```

2. **Finding Specific Employee Groups**  
   **The Ask:** I needed to find two groups of employees: men born from 1962-1970 who were hired after 2001, and women born from 1972-1975 who were hired in 2001 or 2002.  
   **SQL Solution:**
   ```sql
   -- Query for Male Employees
   SELECT Gender, BirthDate, HireDate
   FROM HumanResources.Employee
   WHERE Gender = 'M'
     AND YEAR(BirthDate) BETWEEN 1962 AND 1970
     AND YEAR(HireDate) > 2001
   UNION ALL
   -- Query for Female Employees
   SELECT Gender, BirthDate, HireDate
   FROM HumanResources.Employee
   WHERE Gender = 'F'
     AND YEAR(BirthDate) BETWEEN 1972 AND 1975
     AND YEAR(HireDate) BETWEEN 2001 AND 2002;
   ```

3. **Listing the Priciest 'BK' Products**  
   **The Ask:** Find the top 10 most expensive products whose product number starts with 'BK'.  
   **SQL Solution:**
   ```sql
   SELECT TOP 10
       ProductID,
       Name,
       Color
   FROM
       Production.Product
   WHERE
       ProductNumber LIKE 'BK%'
   ORDER BY
       StandardCost DESC;
   ```

4. **Matching Up Contacts**  
   **The Ask:** Pull a list of contacts where the first four letters of their last name also happened to be the first four letters of their email address.  
   **SQL Solution:**
   ```sql
   SELECT
       p.FirstName,
       p.LastName,
       ea.EmailAddress,
       CASE
           WHEN LEFT(p.FirstName, 1) = LEFT(p.LastName, 1)
           THEN CONCAT(p.FirstName, ' ', p.LastName)
           ELSE NULL
       END AS FullName,
       CASE
           WHEN LEFT(p.FirstName, 1) = LEFT(p.LastName, 1)
           THEN LEN(CONCAT(p.FirstName, ' ', p.LastName))
           ELSE NULL
       END AS FullNameLength
   FROM
       Person.Person AS p
   INNER JOIN
       Person.EmailAddress AS ea ON p.BusinessEntityID = ea.BusinessEntityID
   WHERE
       LEFT(p.LastName, 4) = LEFT(ea.EmailAddress, 4);
   ```

5. **Finding Products That Take Longer to Make**  
   **The Ask:** Find all product subcategories that take, on average, 3 days or more to manufacture.  
   **SQL Solution:**
   ```sql
   SELECT DISTINCT
       psc.Name,
       p.DaysToManufacture
   FROM
       Production.ProductSubcategory AS psc
   LEFT JOIN
       Production.Product AS p ON p.ProductSubcategoryID = psc.ProductSubcategoryID
   WHERE
       p.DaysToManufacture >= 3;
   ```

6. **Sorting Products into Value Tiers**  
   **The Ask:** Label products based on their price and filter to only show Black, Silver, or Red products.  
   **SQL Solution:**
   ```sql
   SELECT
       Name,
       ListPrice,
       Color,
       CASE
           WHEN ListPrice < 200 THEN 'Low Value'
           WHEN ListPrice BETWEEN 201 AND 750 THEN 'Mid Value'
           WHEN ListPrice BETWEEN 751 AND 1250 THEN 'Mid to High Value'
           ELSE 'Higher Value'
       END AS ProductSegmentation
   FROM
       Production.Product
   WHERE
       Color IN ('Black', 'Silver', 'Red');
   ```

7. **Counting Job Titles**  
   **The Ask:** How many unique job titles do we have for our employees?  
   **SQL Solution:**
   ```sql
   SELECT
       COUNT(DISTINCT JobTitle) AS DistinctJobTitleCount
   FROM
       HumanResources.Employee;
   ```

8. **Calculating Age When Hired**  
   **The Ask:** For every employee, figure out how old they were when they joined the company.  
   **SQL Solution:**
   ```sql
   SELECT
       BusinessEntityID,
       DATEDIFF(YEAR, BirthDate, HireDate) AS AgeAtHiring
   FROM
       HumanResources.Employee;
   ```

9. **Seeing Who Gets a Service Award Soon**  
   **The Ask:** Who's getting a long-service award (for 20 years of work) within the next five years?  
   **SQL Solution:**
   ```sql
   SELECT
       COUNT(*) AS EligibleForAward
   FROM
       HumanResources.Employee
   WHERE
       DATEDIFF(YEAR, HireDate, GETDATE()) BETWEEN 15 AND 19;
   ```

10. **Calculating Years Left Until Retirement**  
    **The Ask:** Assuming retirement is at age 65, how many more years does each employee have left to work?  
    **SQL Solution:**
    ```sql
    SELECT
        BusinessEntityID,
        DATEDIFF(YEAR, BirthDate, GETDATE()) AS CurrentAge,
        65 - DATEDIFF(YEAR, BirthDate, GETDATE()) AS YearsToRetirement
    FROM
        HumanResources.Employee;
    ```

11. **Rolling Out a New Price and Commission Plan**  
    **The Ask:** Apply a new pricing rule based on color and figure out the commission.  
    **SQL Solution:**
    ```sql
    WITH NewPriceCTE AS (
        SELECT
            ProductID,
            Name,
            Color,
            StandardCost,
            CASE
                WHEN Color = 'White' THEN StandardCost * 1.08
                WHEN Color = 'Yellow' THEN StandardCost * 0.925
                WHEN Color = 'Black' THEN StandardCost * 1.172
                WHEN Color IN ('Multi', 'Silver', 'Silver/Black', 'Blue') THEN SQRT(StandardCost) * 2
                ELSE StandardCost
            END AS NewPrice
        FROM
            Production.Product
    )
    SELECT
        *,
        NewPrice * 0.375 AS Commission
    FROM
        NewPriceCTE;
    ```

12. **Getting the Scoop on Salespeople**  
    **The Ask:** Create a list of all the salespeople including their full name, hire date, sick leave hours, sales quota, and region.  
    **SQL Solution:**
    ```sql
    SELECT
        p.FirstName,
        p.LastName,
        e.HireDate,
        e.SickLeaveHours,
        st.Name AS Region,
        sp.SalesQuota
    FROM
        Sales.SalesPerson AS sp
    JOIN
        HumanResources.Employee AS e ON sp.BusinessEntityID = e.BusinessEntityID
    JOIN
        Person.Person AS p ON e.BusinessEntityID = p.BusinessEntityID
    JOIN
        Sales.SalesTerritory AS st ON sp.TerritoryID = st.TerritoryID
    ORDER BY
        p.LastName, p.FirstName;
    ```

13. **Building a Big Sales Report**  
    **The Ask:** Create a detailed sales report showing product name, category, salesperson, revenue, month, quarter, and region.  
    **SQL Solution:**
    ```sql
    SELECT
        p.Name AS ProductName,
        pc.Name AS ProductCategoryName,
        psc.Name AS ProductSubcategoryName,
        CONCAT(pp.FirstName, ' ', pp.LastName) AS SalesPerson,
        SUM(sod.LineTotal) AS Revenue,
        DATEPART(month, soh.OrderDate) AS MonthOfTransaction,
        DATEPART(quarter, soh.OrderDate) AS QuarterOfTransaction,
        st.Name AS Region
    FROM
        Sales.SalesOrderHeader AS soh
    JOIN Sales.SalesOrderDetail AS sod ON soh.SalesOrderID = sod.SalesOrderID
    JOIN Production.Product AS p ON sod.ProductID = p.ProductID
    JOIN Production.ProductSubcategory AS psc ON p.ProductSubcategoryID = psc.ProductSubcategoryID
    JOIN Production.ProductCategory AS pc ON psc.ProductCategoryID = pc.ProductCategoryID
    JOIN Sales.SalesPerson AS sp ON soh.SalesPersonID = sp.BusinessEntityID
    JOIN Person.Person AS pp ON sp.BusinessEntityID = pp.BusinessEntityID
    JOIN Sales.SalesTerritory AS st ON sp.TerritoryID = st.TerritoryID
    GROUP BY
        p.Name, pc.Name, psc.Name, pp.FirstName, pp.LastName,
        DATEPART(month, soh.OrderDate), DATEPART(quarter, soh.OrderDate), st.Name
    ORDER BY
        pp.LastName, pp.FirstName;
    ```

14. **Showing Order Details**  
    **The Ask:** Pull all important details for an order: order number, date, total amount, customer, salesperson, and commission percentage.  
    **SQL Solution:**
    ```sql
    SELECT
        soh.SalesOrderNumber,
        soh.OrderDate,
        soh.TotalDue AS OrderAmount,
        soh.CustomerID,
        CONCAT(p.FirstName, ' ', p.LastName) AS SalespersonName,
        sp.CommissionPct AS CommissionPercentage
    FROM
        Sales.SalesOrderHeader AS soh
    JOIN
        Sales.SalesPerson AS sp ON soh.SalesPersonID = sp.BusinessEntityID
    JOIN
        Person.Person AS p ON sp.BusinessEntityID = p.BusinessEntityID;
    ```

15. **Calculating Commissions and Margins**  
    **The Ask:** Figure out commission (14.79% of the standard cost) and margin based on special cost adjustments for different colors.  
    **SQL Solution:**
    ```sql
    SELECT
        ProductID,
        Name,
        Color,
        StandardCost,
        StandardCost * 0.14790 AS Commission,
        (CASE
            WHEN Color = 'Black' THEN StandardCost * 1.22
            WHEN Color = 'Red' THEN StandardCost * 0.88
            WHEN Color = 'Silver' THEN StandardCost * 1.15
            WHEN Color = 'Multi' THEN StandardCost * 1.05
            WHEN Color = 'White' AND StandardCost > 0 THEN (2 * StandardCost) / SQRT(StandardCost)
            ELSE StandardCost
         END) - StandardCost AS Margin
    FROM
        Production.Product;
    ```

16. **Creating a Reusable "Top 5" List**  
    **The Ask:** Create a view showing the top 5 most expensive products for each color.  
    **SQL Solution:**
    ```sql
    CREATE VIEW vTop5ProductsByColor AS
    WITH ProductRankCTE AS (
        SELECT
            ProductID,
            Name,
            Color,
            ListPrice,
            ROW_NUMBER() OVER(PARTITION BY Color ORDER BY ListPrice DESC) AS PriceRank
        FROM
            Production.Product
        WHERE
            Color IS NOT NULL
    )
    SELECT
        ProductID,
        Name,
        Color,
        ListPrice
    FROM
        ProductRankCTE
    WHERE
        PriceRank <= 5;
    ```

## Conclusion

This project was a fantastic opportunity to enhance my SQL skills and tackle real-world business problems using data analysis. Feel free to explore the code and reach out if you have any questions or feedback!

```
