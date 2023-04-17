# Cube store DB

A python-first database for dense and sparse multidimensional data cubes.

For example

```python
import cubestore as cs

with cs.CubeDB("128.0.0.1", "API_KEY") as db:
    # Assign a numpy array to a key
    db["SomeData"] = np.random.rand(100, 100, 100)

    # Store a larger-than-memory array on disk
    db["SomeOtherData"] = cs.empty((100, 100, 100, 100, 100, 100), sparse=False)

    # Use this for storing sparse data which results in a pivot table like structure
    db["SomeSparseData"] = cs.empty((100, 100, 100, 100, 100, 100), sparse=True)

    # Append data to a cube. It is recommended to append to the last dimension.
    print(db["SomeLogs"].shape) # (n, 100)
    db["SomeLogs"].append(np.random.rand(100))

    # Support for named dimensions with string labels
    db["WeatherData"] = cs.empty((5, 3, 10), sparse=True, names=["City", "SensorType", "SensorID"], coords={"SensorType": ["Temperature", "Humidity", "Pressure"]})
    db["Temps"] = db["WeatherData"][:, "Temperature", :]

    db["SalesData"] = cs.empty((5, 8), sparse=False, names=["City", "Product"], coords=[["NY", "LA", "SF", "CHI", "DC"], ["Shoes", "Shirts", "Pants", "Hats", "Socks", "Underwear", "Jackets", "Sweaters"]])

    # You can query the data like numpy arrays
    print(db["SomeData"][0, 0, 0])
    print(db["SomeData"][0])
    print(db["SomeData"][0:10])
    print(db["SomeData"][0:10, 0:10])

    # You can run preprocessing on the data on the server to save bandwidth. It also supports broadcasting in numpy style.
    total_sum = (db["SomeData"] + db["SomeOtherData"]).sum(axis=(0, 1)).numpy()

    # You can leave the data on the server and use it in other cubes
    db["TotalSalesPerCity"] = db["SalesData"].sum(axis=("Product",))
```

The server can be run like this

```bash
docker run -p 3030:3030 -v /path/to/data:/data -e CUBESTORE_API_KEY=API_KEY -e CUBESTORE_DATA_DIR=/data cubestore/cubestore:latest
```

or this

```bash
python -m cubestore --api-key API_KEY --data-dir /path/to/data
```
