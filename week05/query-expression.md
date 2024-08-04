# Query Expressions

ORM ของ Django นั้น support การใช้งาน function ในการคำนวณ (+, -, *, /) การ aggregate ต่างๆ เช่น SUM(), MIN(), MAX(), COUNT(), AVG() และการทำ Subquery

ซึ่งในส่วนนี้เราจะเรียกว่าการทำ Query Expression

สมมติเรามี model `Company` ดังนี้

```python
from datetime import date

from django.db import models


class Company(models.Model):
    name = models.CharField(max_length=100)
    ticker = models.CharField(max_length=20, null=True)
    num_employees = models.IntegerField()
    num_tables = models.IntegerField()
    num_chairs = models.IntegerField()

    def __str__(self):
        return self.name
```

เรามา setup project กันสำหรับ tutorial นี้

1. สร้าง project ใหม่ชื่อ `week5_tutorial` (สร้าง vitual environment ใหม่ด้วย)
2. สร้าง app ชื่อ `companies` และทำการตั้งค่าใน `settings.py`
3. แก้ไขไฟล์ `/companies/models.py` และเพิ่ม code ด้่านบนลงไป โดย models นี้เราจะใช้ในการทำ tutorial วันนี้กัน
4. `makemigrations` และ `migrate`

เปิด Django shell จากนั้นสร้างแถวข้อมูลด้วยคำสั่ง

```python
>>> from company.models
>>> Company.objects.create(name="Company AAA", num_employees=120, num_chairs=150, num_tables=60)
>>> Company.objects.create(name="Company BBB", num_employees=50, num_chairs=30, num_tables=20)
>>> Company.objects.create(name="Company CCC", num_employees=100, num_chairs=40, num_tables=40)
```

**ลองทดสอบคำสั่งด้านล่างนี้ดู แล้วดูสิว่าได้ผลลัพธ์เป็นอย่างไร**

```python
>>> from company.models import Company
>>> from django.db.models import Count, F, Value
>>> from django.db.models.functions import Length, Upper
>>> from django.db.models.lookups import GreaterThan

# Find companies that have more employees than chairs.
>>> Company.objects.filter(num_employees__gt=F("num_chairs"))

# Find companies that have at least twice as many employees as chairs.
>>> Company.objects.filter(num_employees__gt=F("num_chairs") * 2)

# Find companies that have more employees than the number of chairs and tables combined.
>>> Company.objects.filter(num_employees__gt=F("num_chairs") + F("num_tables"))

# How many chairs are needed for each company to seat all employees?
>>> company = (
...     Company.objects.filter(num_employees__gt=F("num_chairs"))
...     .annotate(chairs_needed=F("num_employees") - F("num_chairs"))
...     .first()
... )
>>> company.num_employees
50
>>> company.num_chairs
30
>>> company.chairs_needed
20

# Create a new company using expressions.
>>> company = Company.objects.create(name="Google", ticker=Upper(Value("goog")))
# Be sure to refresh it if you need to access the field.
>>> company.refresh_from_db()
>>> company.ticker
'GOOG'

# Expressions can also be used in order_by(), either directly
>>> Company.objects.order_by(Length("name").asc())
>>> Company.objects.order_by(Length("name").desc())

# Lookup expressions can also be used directly in filters
>>> Company.objects.filter(GreaterThan(F("num_employees"), F("num_chairs")))
# or annotations.
>>> Company.objects.annotate(
...     need_chairs=GreaterThan(F("num_employees"), F("num_chairs")),
... )
```

## Aggregate expression

สำหรับ tutorial นี้ให้ทำตามขั้นตอนนี้ 

1. สร้าง app ใหม่ชื่อ `books` ใน project `week5_tutorial` อันเดิม
2. เพิ่ม app books ใน `settings.py`
3. แก้ไขไฟล์ `/books/models.py` และเพิ่ม code ด้่านล่างลงไป โดย models เหล่านี้เราจะใช้ในการทำ tutorial นี้กัน
4. `makemigrations` และ `migrate`

```python
from django.db import models


class Author(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()


class Publisher(models.Model):
    name = models.CharField(max_length=300)


class Book(models.Model):
    name = models.CharField(max_length=300)
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rating = models.FloatField()
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
    pubdate = models.DateField()


class Store(models.Model):
    name = models.CharField(max_length=300)
    books = models.ManyToManyField(Book)
```

ทำการ import ข้อมูลเข้าตารางทั้งหมดด้วย SQL ในไฟล์ `books.sql`


**ลองทดสอบคำสั่งด้านล่างนี้ดู แล้วดูสิว่าได้ผลลัพธ์เป็นอย่างไร**

```python
>>> from books.models import Book
# Total number of books.
>>> Book.objects.count()
59

# Total number of books with publisher=Penguin Books
>>> Book.objects.filter(publisher__name="Penguin Books").count()
20

# Average price across all books, provide default to be returned instead
# of None if no books exist.
>>> from django.db.models import Avg
>>> Book.objects.aggregate(Avg("price", default=0))
{'price__avg': Decimal('9.7018644067796610')}

# Max price across all books, provide default to be returned instead of
# None if no books exist.
>>> from django.db.models import Max
>>> Book.objects.aggregate(Max("price", default=0))
{'price__max': Decimal('14.99')}

# All the following queries involve traversing the Book<->Publisher
# foreign key relationship backwards.

# Each publisher, each with a count of books as a "num_books" attribute.
>>> from books.models import Publisher
>>> from django.db.models import Count
>>> pubs = Publisher.objects.annotate(num_books=Count("book"))
>>> pubs
<QuerySet [<Publisher: BaloneyPress>, <Publisher: SalamiPress>, ...]>
>>> pubs[0].num_books
20

# Each publisher, with a separate count of books with a rating above and below 4
>>> from django.db.models import Q
>>> above = Publisher.objects.annotate(above_4=Count("book", filter=Q(book__rating__gt=4)))
>>> below = Publisher.objects.annotate(below_4=Count("book", filter=Q(book__rating__lte=4)))
>>> above[0].above_4
16
>>> below[0].below_4
4

# The top 5 publishers, in order by number of books.
>>> pubs = Publisher.objects.annotate(num_books=Count("book")).order_by("-num_books")[:5]
>>> pubs[0].num_books
39
```
