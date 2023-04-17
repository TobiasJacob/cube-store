# Cube store DB

A python-first database for dense and sparse multidimensional data cubes.

Iterate efficently and apply lambdas remotly or on larger than memory datasets in chunks.

```python
import cubestore as cs

with cs.LocalCube("/filepath") as local_db, cs.CubeDB("128.0.0.1", "API_KEY") as remote_db:
    # Using pre-defined cubes
    sales_per_product = 0
    for chunk in local_db["SomeData"].iter(axis=("City", "Product"), chunk_size=(10, None)):
        sales_per_product = sales_per_product + chunk.sum(axis=("City"))

    # Preprocessing data remotly on the server with lambdas
    sales_per_product = 0

    def sum_sales(chunk: np.ndarray):
        return chunk.sum(axis=("City"))

    for chunk_result in remote_db["SomeData"].iter(axis=("City", "Product"), chunk_size=(10, None), lambda=sum_sales):
        sales_per_product = sales_per_product + chunk_result

    # Copy data from remote cube to local cube
    local_db["SomeData"] = cs.full_like(remote_db["SomeData"], 0)
    for chunk in remote_db["SomeData"].iter(chunk_size=(..., 100)):
        local_db["SomeData"].append(chunk)
```

If lambdas are too insecure, you can work with predefined functions as well.

```python
import cubestore as cs

with cs.CubeDB("128.0.0.1", "API_KEY") as db:
    # Assign a numpy array to a key
    db["SomeData"] = np.random.rand(100, 100, 100)

    # Store a larger-than-memory array on disk
    db["SomeOtherData"] = cs.zeros((100, 100, 100, 100, 100, 100), sparse=False, dtype=cs.int32)

    # Use this for storing sparse data which results in a pivot table like structure
    db["SomeSparseData"] = cs.zeros((100, 100, 100, 100, 100, 100), sparse=True)

    # Append data to a cube. It is recommended to append to the last dimension.
    print(db["SomeLogs"].shape) # (n, 100)
    db["SomeLogs"].append(np.random.rand(100))

    # Support for named dimensions with string labels
    db["WeatherData"] = cs.zeros((5, 3, 10), sparse=True, names=["City", "SensorType", "SensorID"], coords={"SensorType": ["Temperature", "Humidity", "Pressure"]})
    db["Temps"] = db["WeatherData"][:, "Temperature", :]

    db["SalesData"] = cs.zeros((5, 8), sparse=False, names=["City", "Product"], coords=[["NY", "LA", "SF", "CHI", "DC"], ["Shoes", "Shirts", "Pants", "Hats", "Socks", "Underwear", "Jackets", "Sweaters"]])

    # You can query the data like numpy arrays
    print(db["SomeData"][0, 0, 0])
    print(db["SomeData"][0])
    print(db["SomeData"][0:10])
    print(db["SomeData"][0:10, 0:10])

    # You can run preprocessing on the data on the server to save bandwidth. It also supports broadcasting in numpy style.
    total_sum = (db["SomeData"] + db["SomeOtherData"]).sum(axis=(0, 1)).numpy()

    # You can leave the data on the server and use it in other cubes
    db["TotalSalesPerCity"] = db["SalesData"].sum(axis=("Product",))

    # Iterate over data efficiently
    for city, product, sales in db["SalesData"].iter(axis=("City", "Product")):
        print(city, product, sales)

    # You can also iterate over dimensions
    for city, product_sales in db["SalesData"].iter(axis="City"):
        # product_sales is now a numpy array of shape (8,)
        print(city, product_sales)

    # Or iterate in chunks
    for chunk in db["SalesData"].iter(axis=("City", "Product"), chunk_size=(1, 100)):
        print(chunk)

    # You can also iterate over the non-zero values only
    for city, product, sales in db["SalesData"].iter_sparse():
        print(city, product, sales)

```

Support functions
- `zeros` - create a cube filled with zeros
- `ones` - create a cube filled with ones
- `full` - create a cube filled with a constant value
- `random` - create a cube filled with random values
- `arange` - create a cube filled with a range of values
- `sum` - sum over a dimension
- `prod` - product over a dimension
- `mean` - mean over a dimension
- `std` - standard deviation over a dimension
- `var` - variance over a dimension
- `min` - minimum over a dimension
- `max` - maximum over a dimension
- `argmin` - index of minimum over a dimension
- `argmax` - index of maximum over a dimension
- `cumsum` - cumulative sum over a dimension
- `cumprod` - cumulative product over a dimension
- `+` - add two cubes
- `-` - subtract two cubes
- `*` - multiply two cubes
- `/` - divide two cubes
- `**` - raise to a power
- `abs` - absolute value
- `sqrt` - square root
- `exp` - exponential
- `log` - natural logarithm
- `log2` - base 2 logarithm
- `log10` - base 10 logarithm
- `sin` - sine
- `cos` - cosine
- `tan` - tangent
- `asin` - arc sine
- `acos` - arc cosine
- `atan` - arc tangent
- `sinh` - hyperbolic sine
- `cosh` - hyperbolic cosine
- `tanh` - hyperbolic tangent
- `asinh` - hyperbolic arc sine
- `acosh` - hyperbolic arc cosine
- `atanh` - hyperbolic arc tangent
- `isnan` - check if a value is NaN
- `isinf` - check if a value is infinite
- `isfinite` - check if a value is finite
- `clip` - clip values to a range
- `round` - round to a number of decimal places
- `floor` - round down to the nearest integer
- `ceil` - round up to the nearest integer
- `dot` - dot product of two cubes
- `matmul` - matrix multiplication of two cubes
- `transpose` - transpose a cube
- `swapaxes` - swap two axes
- `reshape` - reshape a cube
- `broadcast_to` - broadcast a cube to a new shape
- `concatenate` - concatenate two cubes
- `stack` - stack two cubes
- `flip` - flip a cube along the specified axis
- `sort` - sort a cube
- `argsort` - sort a cube and return the indices
- `squeeze` - remove single-dimensional entries from the shape of a cube
- `searchsorted` - find indices where elements should be inserted to maintain order
- `resize` - change shape and size of a cube in-place
- `trim_zeros` - trim the leading and/or trailing zeros from a cube or sequence
- `diff` - calculate the n-th discrete difference along the given axis
- `cross` - return the cross product of two (arrays of) vectors
- `histogram` - compute the histogram of a set of data
- `digitize` - return the indices of the bins to which each value in input array belongs
- `unique` - find the unique elements of an array
- `intersect` - find the intersection of two axis of an array
- `union` - find the union of two axis of an array
- `count_nonzero` - count the number of non-zero values in the cube
- `nonzero` - return the indices of the non-zero values in the cube
- `shape` - return the shape of a cube
- `size` - return the size of a cube
- `ndim` - return the number of dimensions of a cube
- `nbytes` - return the number of bytes in the data buffer of a cube
- `fill` - fill the cube with a scalar value
- `flatten` - return a flattened cube
- `T` - return the transpose of the cube
- `strides` - the tuple of bytes to step in each dimension when traversing an array

The server can be run like this

```bash
docker run -p 3030:3030 -v /path/to/data:/data -e CUBESTORE_API_KEY=API_KEY -e CUBESTORE_DATA_DIR=/data cubestore/cubestore:latest
```

or this

```bash
python -m cubestore --api-key API_KEY --data-dir /path/to/data
```
