# product_catalog_service
Microservice that manages product listings, categories, and inventory.

### Product Listing

This is the digital equivalent of a product on a shelf. It's the collection of all the information 
about a single item for sale. A good product listing typically includes:

    * Product Name: (e.g., "Men's Classic T-Shirt")
    * Description: Details about the product, its features, and benefits.
    * Price: How much it costs.
    * Images/Videos: High-quality visuals from different angles.
    * SKU (Stock Keeping Unit): A unique code to identify the product internally.
    * Specifications: Things like size, color, material, dimensions, etc.

### Product Categories

These are groups you use to organize your products, making it easier for customers to browse your store.
Think of them as the aisles in a supermarket. For example, you might have categories like:
    * Electronics
    * Smartphones
    * Laptops
    * Apparel
    * Shirts
    * Pants

Categories help customers find what they're looking for without having to sift through every single item you sell.

### Product Inventory:

This simply refers to the quantity of a specific product that you have available to sell.
The inventory management system's job is to: 
    * Track how many units of each product are in stock.
    * Decrease the count when a product is sold.
    * Alert you when stock is low so you can reorder.
    * Prevent customers from buying products that are out of stock.

In essence, your Product Catalog Service will be the backend system that manages all this data:
what the products are (listings), how they're organized (categories), and how many you have (inventory).


  Category Model

  This model is for organizing your products. It's designed to support nested categories (e.g., "Apparel" > "Shirts" > "T-Shirts").

    1 class Category(models.Model):
    2     # A field to store the category name, like "Electronics".
    3     # It must be unique within the database.
    4     name = models.CharField(max_length=255, unique=True)
    5 
    6     # A field for creating user-friendly URLs, like "electronics".
    7     # It's automatically generated from the name if left blank.
    8     slug = models.SlugField(max_length=255, unique=True, blank=True)
    9 
   10     # An optional field for a more detailed description of the category.
   11     description = models.TextField(blank=True)
   12 
   13     # This is the clever part for nesting. It links a category to another
   14     # category as its parent. For example, "Shirts" would have "Apparel"
   15     # as its parent. 'self' means the relationship is with the same model.
   16     parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='children')
   17 
   18     class Meta:
   19         # This just tells Django's admin interface to use "Categories"
   20         # as the plural name, which is grammatically correct.
   21         verbose_name_plural = 'Categories'
   22 
   23     def save(self, *args, **kwargs):
   24         # This custom method automatically creates the slug from the name
   25         # before the category is saved to the database.
   26         if not self.slug:
   27             self.slug = slugify(self.name)
   28         super().save(*args, **kwargs)
   29 
   30     def __str__(self):
   31         # This defines how a Category object is represented as a string,
   32         # which is useful in the admin panel. It will just show the name.
   33         return self.name

  ---

  Product Model

  This model represents an actual item that you sell in your store.

    1 class Product(models.Model):
    2     # The name of the product, e.g., "Men's Classic T-Shirt".
    3     name = models.CharField(max_length=255)
    4 
    5     # A unique, URL-friendly version of the name, like "mens-classic-t-shirt".
    6     slug = models.SlugField(max_length=255, unique=True, blank=True)
    7 
    8     # A field for the detailed product description.
    9     description = models.TextField(blank=True)
   10 
   11     # The price of the product. DecimalField is used to avoid floating-point
   12     # rounding errors with money.
   13     price = models.DecimalField(max_digits=10, decimal_places=2)
   14 
   15     # This links the product to a Category. It's a many-to-one relationship:
   16     # one category can have many products, but each product has only one category.
   17     category = models.ForeignKey(Category, on_delete=models.CASCADE, related_name='products')
   18 
   19     def save(self, *args, **kwargs):
   20         # Just like with Category, this auto-generates the slug from the name.
   21         if not self.slug:
   22             self.slug = slugify(self.name)
   23         super().save(*args, **kwargs)
   24 
   25     def __str__(self):
   26         # The string representation will be the product's name.
   27         return self.name

  ---

  Inventory Model

  This model is responsible for tracking the stock quantity for each product.

    1 class Inventory(models.Model):
    2     # This creates a strict one-to-one link with a Product.
    3     # Each product can have only one inventory entry, and each inventory
    4     # entry is tied to exactly one product.
    5     product = models.OneToOneField(Product, on_delete=models.CASCADE, related_name='inventory')
    6 
    7     # The number of items in stock. PositiveIntegerField ensures
    8     # you can't have a negative quantity.
    9     quantity = models.PositiveIntegerField(default=0)
   10 
   11     class Meta:
   12         # Corrects the plural form in the admin interface.
   13         verbose_name_plural = 'Inventories'
   14 
   15     def __str__(self):
   16         # Provides a helpful string representation, e.g., "Men's Classic T-Shirt - 100".
   17         return f'{self.product.name} - {self.quantity}'

