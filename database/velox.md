
# Vectors
**"in-memory format for storing query data during execution"**
A single vector represents multiple rows of a single column. RowVector is used to represent 
a set of rows for multiple columns as well as a set of rows for a single column of type struct.

Each vector has a type, encoding and size. Vector size is the number of rows stored in the vector. 

## Buffers
