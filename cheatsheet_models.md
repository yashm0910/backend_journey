# **Django Models & Relationships Cheat Sheet**

## **Core Model Structure**

### **Basic Model Definition**
```python
from django.db import models
from django.contrib.auth.models import User

class Product(models.Model):
    # Primary key (auto-added: id = models.AutoField(primary_key=True))
    name = models.CharField(max_length=200)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    def __str__(self):
        return self.name
    
    class Meta:
        ordering = ['-created_at']
        verbose_name_plural = "Products"
```

---

## **Field Types (Data Types)**

### **String Fields**
```python
# Fixed length
short_code = models.CharField(max_length=10)  # VARCHAR(10)

# Variable length text
description = models.TextField()  # TEXT/LONGTEXT

# Email with validation
email = models.EmailField(max_length=255)  # VARCHAR + email validation

# URL with validation
website = models.URLField(max_length=200)  # VARCHAR + URL validation

# File paths
file_path = models.FilePathField(path='/files/')

# UUID (good for API IDs)
import uuid
uid = models.UUIDField(default=uuid.uuid4, editable=False)
```
*Use CharField for short text, TextField for long content.*

### **Numeric Fields**
```python
# Integers
quantity = models.IntegerField()  # INTEGER
small_num = models.SmallIntegerField()  # SMALLINT
positive = models.PositiveIntegerField()  # Only positive
auto_id = models.AutoField(primary_key=True)  # Auto-increment

# Decimal (for money)
price = models.DecimalField(
    max_digits=10,     # Total digits (including decimals)
    decimal_places=2   # Digits after decimal
)  # DECIMAL(10, 2)

# Floating point
rating = models.FloatField()  # FLOAT

# Binary
data = models.BinaryField()  # BLOB
```
*Use DecimalField for money/critical calculations, FloatField for approximations.*

### **Date/Time Fields**
```python
# Date only
birth_date = models.DateField()  # DATE

# Date & time
created_at = models.DateTimeField()  # DATETIME

# Auto timestamps (MUST HAVE THESE!)
created = models.DateTimeField(auto_now_add=True)    # Set on creation
updated = models.DateTimeField(auto_now=True)        # Update on save

# Time only
start_time = models.TimeField()  # TIME

# Duration
duration = models.DurationField()  # INTERVAL
```
*Always include `auto_now_add` and `auto_now` for tracking timestamps.*

### **Boolean & Choice Fields**
```python
# Simple boolean
is_active = models.BooleanField(default=True)  # BOOLEAN

# Nullable boolean
is_verified = models.BooleanField(null=True)  # NULLABLE BOOLEAN

# Choices (enumerated values)
STATUS_CHOICES = [
    ('D', 'Draft'),
    ('P', 'Published'),
    ('A', 'Archived'),
]

status = models.CharField(
    max_length=2,
    choices=STATUS_CHOICES,
    default='D'
)  # Creates dropdown in admin

# Better for Python 3.11+
from django.db.models import TextChoices

class Status(models.TextChoices):
    DRAFT = 'D', 'Draft'
    PUBLISHED = 'P', 'Published'
    ARCHIVED = 'A', 'Archived'

status = models.CharField(
    max_length=2,
    choices=Status.choices,
    default=Status.DRAFT
)
```
*Use choices for fixed option sets, BooleanField for true/false.*

### **File & Image Fields**
```python
# File upload
document = models.FileField(upload_to='documents/')  # File path in MEDIA_ROOT

# Image upload (requires Pillow: pip install Pillow)
photo = models.ImageField(
    upload_to='photos/%Y/%m/%d/',  # Organized by date
    height_field='height',         # Auto-populate
    width_field='width',           # Auto-populate
    max_length=255
)

# File size validation
from django.core.validators import FileExtensionValidator
file = models.FileField(
    upload_to='uploads/',
    validators=[FileExtensionValidator(['pdf', 'docx'])]
)
```
*Remember: Files are stored in MEDIA_ROOT + upload_to path.*

### **Special Fields**
```python
# JSON data
metadata = models.JSONField(default=dict)  # PostgreSQL only

# Slug (URL-friendly)
slug = models.SlugField(max_length=255, unique=True)  # URL-safe string

# IP Address
ip_address = models.GenericIPAddressField()  # IPv4/IPv6

# Array (PostgreSQL only)
tags = models.ArrayField(
    models.CharField(max_length=50),
    size=10,
    default=list
)
```

---

## **Relationships (CRITICAL SECTION)**

### **1. ForeignKey (One-to-Many / Many-to-One)**
```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,      # Delete books if author deleted
        related_name='books',          # author.books.all()
        related_query_name='book'      # Author.objects.filter(book__title=...)
    )
    
    # For optional relationship
    publisher = models.ForeignKey(
        'Publisher',                   # Can use string if defined later
        on_delete=models.SET_NULL,     # Set to NULL if publisher deleted
        null=True,                     # Allows NULL
        blank=True                     # Allows blank in forms
    )
```
*Each Book has one Author, each Author can have many Books.*

### **2. OneToOneField (One-to-One)**
```python
class UserProfile(models.Model):
    user = models.OneToOneField(
        User,                          # Links to Django's User model
        on_delete=models.CASCADE,      # Delete profile if user deleted
        primary_key=True               # Profile ID = User ID
    )
    bio = models.TextField()
    phone = models.CharField(max_length=20)

# Access: user.userprofile.bio
# Or better: user.profile.bio (if using related_name='profile')
```
*Extends another model (like User-Profile). Each has exactly one.*

### **3. ManyToManyField (Many-to-Many)**
```python
class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)

class Article(models.Model):
    title = models.CharField(max_length=200)
    tags = models.ManyToManyField(
        Tag,
        related_name='articles',       # tag.articles.all()
        blank=True                     # Article can have no tags
    )
    
    # With through model (extra data on relationship)
    authors = models.ManyToManyField(
        'Author',
        through='ArticleAuthor',       # Custom join table
        related_name='authored_articles'
    )

class ArticleAuthor(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    is_primary = models.BooleanField(default=False)  # Extra data!
    assigned_date = models.DateField(auto_now_add=True)
```
*Each Article can have many Tags, each Tag can be on many Articles.*

---

## **on_delete Options (MUST CHOOSE CORRECTLY)**

```python
# 1. CASCADE (Most Common)
author = models.ForeignKey(Author, on_delete=models.CASCADE)
# If Author deleted, delete all their Books

# 2. PROTECT (Safe)
category = models.ForeignKey(Category, on_delete=models.PROTECT)
# Prevent Category deletion if any Products reference it

# 3. SET_NULL (Optional relationship)
publisher = models.ForeignKey(
    Publisher, 
    on_delete=models.SET_NULL,
    null=True,      # REQUIRED for SET_NULL
    blank=True
)
# Set publisher to NULL if Publisher deleted

# 4. SET_DEFAULT
status = models.ForeignKey(
    Status,
    on_delete=models.SET_DEFAULT,
    default=1       # Must specify default value
)
# Set to default Status if referenced one deleted

# 5. SET() - Custom
def get_orphan_user():
    return User.objects.get_or_create(username='orphan')[0]

owner = models.ForeignKey(
    User,
    on_delete=models.SET(get_orphan_user)
)
# Call function to get replacement value

# 6. DO_NOTHING (Dangerous!)
legacy_ref = models.ForeignKey(
    LegacyModel,
    on_delete=models.DO_NOTHING
)
# Database constraint must handle it
```
*Rule: Use CASCADE for compositions, SET_NULL for optional, PROTECT to prevent deletion.*

---

## **Model Meta Options**

```python
class Product(models.Model):
    # Fields here...
    
    class Meta:
        # Database table name
        db_table = 'store_products'
        
        # Default ordering
        ordering = ['-created_at', 'name']  # Most recent first
        
        # Unique together (composite unique constraint)
        unique_together = [
            ('category', 'sku'),  # SKU must be unique per category
        ]
        
        # Indexes (for performance)
        indexes = [
            models.Index(fields=['name']),
            models.Index(fields=['price', 'category']),  # Composite
            models.Index(fields=['-created_at'], name='recent_idx'),
        ]
        
        # Constraints (database-level)
        constraints = [
            models.CheckConstraint(
                check=models.Q(price__gte=0),
                name='price_non_negative'
            ),
            models.UniqueConstraint(
                fields=['email', 'is_active'],
                condition=models.Q(is_active=True),
                name='unique_active_email'
            )
        ]
        
        # Admin display
        verbose_name = "Product"              # Singular
        verbose_name_plural = "Products"      # Plural
        
        # Default permissions
        permissions = [
            ("can_publish", "Can publish products"),
            ("can_archive", "Can archive products"),
        ]
```

---

## **Model Methods & Properties**

```python
class Order(models.Model):
    subtotal = models.DecimalField(max_digits=10, decimal_places=2)
    tax_rate = models.DecimalField(max_digits=5, decimal_places=2)
    discount = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
    # 1. String representation (MUST HAVE)
    def __str__(self):
        return f"Order #{self.id} - ${self.total}"
    
    # 2. Property (calculated field, not in DB)
    @property
    def total(self):
        tax = self.subtotal * (self.tax_rate / 100)
        return self.subtotal + tax - self.discount
    
    # 3. Model method
    def apply_discount(self, percentage):
        """Apply percentage discount to order"""
        self.discount = self.subtotal * (percentage / 100)
        self.save()
    
    # 4. Absolute URL (for admin/view links)
    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('order_detail', args=[str(self.id)])
    
    # 5. Save override (careful!)
    def save(self, *args, **kwargs):
        # Pre-save logic
        if not self.slug:
            self.slug = slugify(self.title)
        
        # Call parent save
        super().save(*args, **kwargs)
        
        # Post-save logic
        if self.status == 'completed':
            self.send_notification()
    
    # 6. Manager method (class-level)
    @classmethod
    def get_recent_orders(cls, days=7):
        from django.utils import timezone
        cutoff = timezone.now() - timezone.timedelta(days=days)
        return cls.objects.filter(created_at__gte=cutoff)
    
    # 7. Validation
    def clean(self):
        from django.core.exceptions import ValidationError
        if self.discount > self.subtotal:
            raise ValidationError("Discount cannot exceed subtotal")
```

---

## **Custom Model Managers**

```python
# Custom manager for active objects
class ActiveManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)
    
    def by_category(self, category):
        return self.filter(category=category)

class Product(models.Model):
    # Default manager
    objects = models.Manager()  # Always keep default
    
    # Custom manager
    active = ActiveManager()    # Product.active.all()
    
    # Fields...
    is_active = models.BooleanField(default=True)
```

---

## **Common Patterns & Examples**

### **1. Base Model with Timestamps**
```python
class TimeStampedModel(models.Model):
    """Abstract base model with created/updated timestamps"""
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True  # Won't create database table

class Product(TimeStampedModel):
    # Inherits created_at, updated_at
    name = models.CharField(max_length=200)
```

### **2. User-Created Models**
```python
class UserCreatedModel(models.Model):
    """Abstract model for user-created content"""
    created_by = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='%(class)s_created'
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_by = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        related_name='%(class)s_updated'
    )
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True
```

### **3. Soft Delete Pattern**
```python
class SoftDeleteModel(models.Model):
    """Instead of real delete, mark as deleted"""
    is_deleted = models.BooleanField(default=False)
    deleted_at = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        abstract = True
    
    def delete(self, *args, **kwargs):
        """Override delete to soft delete"""
        self.is_deleted = True
        self.deleted_at = timezone.now()
        self.save()
    
    def hard_delete(self, *args, **kwargs):
        """Real delete"""
        super().delete(*args, **kwargs)
    
    # Custom manager to exclude deleted
    class QuerySet(models.QuerySet):
        def active(self):
            return self.filter(is_deleted=False)
        
        def deleted(self):
            return self.filter(is_deleted=True)
    
    objects = QuerySet.as_manager()
```

### **4. Category/Subcategory Pattern**
```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    parent = models.ForeignKey(
        'self',                     # Self-referential
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children'
    )
    slug = models.SlugField(unique=True)
    
    class Meta:
        verbose_name_plural = "Categories"
    
    def __str__(self):
        return self.name
    
    @property
    def is_root(self):
        return self.parent is None
    
    def get_ancestors(self):
        """Get all parent categories"""
        ancestors = []
        current = self
        while current.parent:
            ancestors.append(current.parent)
            current = current.parent
        return ancestors
    
    def get_descendants(self):
        """Get all child categories (recursive)"""
        descendants = []
        for child in self.children.all():
            descendants.append(child)
            descendants.extend(child.get_descendants())
        return descendants
```

### **5. Tagging System**
```python
class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)
    
    def __str__(self):
        return self.name

class TaggableModel(models.Model):
    tags = models.ManyToManyField(Tag, blank=True)
    
    class Meta:
        abstract = True
    
    def add_tags(self, tag_names):
        """Add multiple tags by name"""
        for name in tag_names:
            tag, _ = Tag.objects.get_or_create(
                name=name.strip(),
                defaults={'slug': slugify(name)}
            )
            self.tags.add(tag)
```

---

## **Migrations Checklist**

```bash
# After changing models:
python manage.py makemigrations      # Create migration
python manage.py migrate             # Apply migration

# Common migration commands:
python manage.py makemigrations app_name
python manage.py migrate app_name
python manage.py showmigrations
python manage.py sqlmigrate app_name 0001
```

### **Migration Operations:**
```python
# In migrations/000X_*.py
operations = [
    migrations.AddField(...),
    migrations.RemoveField(...),
    migrations.AlterField(...),
    migrations.RenameField(...),
    migrations.RenameModel(...),
    migrations.CreateModel(...),
    migrations.DeleteModel(...),
]
```

---

## **Golden Rules for Django Models**

1. **Always include**:
   - `__str__()` method for admin readability
   - `created_at` and `updated_at` fields
   - Proper `on_delete` for ForeignKeys

2. **Relationship rules**:
   - Use `ForeignKey` for "belongs to" relationships
   - Use `OneToOneField` for extending other models
   - Use `ManyToManyField` when both sides can have many
   - Always set `related_name` for reverse access

3. **Field options to remember**:
   - `null=True` (database can store NULL)
   - `blank=True` (forms can be empty)
   - `default=value` (default when creating)
   - `unique=True` (enforce uniqueness)
   - `db_index=True` (adds database index)

4. **Performance tips**:
   - Add `db_index=True` to frequently filtered fields
   - Use `select_related()` and `prefetch_related()` for related data
   - Consider `denormalization` for frequently accessed calculated values

5. **Naming conventions**:
   - Model names: `PascalCase` (Product, OrderItem)
   - Field names: `snake_case` (created_at, unit_price)
   - Related names: `plural` (products, order_items)

---

## **Complete Example: E-commerce Models**

```python
from django.db import models
from django.contrib.auth.models import User
from django.utils.text import slugify

class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='subcategories'
    )
    is_active = models.BooleanField(default=True)
    
    class Meta:
        verbose_name_plural = "Categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)

class Product(models.Model):
    category = models.ForeignKey(
        Category,
        on_delete=models.PROTECT,  # Don't delete if category has products
        related_name='products'
    )
    sku = models.CharField(max_length=50, unique=True)
    name = models.CharField(max_length=200)
    slug = models.SlugField(max_length=255, unique=True)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    cost = models.DecimalField(max_digits=10, decimal_places=2, null=True)
    stock = models.PositiveIntegerField(default=0)
    is_available = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['sku']),
            models.Index(fields=['name']),
            models.Index(fields=['price']),
        ]
    
    def __str__(self):
        return f"{self.name} (SKU: {self.sku})"
    
    @property
    def profit_margin(self):
        if self.cost:
            return ((self.price - self.cost) / self.cost) * 100
        return 0
    
    def reduce_stock(self, quantity):
        """Reduce stock, prevent negative"""
        if self.stock >= quantity:
            self.stock -= quantity
            self.save()
            return True
        return False

class ProductImage(models.Model):
    product = models.ForeignKey(
        Product,
        on_delete=models.CASCADE,
        related_name='images'
    )
    image = models.ImageField(upload_to='products/%Y/%m/%d/')
    alt_text = models.CharField(max_length=200, blank=True)
    is_primary = models.BooleanField(default=False)
    order = models.PositiveIntegerField(default=0)
    
    class Meta:
        ordering = ['order', 'id']
        unique_together = ['product', 'is_primary']  # Only one primary per product

class Review(models.Model):
    RATING_CHOICES = [
        (1, '★☆☆☆☆'),
        (2, '★★☆☆☆'),
        (3, '★★★☆☆'),
        (4, '★★★★☆'),
        (5, '★★★★★'),
    ]
    
    product = models.ForeignKey(
        Product,
        on_delete=models.CASCADE,
        related_name='reviews'
    )
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    rating = models.PositiveSmallIntegerField(choices=RATING_CHOICES)
    comment = models.TextField()
    is_approved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ['product', 'user']  # One review per user per product
        ordering = ['-created_at']
```

This comprehensive cheatsheet covers 95% of Django model scenarios. Start with the basic field types and relationships, then incorporate Meta options and custom methods as needed!
