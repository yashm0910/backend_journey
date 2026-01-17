# **Django ORM Operations Cheat Sheet**

## **CRUD Operations (Most Common)**

### **1. Create Objects**
```python
# Method 1: Create and save
product = Product(name='Laptop', price=1000)
product.save()

# Method 2: Create in one step
product = Product.objects.create(name='Laptop', price=1000)

# Method 3: Bulk create (efficient for many objects)
Product.objects.bulk_create([
    Product(name='Phone', price=500),
    Product(name='Tablet', price=300)
])
```
*Creates new database records.*

### **2. Retrieve All Objects**
```python
all_products = Product.objects.all()
users = User.objects.all()
```
*Gets all records from a table.*

### **3. Get Single Object**
```python
# By primary key (pk)
product = Product.objects.get(pk=1)
product = Product.objects.get(id=1)

# By unique field
user = User.objects.get(email='john@example.com')

# Safe get (returns None if not found)
product = Product.objects.filter(id=999).first()

# Get or 404 (for views)
from django.shortcuts import get_object_or_404
product = get_object_or_404(Product, id=1)
```
*Fetches one record - raises error if none or multiple found.*

### **4. Update Objects**
```python
# Update single object
product = Product.objects.get(id=1)
product.price = 1200
product.save()  # Updates only

# Bulk update (more efficient)
Product.objects.filter(category='Electronics').update(discount=10)

# Update with F() expression
from django.db.models import F
Product.objects.filter(quantity__gt=0).update(
    price=F('price') * 1.1  # Increase price by 10%
)
```
*Modifies existing records.*

### **5. Delete Objects**
```python
# Delete single object
product = Product.objects.get(id=1)
product.delete()

# Delete multiple objects
Product.objects.filter(expired=True).delete()

# Delete all (careful!)
Product.objects.all().delete()
```
*Removes records from database.*

---

## **Filtering Operations**

### **6. Basic Filtering**
```python
# Exact match
expensive = Product.objects.filter(price=1000)

# Greater than
expensive = Product.objects.filter(price__gt=1000)

# Less than or equal
cheap = Product.objects.filter(price__lte=500)

# Range
medium = Product.objects.filter(price__range=(500, 1000))
```
*Filters records based on field conditions.*

### **7. String Operations**
```python
# Case-insensitive contains
coffee = Product.objects.filter(name__icontains='coffee')

# Starts with
starts = Product.objects.filter(name__istartswith='pro')

# Ends with
ends = Product.objects.filter(name__iendswith='er')

# Exact (case-sensitive)
exact = Product.objects.filter(name__exact='Coffee')

# In list
categories = Product.objects.filter(category__in=['Electronics', 'Books'])
```
*Filters text fields with various string operations.*

### **8. Null/Boolean Checks**
```python
# Is null
null_names = Product.objects.filter(name__isnull=True)

# Is not null
valid_names = Product.objects.filter(name__isnull=False)

# Boolean
active = Product.objects.filter(is_active=True)
inactive = Product.objects.filter(is_active=False)
```
*Checks for null values or boolean states.*

---

## **Q Objects (Complex Queries)**

### **9. OR Operations**
```python
from django.db.models import Q

# OR condition
products = Product.objects.filter(
    Q(price__lt=100) | Q(category='Clearance')
)

# AND with OR
products = Product.objects.filter(
    Q(category='Electronics') & (Q(price__lt=500) | Q(rating__gt=4))
)

# Negation (NOT)
products = Product.objects.filter(~Q(category='Books'))
```
*Combines multiple conditions with OR/AND/NOT logic.*

### **10. Complex Q Examples**
```python
# Search multiple fields
search = 'laptop'
results = Product.objects.filter(
    Q(name__icontains=search) | 
    Q(description__icontains=search) |
    Q(category__icontains=search)
)

# Multiple conditions
products = Product.objects.filter(
    Q(stock__gt=0) & 
    (Q(price__lt=100) | Q(on_sale=True))
)
```
*Advanced filtering with multiple conditions.*

---

## **F Objects (Field Comparisons)**

### **11. Compare Fields**
```python
from django.db.models import F

# Compare two fields
expensive = Product.objects.filter(price__gt=F('cost') * 2)

# Update based on another field
Product.objects.filter(stock__gt=0).update(
    value=F('price') * F('stock')
)

# Annotate with field operations
from django.db.models import Value
products = Product.objects.annotate(
    profit=F('price') - F('cost')
)
```
*References and compares fields within same record.*

---

## **Sorting & Limiting**

### **12. Ordering Results**
```python
# Ascending
products = Product.objects.all().order_by('price')

# Descending
products = Product.objects.all().order_by('-price')

# Multiple fields
products = Product.objects.all().order_by('category', '-price')

# Random order
import random
products = list(Product.objects.all())
random.shuffle(products)
```
*Sorts query results in specified order.*

### **13. Limiting Results**
```python
# First N records
first_5 = Product.objects.all()[:5]

# Skip first N
skip_5 = Product.objects.all()[5:]

# Slice range
items_5_to_10 = Product.objects.all()[5:10]

# Get first/last
first = Product.objects.first()
last = Product.objects.last()
```
*Limits number of returned records (SQL LIMIT).*

---

## **Aggregation & Annotation**

### **14. Aggregations**
```python
from django.db.models import Count, Sum, Avg, Max, Min

# Count
total = Product.objects.count()
count_by_cat = Product.objects.aggregate(
    total=Count('id'),
    avg_price=Avg('price'),
    max_price=Max('price')
)

# Group by aggregation
from django.db.models import Count
category_counts = Product.objects.values('category').annotate(
    count=Count('id'),
    avg_price=Avg('price')
)
```
*Performs calculations across multiple records.*

### **15. Annotations**
```python
from django.db.models import Count, F, Value
from django.db.models.functions import Concat

# Add calculated field
products = Product.objects.annotate(
    profit=F('price') - F('cost')
)

# Count related objects
categories = Category.objects.annotate(
    product_count=Count('products')
)

# Conditional annotation
from django.db.models import Case, When, IntegerField
products = Product.objects.annotate(
    price_category=Case(
        When(price__lt=100, then=Value('cheap')),
        When(price__range=(100, 500), then=Value('medium')),
        When(price__gt=500, then=Value('expensive')),
        output_field=CharField()
    )
)
```
*Adds calculated fields to each record in queryset.*

---

## **Related Objects & Joins**

### **16. Foreign Key Relationships**
```python
# Forward relationship (One-to-Many)
products = Product.objects.filter(category__name='Electronics')

# Reverse relationship
category = Category.objects.get(id=1)
category_products = category.product_set.all()  # All products in category

# Many-to-Many
users = User.objects.filter(groups__name='Admins')
```
*Queries across related tables.*

### **17. Select Related (JOIN optimization)**
```python
# For ForeignKey or OneToOne (uses SQL JOIN)
products = Product.objects.select_related('category').all()
# Access category without extra query: product.category.name

# For ManyToMany or reverse FK (uses separate query)
products = Product.objects.prefetch_related('tags').all()
# Access tags without extra query: product.tags.all()
```
*Optimizes database queries for related objects.*

---

## **Field Selection**

### **18. Values & Values List**
```python
# Returns dictionaries
names = Product.objects.values('id', 'name', 'price')

# Returns tuples
ids_names = Product.objects.values_list('id', 'name')

# Flat list (single field)
ids = Product.objects.values_list('id', flat=True)

# With related fields
products = Product.objects.values('id', 'name', 'category__name')
```
*Selects specific fields instead of whole objects.*

---

## **Date/Time Operations**

### **19. Date Filters**
```python
from datetime import datetime, timedelta
from django.utils import timezone

# Today's items
today = timezone.now().date()
today_orders = Order.objects.filter(created_at__date=today)

# Last 7 days
week_ago = timezone.now() - timedelta(days=7)
recent = Order.objects.filter(created_at__gte=week_ago)

# Month/year filters
january = Order.objects.filter(created_at__month=1)
year_2023 = Order.objects.filter(created_at__year=2023)

# Date ranges
start_date = datetime(2023, 1, 1)
end_date = datetime(2023, 12, 31)
year_data = Order.objects.filter(
    created_at__range=(start_date, end_date)
)
```
*Filters based on date/time fields.*

---

## **Advanced Operations**

### **20. Transactions**
```python
from django.db import transaction

# Using context manager
with transaction.atomic():
    order = Order.objects.create(total=100)
    OrderItem.objects.create(order=order, product=product, quantity=2)
    # If anything fails here, everything rolls back

# Using decorator
@transaction.atomic
def create_order(user, items):
    order = Order.objects.create(user=user)
    # Multiple operations here
```
*Ensures multiple operations succeed or fail together.*

### **21. Raw SQL**
```python
from django.db import connection

# Raw query returning model instances
products = Product.objects.raw('SELECT * FROM store_product WHERE price > 100')

# Direct SQL execution
with connection.cursor() as cursor:
    cursor.execute("UPDATE store_product SET discount = 10 WHERE price > 500")
    cursor.execute("SELECT COUNT(*) FROM store_product")
    row = cursor.fetchone()
```
*Executes raw SQL when ORM isn't sufficient.*

### **22. Distinct & Exclude**
```python
# Remove duplicates
categories = Product.objects.values_list('category', flat=True).distinct()

# Exclude certain records
available = Product.objects.exclude(stock=0)
non_electronics = Product.objects.exclude(category='Electronics')
```
*Removes duplicates or excludes specific records.*

---

## **Query Optimization**

### **23. Defer & Only**
```python
# Defer loading of large fields
products = Product.objects.defer('description', 'specifications')

# Load only specific fields
products = Product.objects.only('id', 'name', 'price')

# Chaining
products = Product.objects.only('id', 'name').select_related('category')
```
*Controls which fields are loaded from database.*

### **24. Exists & Count Optimization**
```python
# Check if any records exist (more efficient than count())
has_products = Product.objects.exists()

# Check if specific record exists
is_available = Product.objects.filter(id=1).exists()

# Count optimization
count = Product.objects.count()  # SELECT COUNT(*)
```
*Efficient existence and counting checks.*

---

## **Practical Examples for Views**

### **25. Common View Patterns**
```python
# List view with pagination
def product_list(request):
    from django.core.paginator import Paginator
    
    products = Product.objects.filter(is_active=True).order_by('-created_at')
    paginator = Paginator(products, 20)  # 20 per page
    page = request.GET.get('page', 1)
    page_obj = paginator.get_page(page)
    
    return render(request, 'products/list.html', {'products': page_obj})

# Search view
def search(request):
    query = request.GET.get('q', '')
    if query:
        results = Product.objects.filter(
            Q(name__icontains=query) |
            Q(description__icontains=query)
        )[:10]
    else:
        results = Product.objects.none()
    
    return render(request, 'search.html', {'results': results, 'query': query})

# Dashboard statistics
def dashboard(request):
    stats = {
        'total_products': Product.objects.count(),
        'total_value': Product.objects.aggregate(
            total=Sum(F('price') * F('stock'))
        )['total'],
        'top_categories': Category.objects.annotate(
            product_count=Count('products')
        ).order_by('-product_count')[:5]
    }
    return render(request, 'dashboard.html', stats)
```

---

## **Quick Reference Card**

### **Common Field Lookups:**
- `exact` / `iexact` - Exact match (case sensitive/insensitive)
- `contains` / `icontains` - Contains substring
- `in` - Value in list
- `gt` / `gte` - Greater than / greater than or equal
- `lt` / `lte` - Less than / less than or equal
- `startswith` / `istartswith` - Starts with
- `endswith` / `iendswith` - Ends with
- `range` - Between range
- `isnull` - Is NULL
- `regex` / `iregex` - Regex match

### **Common Methods:**
- `all()` - Get all
- `get()` - Get single
- `filter()` - Filter results
- `exclude()` - Exclude results
- `order_by()` - Sort results
- `values()` - Get specific fields as dicts
- `annotate()` - Add calculated fields
- `aggregate()` - Calculate across all records
- `select_related()` - Optimize FK queries
- `prefetch_related()` - Optimize M2M queries

### **Performance Tips:**
1. Use `exists()` instead of `count()` if just checking existence
2. Use `update()` for bulk updates instead of loop + `save()`
3. Use `select_related()` for ForeignKey, `prefetch_related()` for ManyToMany
4. Use `only()`/`defer()` to limit loaded fields
5. Use `iterator()` for large querysets
6. Use database indexes for frequently filtered fields

This cheat sheet covers 90% of what you'll need daily. Start with CRUD operations and basic filtering, then gradually incorporate more advanced features as needed!
