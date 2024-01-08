
# Structured vs. Unstructured Data

A repository for storing structured data is called a data warehouse. A repository for storing unstructured data is called a data lake

# Transactional and Analytical Processing

## OLTP:
主要是insert
Typically row-major
INSERT INTO RideTable(RideID, Username, DriverID, City, Month, Price)
VALUES ('10', 'memelord', '3932839', 'Stanford', 'July', '20.4');

ACID (Atomicity, Consistency, Isolation, Durability)


## OLAP:

Mostly SELECT
Typically column-major
SELECT AVG(Price)
FROM RideTable
WHERE City = 'Stanford' AND Month = 'July';


![image](https://github.com/spevenhe/Study/assets/42630862/5c89eef8-aa04-41e2-a138-4638027cfa18)

ETL
![image](https://github.com/spevenhe/Study/assets/42630862/58fa710f-9ef3-4fea-bcd4-5e78cc5592d8)


曾经有个想法，etl，存所有数据到data lake，不维护schema， 使用的时候再load。但是数据规模大之后就不行了，搜索原始数据效率过低
