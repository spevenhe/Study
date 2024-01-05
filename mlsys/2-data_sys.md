
# Structured vs. Unstructured Data

A repository for storing structured data is called a data warehouse. A repository for storing unstructured data is called a data lake

# Transactional and Analytical Processing

## OLTP:
主要是insert
Typically row-major
INSERT INTO RideTable(RideID, Username, DriverID, City, Month, Price)
VALUES ('10', 'memelord', '3932839', 'Stanford', 'July', '20.4');


## OLAP:

Mostly SELECT
Typically column-major
SELECT AVG(Price)
FROM RideTable
WHERE City = 'Stanford' AND Month = 'July';
