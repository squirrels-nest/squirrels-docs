# Best Practices for Using REST API Response

Here are some best practices around using the response of the parameters and dataset API.

1. If choosing to cache the response, the cache key should incorporate all query parameters, the "x-minor-version" header, and the "x-num-decimals" header. And coordinate with the data team to understand how frequently the back-end data changes to decide on an appropriate time-to-live for the cache. However, note that there is some level of caching done by the squirrels project already.
2. Sometimes, the set of dataset columns can be different based on query parameters provided. If you are uploading the results to a database table with a fixed schema, then columns in the database table and not the dataset API response should be null. Also, columns in the dataset API response but not the database table should be ignored.
3. The standard for new minor version of the dataset API should be that no existing columns are removed (though this is not the standard for new major versions). However, the ordering of the columns is not guaranteed. Thus when processing the result, use columns by name and not by its order.
