# Model Relationships

## Many-to-one relationships

สำหรับการนิยาม Many-to-one Relationships จะใช้การประกาศ ForeignKey เป็น field ใน models

ต่อเนื่องจากตัวอย่าง `books/models.py` ใน model `Book` จะเห็นว่ามีการประกาศ ForeignKey เก็บ `publisher_id` ซึ่งชี้ไปยัง instance ใน model `Publisher`

```python
# /books/models.py
...
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
...
```

สมมติเราจะสร้าง book เล่มใหม่ที่มีการ FK ไปหา publisher "Penguin Books"

```python
>>> from books.models import Publisher, Book
>>> from datetime import datetime
# Get the publisher instance
>>> penguin_pub = Publisher.objects.get(name="Penguin Books")

# Create a new book
>>> book = Book.objects.create(
    name="Web Programming is HARD",
    pages=200,
    price=10.00,
    rating=4.5,
    publisher=penguin_pub,
    pubdate=datetime.now().date()
)
>>> book
<Book: Book object (61)>
>>> book.publisher.name
'Penguin Books'
>>> book.publisher.id
1
```

ในกรณีที่เราต้องการรายการ book ทั้งหมดที่เกี่ยวข้องกับ publisher "Penguin Books" สามารถทำได้ดังนี้

```python
>>> from books.models import Publisher, Book
# Get the publisher instance
>>> penguin_pub = Publisher.objects.get(name="Penguin Books")

# Get all books published by "Penguin Books"
>>> books = penguin_pub.book_set.all()
<QuerySet [<Book: Book object (2)>, <Book: Book object (3)>, <Book: Book object (4)>, <Book: Book object (5)>, <Book: Book object (6)>, <Book: Book object (7)>, <Book: Book object (8)>, <Book: Book object (9)>, <Book: Book object (10)>, <Book: Book object (11)>, <Book: Book object (12)>, <Book: Book object (13)>, <Book: Book object (14)>, <Book: Book object (15)>, <Book: Book object (16)>, <Book: Book object (17)>, <Book: Book object (18)>, <Book: Book object (19)>, <Book: Book object (20)>, <Book: Book object (21)>, '...(remaining elements truncated)...']>

# How may books?
>>> penguin_pub.book_set.count()
21

# Get top 10 best rating books
>>> penguin_pub.book_set.order_by("-rating")[:10]
<QuerySet [<Book: Book object (14)>, <Book: Book object (4)>, <Book: Book object (15)>, <Book: Book object (9)>, <Book: Book object (12)>, <Book: Book object (3)>, <Book: Book object (8)>, <Book: Book object (18)>, <Book: Book object (61)>, <Book: Book object (10)>]>

# Get books with name starting with "The"
>>> penguin_pub.book_set.filter(name__startswith="The")
<QuerySet [<Book: Book object (2)>, <Book: Book object (6)>, <Book: Book object (9)>, <Book: Book object (15)>, <Book: Book object (18)>]>

# Get only ids
>>> penguin_pub.book_set.filter(name__startswith="The").values_list("id", flat=True)
<QuerySet [2, 6, 9, 15, 18]>

# Get id and name
>>> penguin_pub.book_set.filter(name__startswith="The").values("id", "name")
<QuerySet [{'id': 2, 'name': 'The Great Gatsby'}, {'id': 6, 'name': 'The Catcher in the Rye'}, {'id': 9, 'name': 'The Odyssey'}, {'id': 15, 'name': 'The Hobbit'}, {'id': 18, 'name': 'The Hitchhiker Guide to the Galaxy'}]>
```

สมมติว่าเราต้องการต้นหาจากทางฝั่ง book บ้าง ถ้าเราต้องหนังสือที่ rating >= 4.5 และ published โดยสำนักพิมพ์ "Oxford University Press"

```python
>>> from books.models import Book

>>> results = Book.objects.filter(publisher__name="Oxford University Press", rating__gte=4.5)
>>> results
<QuerySet [<Book: Book object (24)>, <Book: Book object (27)>, <Book: Book object (30)>, <Book: Book object (33)>, <Book: Book object (44)>, <Book: Book object (56)>]>
```

หรือจะ filter จากทางฝั่ง publisher ก็ได้

```python
>>> from books.models import Publisher
>>> Publisher.objects.filter(book__id=20)
<QuerySet [<Publisher: Publisher object (1)>]>

>>> Publisher.objects.filter(book__pubdate='1967-05-30')
<QuerySet [<Publisher: Publisher object (1)>]>
# SELECT "books_publisher"."id", "books_publisher"."name" FROM "books_publisher" INNER JOIN "books_book" ON ("books_publisher"."id" = "books_book"."publisher_id") WHERE "books_book"."pubdate" = 1967-05-30
```

## One-to-one Relationships

เรามาเพิ่มความสัมพันธ์แบบ one-to-one ให้กับตารางใน database ของเรากัน

แก้ไขเพิ่ม code ด้านล่างนี้ในไฟล์ `/books/models.py`

```python
...
# เพิ่มไว้ล่างสุดเลยนะครับ
class StoreContact(models.Model):
    mobile = models.CharField(max_length=20)
    email = models.EmailField(max_length=50, blank=True, null=True)
    address = models.TextField()
    store = models.OneToOneField(Store, on_delete=models.CASCADE)
```

เรามาสร้างเพิ่ม store contact กันสำหรับ "KMITL Book Store"

```python
>>> from books.models import Store, StoreContact
# Get KMITL Book Store
>>> store = Store.objects.get(name="KMITL Book Store")
# Create contact information
>>> contact = StoreContact(
    mobile="021113333",
    email="book_shop@it.kmitl.ac.th",
    address="KMITL",
    store=store
)
>>> contact.save()
```

การเข้าถึงข้อมูล สามารถทำได้จากทั้ง 2 ฝั่ง

```python
>>> contact.store
<Store: Store object (2)>

>>> store.storecontact
<StoreContact: StoreContact object (1)>

# สามารถ filter ได้คล้ายกับ one-to-many
>>> Store.objects.filter(storecontact__mobile="021113333")
<QuerySet [<Store: Store object (2)>]>
```

## Many-to-many Relationships

จะเห็นได้ว่า model Book และ Author มีความสัมพันธ์กันแบบ many-to-many

```python
class Author(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()

...

class Book(models.Model):
    name = models.CharField(max_length=300)
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rating = models.FloatField()
    authors = models.ManyToManyField(Author)
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
    pubdate = models.DateField()
```

เมื่อเราสั่ง `python manage.py migrate` Django จะทำการสร้างตารางกลางให้อัตโนมัตื อย่างในกรณีนี้จะมีตาราง `books_book_author` ถูกสร้างขึ้นมา

สำหรับการเพิ่ม author ให้กับ book

```python
>>> from books.models import Book, Author
>>> a1 = Author.objects.get(pk=1)
>>> a2 = Author.objects.get(pk=2)

>>> book = Book.objects.get(pk=10)
>>> book.authors.add(a1, a2)

>>> book.authors.all()
<QuerySet [<Author: Author object (5)>, <Author: Author object (1)>, <Author: Author object (2)>]>
```

สำหรับการเพิ่ม book ใหม่ให้กับ author

```python
>>> from books.models import Book, Author
>>> b1 = Book.objects.get(pk=11)
>>> b2 = Book.objects.get(pk=12)

>>> author = Author.objects.get(pk=10)
>>> author.book_set.add(b1, b2)

>>> author.book_set.all()
<QuerySet [<Book: Book object (11)>, <Book: Book object (12)>, <Book: Book object (19)>, <Book: Book object (20)>, <Book: Book object (30)>, <Book: Book object (50)>]>
```

สามารถทำการ filter ได้จากทั้ง 2 ฝั่งเช่นเดียวกับ one-to-one และ one-to-many

```python
>>> Book.objects.filter(authors__name="F. Scott Fitzgerald")
<QuerySet [<Book: Book object (51)>, <Book: Book object (2)>, <Book: Book object (21)>, <Book: Book object (31)>, <Book: Book object (10)>]>

>>> Author.objects.filter(book__name="Crime and Punishment")
<QuerySet [<Author: Author object (5)>, <Author: Author object (1)>, <Author: Author object (2)>]>
```

สามารถทำการ ยกเลิก ความสัมพันธ์ ได้โดยใช้ `remove()` หรือ `clear()` ถ้าต้องการลบความสัมพันธ์ทั้งหมด

```python
>>> book = Book.objects.get(pk=10)

>>> book.authors.remove(a1)
>>> book.authors.all()
<QuerySet [<Author: Author object (2)>, <Author: Author object (5)>]>

>>> book.authors.clear()
>>> book.authors.all()
<QuerySet []>
```
