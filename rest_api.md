# DJANGO-REST-FRAMEWORK


- API-application programming interface 
- just like buttons on remote,endpoints are buttons to control datat retrieval
- example: http://127.0.0.1:8000/store/products/1 
- here 'store' is main router to the desiredapp
- also , products is router to the desired view in store/views.py 
- client --> request using endpoints --> response generated 


 REST- Representation State Transfer
- rest defines bunch of rules ,client use these rules to interact with the web
- makes pis scalable ,reliable ,fast,ease of understaanding
- main rules that make an api restful are:

1.resources
2.resource representation
3.http methods 

### Resources
- objects what client wanna interact with
- example: http://127.0.0.1:8000/store/products/1 
- here ,store , products are nothing else but resources 
- more resources in the URL can just raise complexities and make urls messy

### Representations
- how the resources are returned
- html , xml ,json [main formats of response that clients understand]

### HTTP Methods
- each endpoint supports various kinds of opertions ,some allow reading,some allow writing
- http methods is how clients tell the server ,what they have to do with resource
-  GET -getting resource
- POST -creating resource
- PUT -updating the resource
- PATCH -updating a part of the resource
- DELETE -to delete a resource

# Serializers [24 mins-ch]
- used to converts model objects in form of dictionary
- created serializers.py ,made similar class to products model with objects only what to fetch in dictionary
- added :   REST_FRAMEWORK={
            'COERCE_DECIMAL_TO_STRING': False
            }
  so decimals are in form of decimal not string

- if handling an exception ,instead of using 'magic numers' like 'status=404' ,we should use constants for better readability
- URL patterns like "products/<int:id>" ,we can access products using their specific id

API MODELS != DATA MODELS -bcz API models are interface ,but data models is real implementation of the project ,So changing an API we need to study impact of the change before building respective versions of that API

- learned to apply methods in serialized objects using SerializerMethodField(method_name='method_name')
- PrimaryKeyRelatedField may be used to represent the target of the relationship using its primary key.

- we use 'class Meta:' as an inner class that holds metadata (configuration/settings) about the main class
- with meta ,the logics + model attributes arent messed up and handled seperately
- INDIDE META Class:
'''Python
class Meta:
    # 1. Which MODEL to use
    model = Product
    
    # 2. Which FIELDS to include from the MODEL
    fields = ['id', 'title', 'price', 'collection']
    # These MUST exist in the Product model
    
    # 3. Other CONFIGURATION (no logic!)
    depth = 1                     # How deep to follow relationships
    read_only_fields = ['id']    # Fields users can't modify
    exclude = ['password']       # Fields to exclude

- OUTSIDE META Class:
'''Python
class ProductSerializer(serializers.ModelSerializer):
    # 1. CUSTOM FIELDS (not in original model)
    price_with_tax = serializers.SerializerMethodField()
    discount_percentage = serializers.DecimalField(max_digits=5, decimal_places=2)
    
    # 2. METHODS for custom calculations
    def get_price_with_tax(self, obj):
        # Business logic here
        return obj.price * Decimal('1.1')
    
    # 3. VALIDATION methods
    def validate_price(self, value):
        if value < 0:
            raise serializers.ValidationError("Price can't be negative")
        return value
    
    # 4. CONFIGURATION (Meta class)
    class Meta:
        model = Product
        fields = ['id', 'title', 'price', 'price_with_tax', 'collection']
