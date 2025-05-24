# Python Django Reference Card

## Table of Contents
1. [Project Setup](#project-setup)
2. [Models](#models)
3. [Views](#views)
4. [Templates](#templates)
5. [URLs](#urls)
6. [Forms](#forms)
7. [Authentication](#authentication)
8. [Admin Interface](#admin-interface)
9. [Middleware](#middleware)
10. [Signals](#signals)
11. [Testing](#testing)
12. [Deployment](#deployment)
13. [REST API with Django REST Framework](#rest-api)
14. [Security Best Practices](#security)
15. [Performance Optimization](#performance)

## Project Setup <a name="project-setup"></a>

### Installation
```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Django
pip install django

# Start project
django-admin startproject myproject
cd myproject

# Create app
python manage.py startapp myapp
```

### Project Structure
```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
└── myapp/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations/
    ├── models.py
    ├── tests.py
    └── views.py
```

### Settings Configuration
```python
# myproject/settings.py

# Add app to INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # Add your app here
]

# Database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydatabase',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

## Models <a name="models"></a>

### Basic Model
```python
# myapp/models.py
from django.db import models
from django.contrib.auth.models import User

class Category(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name_plural = "Categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name

class Product(models.Model):
    STATUS_CHOICES = (
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    )
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    category = models.ForeignKey(Category, on_delete=models.CASCADE, related_name='products')
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    is_featured = models.BooleanField(default=False)
    
    def __str__(self):
        return self.title
```

### Common Field Types
```python
# String fields
models.CharField(max_length=100)
models.TextField()
models.SlugField()
models.EmailField()
models.URLField()

# Numeric fields
models.IntegerField()
models.PositiveIntegerField()
models.FloatField()
models.DecimalField(max_digits=10, decimal_places=2)

# Boolean fields
models.BooleanField(default=False)
models.NullBooleanField()

# Date/time fields
models.DateField(auto_now_add=True)  # Set only when created
models.DateTimeField(auto_now=True)  # Updated every time

# File fields
models.FileField(upload_to='files/')
models.ImageField(upload_to='images/')

# Relationship fields
models.ForeignKey(OtherModel, on_delete=models.CASCADE, related_name='items')
models.OneToOneField(OtherModel, on_delete=models.CASCADE)
models.ManyToManyField(OtherModel, related_name='items')
```

### Migrations
```bash
# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Show migration status
python manage.py showmigrations

# Generate SQL for migration
python manage.py sqlmigrate myapp 0001
```

### QuerySets
```python
# Basic queries
Product.objects.all()
Product.objects.filter(price__lt=100)
Product.objects.exclude(status='archived')
Product.objects.get(id=1)

# Chaining
Product.objects.filter(category__name='Electronics').exclude(status='draft')

# Field lookups
Product.objects.filter(title__contains='phone')
Product.objects.filter(price__gt=50)
Product.objects.filter(created_at__year=2023)
Product.objects.filter(category__in=[1, 2, 3])

# Q objects for complex queries
from django.db.models import Q
Product.objects.filter(Q(price__lt=100) | Q(is_featured=True))

# Ordering
Product.objects.order_by('price')
Product.objects.order_by('-created_at')  # Descending

# Aggregation
from django.db.models import Avg, Count, Sum, Min, Max
Product.objects.aggregate(Avg('price'))
Category.objects.annotate(product_count=Count('products'))

# Limiting results
Product.objects.all()[:5]  # First 5
Product.objects.all()[5:10]  # 5 to 10

# Selecting specific fields
Product.objects.values('title', 'price')  # Returns dictionaries
Product.objects.values_list('title', 'price')  # Returns tuples
Product.objects.values_list('title', flat=True)  # Returns single values
```

## Views <a name="views"></a>

### Function-Based Views
```python
# myapp/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse
from .models import Product

def home(request):
    return HttpResponse("Hello, World!")

def product_list(request):
    products = Product.objects.filter(status='published')
    return render(request, 'myapp/product_list.html', {'products': products})

def product_detail(request, slug):
    product = get_object_or_404(Product, slug=slug, status='published')
    return render(request, 'myapp/product_detail.html', {'product': product})
```

### Class-Based Views
```python
# myapp/views.py
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Product
from .forms import ProductForm

class ProductListView(ListView):
    model = Product
    template_name = 'myapp/product_list.html'
    context_object_name = 'products'
    paginate_by = 10
    
    def get_queryset(self):
        return Product.objects.filter(status='published')

class ProductDetailView(DetailView):
    model = Product
    template_name = 'myapp/product_detail.html'
    context_object_name = 'product'
    slug_url_kwarg = 'slug'

class ProductCreateView(CreateView):
    model = Product
    form_class = ProductForm
    template_name = 'myapp/product_form.html'
    success_url = reverse_lazy('product_list')
    
    def form_valid(self, form):
        form.instance.created_by = self.request.user
        return super().form_valid(form)

class ProductUpdateView(UpdateView):
    model = Product
    form_class = ProductForm
    template_name = 'myapp/product_form.html'
    
    def get_success_url(self):
        return reverse_lazy('product_detail', kwargs={'slug': self.object.slug})

class ProductDeleteView(DeleteView):
    model = Product
    template_name = 'myapp/product_confirm_delete.html'
    success_url = reverse_lazy('product_list')
```

### Mixins
```python
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin

class StaffRequiredMixin(UserPassesTestMixin):
    def test_func(self):
        return self.request.user.is_staff

class ProductCreateView(LoginRequiredMixin, StaffRequiredMixin, CreateView):
    model = Product
    form_class = ProductForm
    login_url = '/login/'
    redirect_field_name = 'next'
```

## Templates <a name="templates"></a>

### Template Structure
```
myapp/
└── templates/
    └── myapp/
        ├── base.html
        ├── product_list.html
        └── product_detail.html
```

### Base Template
```html
<!-- myapp/templates/myapp/base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Django Site{% endblock %}</title>
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <header>
        <nav>
            <ul>
                <li><a href="{% url 'home' %}">Home</a></li>
                <li><a href="{% url 'product_list' %}">Products</a></li>
                {% if user.is_authenticated %}
                    <li><a href="{% url 'logout' %}">Logout</a></li>
                {% else %}
                    <li><a href="{% url 'login' %}">Login</a></li>
                {% endif %}
            </ul>
        </nav>
    </header>
    
    <main>
        {% if messages %}
            <div class="messages">
                {% for message in messages %}
                    <div class="message {{ message.tags }}">{{ message }}</div>
                {% endfor %}
            </div>
        {% endif %}
        
        {% block content %}{% endblock %}
    </main>
    
    <footer>
        <p>&copy; {% now "Y" %} My Django Site</p>
    </footer>
    
    <script src="{% static 'js/main.js' %}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>
```

### Child Template
```html
<!-- myapp/templates/myapp/product_list.html -->
{% extends "myapp/base.html" %}

{% block title %}Products - {{ block.super }}{% endblock %}

{% block content %}
    <h1>Products</h1>
    
    {% if products %}
        <div class="product-grid">
            {% for product in products %}
                <div class="product-card">
                    <h2>{{ product.title }}</h2>
                    <p>{{ product.price|floatformat:2 }}</p>
                    <a href="{% url 'product_detail' product.slug %}">View Details</a>
                </div>
            {% endfor %}
        </div>
        
        {% if is_paginated %}
            <div class="pagination">
                {% if page_obj.has_previous %}
                    <a href="?page={{ page_obj.previous_page_number }}">Previous</a>
                {% endif %}
                
                <span class="current-page">
                    Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}
                </span>
                
                {% if page_obj.has_next %}
                    <a href="?page={{ page_obj.next_page_number }}">Next</a>
                {% endif %}
            </div>
        {% endif %}
    {% else %}
        <p>No products available.</p>
    {% endif %}
{% endblock %}
```

### Template Tags and Filters
```html
<!-- Variable output -->
{{ variable }}
{{ variable|default:"Nothing" }}

<!-- Conditionals -->
{% if condition %}
    ...
{% elif other_condition %}
    ...
{% else %}
    ...
{% endif %}

<!-- Loops -->
{% for item in items %}
    {{ forloop.counter }}: {{ item }}
    {% empty %}
    No items found.
{% endfor %}

<!-- URL handling -->
{% url 'view_name' arg1 arg2 %}
{% url 'view_name' kwarg1=value1 %}

<!-- Template inheritance -->
{% extends "base.html" %}
{% block blockname %}...{% endblock %}

<!-- Including other templates -->
{% include "snippet.html" %}
{% include "snippet.html" with variable=value %}

<!-- Comments -->
{# This is a comment #}
{% comment %}
    Multi-line comment
{% endcomment %}

<!-- Common filters -->
{{ value|length }}
{{ text|truncatechars:100 }}
{{ list|join:", " }}
{{ datetime|date:"Y-m-d" }}
{{ text|linebreaks }}
{{ text|safe }}  <!-- Don't escape HTML -->
```

### Custom Template Tags
```python
# myapp/templatetags/custom_tags.py
from django import template
from django.utils.safestring import mark_safe

register = template.Library()

@register.simple_tag
def get_setting(name):
    from django.conf import settings
    return getattr(settings, name, "")

@register.filter(name='currency')
def currency(value):
    return f"${value:.2f}"
```

## URLs <a name="urls"></a>

### Project URLs
```python
# myproject/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### App URLs
```python
# myapp/urls.py
from django.urls import path
from . import views

app_name = 'myapp'  # Namespace for URL names

urlpatterns = [
    # Function-based views
    path('', views.home, name='home'),
    path('products/', views.product_list, name='product_list'),
    path('products/<slug:slug>/', views.product_detail, name='product_detail'),
    
    # Class-based views
    path('products/', views.ProductListView.as_view(), name='product_list'),
    path('products/<slug:slug>/', views.ProductDetailView.as_view(), name='product_detail'),
    path('products/new/', views.ProductCreateView.as_view(), name='product_create'),
    path('products/<slug:slug>/edit/', views.ProductUpdateView.as_view(), name='product_update'),
    path('products/<slug:slug>/delete/', views.ProductDeleteView.as_view(), name='product_delete'),
]
```

### URL Patterns
```python
# Path converters
path('articles/<int:year>/', views.year_archive)  # Matches digits only
path('articles/<str:title>/', views.article)  # Matches strings
path('articles/<slug:slug>/', views.article)  # Matches slugs
path('articles/<uuid:id>/', views.article)  # Matches UUIDs
path('articles/<path:file_path>/', views.article)  # Matches paths with slashes

# Regular expressions (with re_path)
from django.urls import re_path
re_path(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive)
```

## Forms <a name="forms"></a>

### ModelForm
```python
# myapp/forms.py
from django import forms
from .models import Product

class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = ['title', 'slug', 'description', 'price', 'category', 'status', 'is_featured']
        # or exclude = ['created_by', 'created_at', 'updated_at']
        widgets = {
            'description': forms.Textarea(attrs={'rows': 5}),
            'status': forms.Select(attrs={'class': 'form-control'}),
        }
        
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['title'].widget.attrs.update({'class': 'form-control'})
        self.fields['price'].widget.attrs.update({'min': '0.01', 'step': '0.01'})
        
    def clean_title(self):
        title = self.cleaned_data.get('title')
        if len(title) < 5:
            raise forms.ValidationError("Title must be at least 5 characters long.")
        return title
        
    def clean(self):
        cleaned_data = super().clean()
        price = cleaned_data.get('price')
        status = cleaned_data.get('status')
        
        if status == 'published' and (price is None or price <= 0):
            raise forms.ValidationError("Published products must have a price greater than zero.")
            
        return cleaned_data
```

### Form
```python
# myapp/forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    subject = forms.CharField(max_length=200)
    message = forms.CharField(widget=forms.Textarea)
    priority = forms.ChoiceField(choices=[
        ('low', 'Low'),
        ('medium', 'Medium'),
        ('high', 'High'),
    ])
    cc_myself = forms.BooleanField(required=False)
```

### Form in Template
```html
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    
    <!-- Display entire form -->
    {{ form.as_p }}
    <!-- or -->
    {{ form.as_table }}
    <!-- or -->
    {{ form.as_ul }}
    
    <!-- Display individual fields -->
    <div class="form-group">
        {{ form.name.errors }}
        <label for="{{ form.name.id_for_label }}">Name:</label>
        {{ form.name }}
    </div>
    
    <button type="submit">Submit</button>
</form>
```

### Handling Forms in Views
```python
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            # Process the form data
            name = form.cleaned_data['name']
            email = form.cleaned_data['email']
            # ... process the form data
            
            # Redirect after successful submission
            return redirect('success')
    else:
        form = ContactForm()
    
    return render(request, 'contact.html', {'form': form})
```

## Authentication <a name="authentication"></a>

### User Model
```python
from django.contrib.auth.models import User
from django.contrib.auth.models import AbstractUser

# Using built-in User model (recommended for most cases)
user = User.objects.create_user(
    username='johndoe',
    email='john@example.com',
    password='securepassword'
)

# Creating a custom user model (in models.py)
class CustomUser(AbstractUser):
    bio = models.TextField(blank=True)
    birth_date = models.DateField(null=True, blank=True)
```

### Authentication Views
```python
# myproject/urls.py
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(template_name='myapp/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(next_page='/'), name='logout'),
    path('password_change/', auth_views.PasswordChangeView.as_view(), name='password_change'),
    path('password_change/done/', auth_views.PasswordChangeDoneView.as_view(), name='password_change_done'),
    path('password_reset/', auth_views.PasswordResetView.as_view(), name='password_reset'),
    path('password_reset/done/', auth_views.PasswordResetDoneView.as_view(), name='password_reset_done'),
    path('reset/<uidb64>/<token>/', auth_views.PasswordResetConfirmView.as_view(), name='password_reset_confirm'),
    path('reset/done/', auth_views.PasswordResetCompleteView.as_view(), name='password_reset_complete'),
]
```

### Registration View
```python
# myapp/views.py
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth import login, authenticate

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password1')
            user = authenticate(username=username, password=password)
            login(request, user)
            return redirect('home')
    else:
        form = UserCreationForm()
    return render(request, 'registration/register.html', {'form': form})
```

### Login Required
```python
# Function-based views
from django.contrib.auth.decorators import login_required

@login_required(login_url='/login/')
def profile(request):
    return render(request, 'profile.html')

# Class-based views
from django.contrib.auth.mixins import LoginRequiredMixin

class ProfileView(LoginRequiredMixin, TemplateView):
    template_name = 'profile.html'
    login_url = '/login/'
    redirect_field_name = 'next'
```

### Permissions
```python
# Function-based views
from django.contrib.auth.decorators import permission_required

@permission_required('myapp.add_product', raise_exception=True)
def create_product(request):
    # View code here
    
# Class-based views
from django.contrib.auth.mixins import PermissionRequiredMixin

class ProductCreateView(PermissionRequiredMixin, CreateView):
    permission_required = 'myapp.add_product'
    template_name = 'product_form.html'
    model = Product
    form_class = ProductForm
```

## Admin Interface <a name="admin-interface"></a>

### Basic Admin Registration
```python
# myapp/admin.py
from django.contrib import admin
from .models import Category, Product

admin.site.register(Category)
admin.site.register(Product)
```

### Customized Admin
```python
# myapp/admin.py
from django.contrib import admin
from .models import Category, Product

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('name', 'description')
    search_fields = ('name',)

@admin.register(Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ('title', 'category', 'price', 'status', 'created_at')
    list_filter = ('status', 'category', 'is_featured')
    search_fields = ('title', 'description')
    prepopulated_fields = {'slug': ('title',)}
    list_editable = ('status', 'price')
    date_hierarchy = 'created_at'
    ordering = ('-created_at',)
    fieldsets = (
        (None, {
            'fields': ('title', 'slug', 'category', 'description')
        }),
        ('Pricing', {
            'fields': ('price',),
            'classes': ('collapse',)
        }),
        ('Status', {
            'fields': ('status', 'is_featured')
        }),
        ('Ownership', {
            'fields': ('created_by',)
        }),
    )
    
    def save_model(self, request, obj, form, change):
        if not change:  # Only set the creator for new objects
            obj.created_by = request.user
        super().save_model(request, obj, form, change)
```

### Inline Admin Models
```python
# myapp/admin.py
from django.contrib import admin
from .models import Category, Product

class ProductInline(admin.TabularInline):  # or admin.StackedInline
    model = Product
    extra = 1
    fields = ('title', 'price', 'status')

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    inlines = [ProductInline]
```

## Middleware <a name="middleware"></a>

### Custom Middleware
```python
# myapp/middleware.py
import time
from django.utils.deprecation import MiddlewareMixin

class RequestTimingMiddleware(MiddlewareMixin):
    def process_request(self, request):
        request.start_time = time.time()
        
    def process_response(self, request, response):
        if hasattr(request, 'start_time'):
            duration = time.time() - request.start_time
            response['X-Request-Duration'] = str(duration)
        return response
```

### Adding Middleware
```python
# myproject/settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'myapp.middleware.RequestTimingMiddleware',  # Custom middleware
]
```

## Signals <a name="signals"></a>

### Signal Handling
```python
# myapp/signals.py
from django.db.models.signals import post_save, pre_delete
from django.dispatch import receiver
from django.contrib.auth.models import User
from .models import Profile

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

### Connecting Signals
```python
# myapp/apps.py
from django.apps import AppConfig

class MyappConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'
    
    def ready(self):
        import myapp.signals  # Import signals when app is ready
```

## Testing <a name="testing"></a>

### Model Tests
```python
# myapp/tests.py
from django.test import TestCase
from django.contrib.auth.models import User
from .models import Category, Product

class ProductModelTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Set up non-modified objects used by all test methods
        test_user = User.objects.create_user(username='testuser', password='12345')
        test_category = Category.objects.create(name='Electronics')
        Product.objects.create(
            title='Test Product',
            slug='test-product',
            description='Test description',
            price=99.99,
            category=test_category,
            created_by=test_user
        )

    def test_title_max_length(self):
        product = Product.objects.get(id=1)
        max_length = product._meta.get_field('title').max_length
        self.assertEqual(max_length, 200)
    
    def test_object_name_is_title(self):
        product = Product.objects.get(id=1)
        expected_object_name = product.title
        self.assertEqual(str(product), expected_object_name)
```

### View Tests
```python
# myapp/tests.py
from django.test import TestCase
from django.urls import reverse
from django.contrib.auth.models import User
from .models import Category, Product

class ProductListViewTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        # Create 13 products for pagination tests
        test_user = User.objects.create_user(username='testuser', password='12345')
        test_category = Category.objects.create(name='Electronics')
        
        number_of_products = 13
        for product_num in range(number_of_products):
            Product.objects.create(
                title=f'Test Product {product_num}',
                slug=f'test-product-{product_num}',
                description='Test description',
                price=99.99,
                category=test_category,
                created_by=test_user,
                status='published'
            )
    
    def test_view_url_exists_at_desired_location(self):
        response = self.client.get('/products/')
        self.assertEqual(response.status_code, 200)
    
    def test_view_url_accessible_by_name(self):
        response = self.client.get(reverse('product_list'))
        self.assertEqual(response.status_code, 200)
    
    def test_view_uses_correct_template(self):
        response = self.client.get(reverse('product_list'))
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'myapp/product_list.html')
    
    def test_pagination_is_ten(self):
        response = self.client.get(reverse('product_list'))
        self.assertEqual(response.status_code, 200)
        self.assertTrue('is_paginated' in response.context)
        self.assertTrue(response.context['is_paginated'] == True)
        self.assertEqual(len(response.context['products']), 10)
```

### Form Tests
```python
# myapp/tests.py
from django.test import TestCase
from .forms import ProductForm

class ProductFormTest(TestCase):
    def test_product_form_title_field_label(self):
        form = ProductForm()
        self.assertTrue(form.fields['title'].label == 'Title' or form.fields['title'].label == None)
    
    def test_product_form_price_field_min_value(self):
        form = ProductForm(data={
            'title': 'Test Product',
            'slug': 'test-product',
            'description': 'Test description',
            'price': -1,
            'category': 1,
            'status': 'published'
        })
        self.assertFalse(form.is_valid())
```

### Client Tests
```python
# myapp/tests.py
from django.test import TestCase, Client
from django.urls import reverse
from django.contrib.auth.models import User

class LoginTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpassword'
        )