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
