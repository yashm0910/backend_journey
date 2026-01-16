# Django ORM Operations Cheat Sheet

## **F() Expressions**
**Purpose:** Perform database operations without pulling data into Python

```python
from django.db.models import F

# Increment a field
Product.objects.filter(id=1).update(price=F('price') + 10)

# Compare two fields in same row
Product.objects.filter(stock=F('sold'))

# Perform calculations in queries
Order.objects.annotate(
    total=F('quantity') * F('unit_price')
)

# Combined operations
Product.objects.update(
    price=F('price') * 1.1,  # Increase by 10%
    discount_price=F('price') * 0.9
)
```

## **Q() Objects**
**Purpose:** Complex queries with OR, AND, NOT operations

```python
from django.db.models import Q

# OR condition
User.objects.filter(Q(age__gt=18) | Q(is_staff=True))

# AND condition
User.objects.filter(Q(age__gt=18) & Q(city='London'))

# NOT condition
User.objects.filter(~Q(status='banned'))

# Complex combinations
Product.objects.filter(
    Q(price__lt=100) | Q(category='sale'),
    Q(stock__gt=0),
    ~Q(status='discontinued')
)

# Using in query
search = "django"
Book.objects.filter(
    Q(title__icontains=search) | 
    Q(author__name__icontains=search) |
    Q(description__icontains=search)
)
```

---

# **Django ORM Comprehensive Cheat Sheet**

## **1. BASIC CRUD OPERATIONS**

### **Create**
```python
# Single create
obj = Model.objects.create(field1=value1, field2=value2)

# Create multiple
Model.objects.bulk_create([
    Model(field1='a'),
    Model(field1='b')
])

# Create with save
obj = Model(field1=value1)
obj.save()
```

### **Read (Retrieve)**
```python
# Get single
obj = Model.objects.get(id=1)
obj = get_object_or_404(Model, id=1)  # Handles 404

# Get all
all_objs = Model.objects.all()

# Filter
objs = Model.objects.filter(field='value')
objs = Model.objects.exclude(field='value')  # Opposite of filter

# First/last
first = Model.objects.first()
last = Model.objects.last()
```

### **Update**
```python
# Update single
obj.field = 'new_value'
obj.save()

# Update multiple
Model.objects.filter(field='old').update(field='new')

# Update with F()
Model.objects.all().update(counter=F('counter') + 1)
```

### **Delete**
```python
# Delete single
obj.delete()

# Delete multiple
Model.objects.filter(field='value').delete()
```

---

## **2. QUERY FILTERS (LOOKUPS)**

### **Basic Lookups**
```python
# Exact match
.objects.filter(name__exact='John')
.objects.filter(name='John')  # Same as above

# Case-insensitive
.objects.filter(name__iexact='john')

# Contains
.objects.filter(name__contains='oh')
.objects.filter(name__icontains='oh')  # Case-insensitive

# Start/End with
.objects.filter(name__startswith='J')
.objects.filter(name__istartswith='j')
.objects.filter(name__endswith='n')
.objects.filter(name__iendswith='N')

# IN clause
.objects.filter(id__in=[1, 2, 3])

# Range
.objects.filter(date__range=('2023-01-01', '2023-12-31'))
```

### **Comparison Lookups**
```python
# Greater/Less than
.objects.filter(price__gt=100)  # >
.objects.filter(price__gte=100) # >=
.objects.filter(price__lt=100)  # <
.objects.filter(price__lte=100) # <=

# Null checks
.objects.filter(field__isnull=True)
.objects.filter(field__isnull=False)
```

### **Date/Time Lookups**
```python
import datetime
from django.utils import timezone

today = timezone.now().date()

.objects.filter(date__year=2023)
.objects.filter(date__month=1)
.objects.filter(date__day=15)
.objects.filter(date__week_day=1)  # Sunday=1, Monday=2...

# Relative dates
.objects.filter(date__gt=timezone.now())
.objects.filter(date__lt=timezone.now())

# Today
.objects.filter(date__date=today)
.objects.filter(date__date__gt=today)
```

---

## **3. FIELD LOOKUPS (RELATIONSHIPS)**

### **ForeignKey Lookups**
```python
# Forward (Model has ForeignKey)
Blog.objects.filter(entry__headline__contains='Lennon')

# Reverse (Model is ForeignKey target)
Entry.objects.filter(blog__name='Beatles Blog')

# Through relationships
Entry.objects.filter(blog__author__name='John')
```

### **ManyToMany Lookups**
```python
# Direct
Article.objects.filter(tags__name='django')

# Reverse
Tag.objects.filter(article__title__contains='ORM')
```

### **OneToOne Lookups**
```python
Place.objects.filter(restaurant__serves_pizza=True)
Restaurant.objects.filter(place__address__contains='Street')
```

---

## **4. QUERY METHODS & CHAINING**

```python
# Chaining filters
qs = Model.objects.all()
qs = qs.filter(field1='a').exclude(field2='b')

# Ordering
.objects.order_by('field')          # ASC
.objects.order_by('-field')         # DESC
.objects.order_by('field1', '-field2')

# Distinct
.objects.distinct()
.objects.values('field').distinct()

# Slicing/Limiting
.objects.all()[:5]        # First 5
.objects.all()[5:10]      # Items 5-9

# Count
count = Model.objects.count()
count = Model.objects.filter(field='value').count()

# Exists
if Model.objects.filter(field='value').exists():
    # do something

# Aggregate
from django.db.models import Avg, Max, Min, Sum, Count

Product.objects.aggregate(
    avg_price=Avg('price'),
    max_price=Max('price'),
    total_items=Count('id')
)
```

---

## **5. ANNOTATE & AGGREGATE**

### **Annotations (Add calculated fields)**
```python
from django.db.models import Count, Sum, Avg, F, Value
from django.db.models.functions import Concat, Lower

# Count related objects
Author.objects.annotate(book_count=Count('books'))

# Sum/Average
Order.objects.annotate(
    total=Sum(F('quantity') * F('price'))
)

# Complex annotations
Product.objects.annotate(
    discounted_price=F('price') * 0.9,
    price_category=Case(
        When(price__lt=100, then=Value('cheap')),
        When(price__lt=500, then=Value('medium')),
        default=Value('expensive'),
        output_field=CharField()
    )
)
```

### **Aggregations**
```python
from django.db.models import Avg, Max, Min, Sum, Count, StdDev, Variance

# Overall aggregates
Product.objects.aggregate(
    total_value=Sum(F('stock') * F('price')),
    avg_price=Avg('price'),
    max_price=Max('price')
)

# Group by aggregates
from django.db.models import Count
Author.objects.values('country').annotate(
    author_count=Count('id')
).order_by('-author_count')
```

---

## **6. ADVANCED QUERIES**

### **Select Related (JOIN for ForeignKey)**
```python
# Reduces queries for ForeignKey (1 query instead of N+1)
entries = Entry.objects.select_related('blog').all()
# Use for ForeignKey and OneToOne
```

### **Prefetch Related (for ManyToMany)**
```python
# Reduces queries for ManyToMany and reverse ForeignKey
articles = Article.objects.prefetch_related('tags', 'authors').all()

# With custom querysets
articles = Article.objects.prefetch_related(
    Prefetch('comments', queryset=Comment.objects.filter(is_approved=True))
)
```

### **Raw SQL**
```python
# When ORM is not enough
Person.objects.raw('SELECT * FROM myapp_person WHERE age > %s', [18])

# Mapping to model
query = '''
    SELECT id, first_name, last_name, birth_date 
    FROM some_other_table
'''
for person in Person.objects.raw(query):
    print(person.first_name)
```

### **Subqueries**
```python
from django.db.models import Subquery, OuterRef

# Subquery in filter
newest = Comment.objects.filter(post=OuterRef('pk')).order_by('-created_at')
Post.objects.annotate(newest_commenter_email=Subquery(newest.values('email')[:1]))
```

---

## **7. TRANSACTIONS**

```python
from django.db import transaction

# Decorator style
@transaction.atomic
def viewfunc(request):
    # This code runs inside a transaction
    do_something()

# Context manager style
def viewfunc(request):
    with transaction.atomic():
        # This code runs inside a transaction
        do_something()
        
    # Outside transaction
    do_something_else()
```

---

## **8. PERFORMANCE OPTIMIZATION**

```python
# 1. Use select_related for ForeignKey
.objects.select_related('author', 'category')

# 2. Use prefetch_related for ManyToMany
.objects.prefetch_related('tags', 'comments')

# 3. Use only() to select specific fields
.objects.only('id', 'name', 'email')

# 4. Use defer() to exclude heavy fields
.objects.defer('large_text_field', 'binary_data')

# 5. Use values()/values_list() for dicts/tuples
.objects.values('id', 'name')  # Returns dict
.objects.values_list('id', 'name')  # Returns tuple
.objects.values_list('id', flat=True)  # Flat list

# 6. Use iterator() for large querysets
for obj in Model.objects.all().iterator(chunk_size=2000):
    process(obj)
```

---

## **9. COMMON PATTERNS**

```python
# Get or create
obj, created = Model.objects.get_or_create(
    field1='value1',
    defaults={'field2': 'default_value'}
)

# Update or create
obj, created = Model.objects.update_or_create(
    field1='value1',
    defaults={'field2': 'new_value'}
)

# Bulk operations
Model.objects.bulk_create(objects_list)
Model.objects.bulk_update(objects_list, ['field1', 'field2'])

# Conditional update
from django.db.models import When, Case, Value

Model.objects.update(
    status=Case(
        When(score__gte=80, then=Value('A')),
        When(score__gte=60, then=Value('B')),
        default=Value('C')
    )
)
```

---

## **10. USEFUL METHODS & PROPERTIES**

```python
# Get URL for model instance
obj.get_absolute_url()

# Get display value for choice field
obj.get_status_display()

# Refresh from database
obj.refresh_from_db()

# Get field value
Model.objects.values_list('field', flat=True)

# Check if queryset contains object
if some_obj in my_queryset:
    # do something

# Latest/Oldest by date
Model.objects.latest('created_at')
Model.objects.earliest('created_at')

# Random entry
Model.objects.order_by('?').first()
```

---

## **QUICK REFERENCE CARD**

### **Most Used (80% of cases):**
1. `filter()`, `exclude()`, `get()`
2. `create()`, `save()`, `delete()`
3. `all()`, `count()`, `exists()`
4. `order_by()`, `values()`, `values_list()`
5. `select_related()`, `prefetch_related()`

### **Common (15% of cases):**
1. `annotate()`, `aggregate()`
2. `Q()` objects, `F()` expressions
3. `get_or_create()`, `update_or_create()`
4. `bulk_create()`, `bulk_update()`
5. `Case/When` expressions

### **Advanced (5% of cases):**
1. Raw SQL queries
2. Complex subqueries
3. Database functions
4. Custom managers/querysets
5. Multi-database operations

---

**Remember:** Always check Django QuerySet documentation for your version, as some features may vary. Use `print(queryset.query)` to see the generated SQL!
