<h1 dir='rtl'>فصل ۱۴: جستجو</h1>

<div dir='rtl'>

جستجو یکی از ویژگی‌های اساسی اکثر وبسایت‌ها و هر چیزی مربوط به تجارت الکنرونیک(e-commerce) است؛ مانند کتابفروشی ما. 
در این فصل نحوه پیاده‌سازی جسنجوی ساده را با فرم و فیلتر هارا یاد می‌گیریم.
سپس آن را با بهبود می‌بخشیم و نگاه عمیق‌تری به آپشن‌های سرچ در جنگو می‌اندازیم.
درحال حاضر فقط سه کتاب در دیتابیس خود داریم اما این کد به تعداد کتاب های دلخواهمان مقیاس‌پذیر است.

</div>

<div dir='rtl'>

جستجو شامل دو بخش است:  
- یک فرم که کوئری جستجو کاربر را با خود ارسال می‌کند. 
- سپس یک صفحه که نتایج جستجو را بر اساس کوئری ارائه می‌کند. 

</div>

<div dir='rtl'>

تعیین نوع مناسب فیلتر جایی است که جستجو, سخت و جالب می‌شود.
اما ابتدا باید فرم و صفحه‌ی نتایج جستجو را ایجاد کنیم.

</div>

<div dir='rtl'>

در این بخش می‌توان با یکی از دوتا شروع کرد که ما اول بخش فیلتر و سپس فرم را آماده می‌کنیم.

</div>

<h1 dir='rtl'>صفحه‌ی نمایش نتایج</h1>

<div dir='rtl'>

با صفحه‌ی نتایج شروع می‌کنیم. مانند سایر صفحات جنگو باید view, URL و template اختصاصی ایجاد کنیم.
ترتیب پیاده‌سازی اهمیت چندانی ندارد ولی ما به همان ترتیب پیش می‌رویم.

</div>

<div dir='rtl'>

داخل `books/urls.py`, مسیر `search/` را با نام `search_results` اضافه کنید که از view با نام `SearchResultsListView` استفاده می‌کند. 

</div>

<br>

**Code**

```python
# books/urls.py
from django.urls import path
from .views import BookListView, BookDetailView, SearchResultsListView # new

urlpatterns = [
    path('', BookListView.as_view(), name='book_list'),
    path('<uuid:pk>', BookDetailView.as_view(), name='book_detail'),
    path('search/', SearchResultsListView.as_view(),
        name='search_results'), # new
]
```

<br>

<div dir='rtl'>

view بعدی `SearchResultsListView` است که در حال حاضر لیستی از تمام کتاب‌های موجود است.
این گزینه اصلی برای استفاده از `ListView` است.
template آن `search_results.html` نامیده می‌شود و در پوشه‌ی `templates/books/` قرار دارد.
تنها کد جدید, مربوط به `SearchResultsListView` است همانطور که قبلا هم `ListView` و مدل `Book` را در بالای فایل ایمپورت کردیم.

</div>

<br>

**Code**

```python
# books/views.py
...
class SearchResultsListView(ListView): # new
    model = Book
    context_object_name = 'book_list'
    template_name = 'books/search_results.html'
```

<br>

<div dir='rtl'>

در آخر template با نام `search_results.html` را ایجاد می‌کنیم.

</div>

<br>

**Command Line**

```
$ touch templates/books/search_results.html
```

<br>

<div dir='rtl'>

حال تمام کتاب‌های موجود, بر اساس عنوان ,نویسنده و قیمت لیست می‌شوند.

</div>

<br>

**Command Line**

```
<!-- templates/books/search_results.html -->
{% extends '_base.html' %}

{% block title %}Search{% endblock title %}

{% block content %}
  <h1>Search Results</h1>
  {% for book in book_list %}
    <div>
      <h3><a href="{{ book.get_absolute_url }}">{{ book.title }}</a></h3>
      <p>Author: {{ book.author }}</p>
      <p>Price: $ {{ book.price }}</p>
    </div>
  {% endfor %}
{% endblock content %}
```

<br>

<div dir='rtl'>

اگر هنوز در یک حساب کاربری لاگین هستید, از آن خارج شوید.
در حال حاضر صفحه‌ی نتایج جستجو در `http://127.0.0.1:8000/books/search/` در دسترس است.

</div>

<br>

PIC!!!

<br>

<h1 dir='rtl'>

فیلتر مقدماتی

</h1>

<div dir='rtl'>

در جنگو از [QuerySet](https://docs.djangoproject.com/en/3.1/topics/db/queries/#retrieving-objects) برای فیلتر کردن نتایج مدل دیتابیس استفاده می‌شود.
در حال حاضر صفحه‌ی نتایج جستجو ما شبیه به یکی از آن نیست, زیرا همه‌ی نتایج را از مدل `Book` نشان می‌دهد.
در نهایت, می‌خواهیم فیلتر را بر اساس جستجوی کاربر اجرا کنیم, اما ابتدا این کار را از طریق چند آپشن فیلترینگ انجام می‌دهیم.
    
</div>

<div dir='rtl'>

راه‌های مختلفی برای سفارشی‌سازی یک queryset وجود دارد, از جمله [manager](https://docs.djangoproject.com/en/3.1/topics/db/managers/#django.db.models.Manager) در خود مدل, اما برای ساده نگه داشتن کارها, می‌توانیم فقط یک فیلتر با یک خط اضافه کنیم.

</div>

<div dir='rtl'>

می‌توانیم `queryset` پیش‌فرض را در `ListView` بازنویسی کنیم که به طور پیش‌فرض تمامی نتایج رانشان می‌دهد.
مستندات queryset کاملا دقیق و قوی است, اما اغلب استفاده از [contains](https://docs.djangoproject.com/en/3.1/ref/models/querysets/#contains) (که به حروف بزرگ حساس است) یا [icontains](https://docs.djangoproject.com/en/3.1/ref/models/querysets/#icontains) (که به حروف بزرگ و کوچک حساس نیست) نقاط شروع خوبی هستند.
فیلتر را بر اساس `title` که حاوی نام “beginners” است پیاده‌سازی می‌کنیم.   

</div>

<br>

**Code**

```python
# books/views.py
class SearchResultsListView(ListView):
    model = Book
    context_object_name = 'book_list'
    template_name = 'books/search_results.html'
    queryset = Book.objects.filter(title__icontains='beginners') # new
```

<br>

<div dir='rtl'>

صفحه نتایج را رفرش کنید و حالا فقط کتابی که شامل عنوان “beginners” است نمایش داده می‌شود.

</div>

PIC!!!
