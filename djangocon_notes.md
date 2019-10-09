# Building Effective Queries in Django

Query expressions - much more efficient

Use the Django ORM - a way to translate database relations into pythonic models

- Q and F objects 
- Database functions
  - Extract, Concat, Replace, ...
- Conditional expressions
  - When, Case, ...

##Q objects
ex: 
Q(birthdate__year__range=(1981, 1996))

or more complex (can be nested, and can be used logical operators like &, |):
Person.objects.filter(
    Q(
        Q(birthdate__year__range=(1981, 1996))
        | Q(birthdate__year__range(1946, 1964))

    )
    & Q(location__postal_code="CA")
)

because it is so extensible, the query can grow to be very complicated
- advisable to include comments

Can also filter dynamically using using if statements

```
user_filter = Q(is_active=True)
value = params["value"]

if "name" in params:
    user_filter.add(
        Q(first_name__icontains=value)
        | Q(last_name__icontains=value)
    ), 
    Q.OR,

```

if you want to see the actual query - you can use .query whcih will print the SQL generated.

##F objects

according to american red cross, one can donate blood every 56 days.



Person.objects.filter(list_of_Q_objects)
  - Q objects can be nexted
  - // comment just a bit because they can get very large and extensible
  - can use icontains
  - can print yor query with .query for exact sql query

F objects (or expressions)
- filter objects?
- no need to store the filter in memory
- filter against other fields

Database functions
- Count - count number of records
- properties cannot be used in queries
- Coalesce and NullIf
  - NullIf checks if the value is an empty string. 
- Conditionals in queries
  - Case, When expressions
  

Queryset - annotations can be added to querysets and managers

