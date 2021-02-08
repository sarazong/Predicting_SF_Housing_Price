# Part 1: W3Schools SQL Lab 

*Introductory level SQL*

--

This challenge uses the [W3Schools SQL playground](http://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all). Please add solutions to this markdown file and submit.

1. Which customers are from the UK? <br>
```python
SELECT CustomerName
FROM Customers 
WHERE Country == 'UK';
```
(Around the Horn, B's Beverages, Consolidated Holidings, Eastern Connection, Island Trading, North/South, and Seven Seas Imports are customers from the UK.)

2. What is the name of the customer who has the most orders? <br>
```python
SELECT c.CustomerID, c.CustomerName, o.CustomerID, COUNT(o.OrderID) 
FROM Customers c 
INNER JOIN Orders o 
ON c.CustomerID = o.CustomerID 
GROUP BY c.CustomerID 
ORDER BY COUNT(o.OrderID) DESC;
```
(Ernst Handel is the customer who has the most orders.)

3. Which supplier has the highest average product price? <br>
```python
SELECT p.SupplierID, AVG(p.Price), s.SupplierID, s.SupplierName 
FROM Products p 
INNER JOIN Suppliers s 
ON p.SupplierID = s.SupplierID 
GROUP BY s.SupplierID 
ORDER BY AVG(p.Price) DESC;
```
(Aux joyeux ecclésiastiques has the hightest average product price of $140.75.)

4. How many different countries are all the customers from? (*Hint:* consider [DISTINCT](http://www.w3schools.com/sql/sql_distinct.asp).) <br>
```python
SELECT COUNT(DISTINCT(Country))
FROM Customers;
```
(The customers are from 21 different countries.)

5. What category appears in the most orders? <br>
```python
SELECT c.CategoryID, c.CategoryName, p.ProductID, COUNT(o.OrderID) 
FROM Categories c 
INNER JOIN Products p 
ON c.CategoryID = p.CategoryID 
INNER JOIN OrderDetails o 
ON p.ProductID = o.ProductID 
GROUP BY c.CategoryID 
ORDER BY COUNT(o.OrderID) DESC;
```
(Dairy products appears in the most orders.)

6. What was the total cost for each order? <br>
```python
SELECT p.ProductID, p.Price, o.Quantity, SUM(p.Price * o.Quantity), o.OrderID 
FROM Products p 
INNER JOIN OrderDetails o 
ON p.ProductID = o.ProductID 
GROUP BY OrderID;
```

7. Which employee made the most sales (by total price)? <br>
```python
SELECT e.EmployeeID, e.LastName, e.FirstName, o.OrderID, SUM(o2.Quantity*p.Price) 
FROM Employees e 
INNER JOIN Orders o 
ON e.EmployeeID = o.EmployeeID 
INNER JOIN OrderDetails o2 
ON o.OrderID = o2.OrderID 
INNER JOIN Products p 
ON o2.ProductID = p.ProductID 
GROUP BY e.EmployeeID 
ORDER BY SUM(o2.Quantity*p.Price) DESC;
```
(Margaret Peacock made the most sales by $105,696)

8. Which employees have BS degrees? (*Hint:* look at the [LIKE](http://www.w3schools.com/sql/sql_like.asp) operator.) <br>
```python
SELECT LastName, FirstName 
FROM Employees 
WHERE Notes LIKE '%BS%';
```
(Janet Leverling and Steven Buchanan have BS degrees.)

9. Which supplier of three or more products has the highest average product price? (*Hint:* look at the [HAVING](http://www.w3schools.com/sql/sql_having.asp) operator.) <br>
```python
SELECT s.SupplierName, p.SupplierID, AVG(p.Price) 
FROM Suppliers s 
INNER JOIN Products p 
ON s.SupplierID = p.SupplierID 
GROUP BY p.SupplierID 
HAVING COUNT(p.SupplierID) >= 3 
ORDER BY AVG(p.Price) DESC;
```
(Tokyo Traders has the highest average product price at $46.)
