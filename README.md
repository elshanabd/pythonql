# pythonql
A LINQ type extension to Python that allows language-integrated queries against relational, XML and JSON data, as well an Python's collections


Python has pretty advanced comprehensions, that cover a big chunk of SQL, to the point where PonyORM was able to build a whole ORM system based on comprehensions. However, group by mechanisms, outerjoins and support for semi-structured data are not handled well at all.


We propose the following extensions to Python( that are implemeneted in this demo preprocessor and query executor):

 - Path expressions. When working with nested data that has varied structure, path expressions are extremely useful. We have modeled our path expression on XPath, however we use a much simplified verison:

  - Child step:  ```for x in data ./``` 
  - Descended step: ```for x in data .//```
  - Filter step: ```for x in data { key == 'green' }```

So we can write path expression in the query language (and elsewhere in Python expressions) like this:
```
  for x in data ./ {key == 'green'} .// { key == 'yellow' }
```

 - Try-except expressions. Python has try-except statement, but in many cases when working with dirty or semi-structured data, we need to be able to use an expression inside an iterator or the query. So we introduced a try-except expressions:
 
```
  [ try {int(x)} except {0} for x in values ]
```

 - Query expressions:
We have a syntax that looks similar to SQL, but is more flexible and of course most of the expressions in the query are
in pure Python. Here is an example PythonQL program:

```Python
# This example illustrates the try-catch business in PythonQL.
# Basically, some data might be dirty, but you still want to be able to write a simple query

from collections import namedtuple
ord = namedtuple('Order', ['cust_id','prod_id','price'])
cust = namedtuple('Cust', ['cust_id','cust_name'])

ords = [ ord(1,1,"16.54"),
         ord(1,2,"18.95"),
         ord(1,5,"8.96"),
         ord(2,1,"????"),
         ord(2,2,"20.00") ]

custs = [ cust(1,"John"), cust(2,"Dave"), cust(3,"Boris") ]

# Basic SQL query, but with some data cleaning
res = (select name, sum(price) as sum
        for o in ords
        let price = try{ float(o.price) } except {0}
        for c in custs
        where c.cust_id == o.cust_id
        group by c.cust_id as id, c.cust_name as name)

print (res)

# Funny query, lets count how many integers there are everywhere in the data
res = ( select x
        for x in ords .//
        let y = try { True if int(x) else False } except { False }
        where y
        )

print (len(res))
```

## Usage:

`python3 pythonql/RunPYQL.py <pythonql program>`
