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

- Used to reference fields
- Avoids loading database objects into memory
- Allows you to do basic arithmatic operations on query expressions
- Can be used with annotations (annotate())

Ex:
```
BloodBank.objecs.annotate(
    existent_amount=Count("bloodbag_set"),
    remaining=F("goal") - F("existent_amount"),
).values_list("goal", "existent_amount", "remaining")
```

What about expressions with different field types?
Ex:
```
Event.objects.annotate(
    ends_on=F("starts_at") + F("duration")
)
```
^^ if `starts_at` is a timefield and `duration` is a datetimefield, it will throw a `FieldError`

Solution is wrap in ExpressionWrapper

```
Event.objects.annotate(
    ends_on=ExpressionWrapper(
        F("starts_at") + F("duration"),
        output_field=DateTimeField()
    )
)
```

can mix F and Q objects together


##Database functions
Ued with annotations are very performant

What if we want to query a model's property? 

```
Person.objects.annotate(
    full_name=Concat("first_name", Value(" "), "last_name")
).filter(full_name__icontains="john").values_list("full_name")
```

Can improve performance further by adding queryset to models:

```
class PersonQuerySet(mdoels.QuerySet):
    def annotate_full_name(self):
        return self.annotate(
            full_name=Concat("first_name", Value(" "), "last_name")
        )

class Person(models.Model):
    objects=PersonQuerySet.as_manager()

# now you can do

Person.objects.annotate_full_name().values_list("full_name")
```

### Coalesce
- database method that returns the first non-null value across multiple fields
```
Person.objects.annotate(
    goes_by=Coalesce("nickname", "first_name")
)
```

## Conditional Expressions

Case, When

django-debug-toolbar

QuerySet.explain()

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

