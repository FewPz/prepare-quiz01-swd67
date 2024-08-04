# Field Lookups

ความสุดยอดของ Django คือการที่เราสามารถเขียน WHERE condition ในการ query ข้อมูลจาก database ได้อย่างง่ายดายโดยการกำหนด lookup type ใน methods `filter()`, `exclude()` และ `get()`

โดย format การกำหนด lookup type เป็นดังนี้ `field__lookuptype=value` (underscore 2 ตัว ระหว่าง ชื่อ field และ lookup type)

ในกรณีที่ต้องการ filter ด้วย foreign key จะต้องเติม `_id` ต่อท้ายชื่อ field ด้วย เช่น

```python
>>> Entry.objects.filter(blog_id=4)
```

- exact
- iexact
- contains
```sql
-- Entry.objects.filter(headline__contains='Lennon')
SELECT ... WHERE headline LIKE '%Lennon%';
```
- icontains
```sql
-- Entry.objects.filter(headline__icontains='Lennon')
SELECT ... WHERE headline ILIKE '%Lennon%';
```
- startswith
- endswith
- in
```sql
-- Entry.objects.filter(headline__in=('a', 'b', 'c'))
SELECT ... WHERE headline IN ('a', 'b', 'c');
```
- gt, gte, lt, lte
```sql
SELECT ... WHERE id > 4;
```
- range
```sql
-- Entry.objects.filter(pub_date__range=(start_date, end_date))
SELECT ... WHERE pub_date BETWEEN '2005-01-01' AND '2005-03-31';
```
- date, year, month, day, week, week_day
```sql
-- Entry.objects.filter(pub_date__year=2005)
-- Entry.objects.filter(pub_date__year__gte=2005)
SELECT ... WHERE pub_date BETWEEN '2005-01-01' AND '2005-12-31';
SELECT ... WHERE pub_date >= '2005-01-01';
```
- isnull
```sql
-- Entry.objects.filter(pub_date__isnull=True)
SELECT ... WHERE pub_date IS NULL;
```
- regex
```sql
-- Entry.objects.get(title__regex=r"^(An?|The) +")
SELECT ... WHERE title ~ '^(An?|The) +'; -- PostgreSQL
```

**HINT:** ถ้าอยากลองพิมพ์ SQL query ออกมาดูสามารถทำได้โดยใช้ `.query`

```python
q = Entry.objects.filter(headline__in=('a', 'b', 'c'))

print(q.query)
```

## Lookups that span relationships

QuerySet API ของ Django ช่วยให้เราสามารถ query ข้อมูลที่เกี่ยวข้องกับตารางอื่นที่มี relationship กันได้อย่างสะดวก โดย Django จะไปจัดการเรื่องการ generate SQL JOINs ให้หลังบ้าน

ยกตัวอย่างเช่น ถ้าเราต้องการดึงข้อมูล Entry ทั้งหมดของ Blog ที่มี name = "Beatles Blog"

```python
>>> Entry.objects.filter(blog__name="Beatles Blog")
>>> Entry.objects.filter(blog__name__contains="Beatles Blog")
```

สังเกตว่าเราเพียงเอาชื่อ field foreign key มาต่อด้วยชื่อ field ของตารางที่อ้างอิงไปถึง โดยคั่นด้วย underscore 2 ตัว - `blog__name` - และยังสามารถต่อด้วย lookup type ได้อีก

นอกจากนั้นเรายังสามารถ query ข้อมูลไปกี่ต่อก็ได้ ดังตัวอย่าง

```python
Blog.objects.filter(entry__authors__name="Lennon")
Blog.objects.filter(entry__authors__name__isnull=True)
```

## Filters can reference fields on the model

ในกรณีที่เราต้องการเปรียบเทียบค่าของ field ใน model กับ field อื่นใน model เดียวกัน เราสามารถใช้ **F expressions** ได้ `F()`

```python
>>> from django.db.models import F
>>> Entry.objects.filter(number_of_comments__gt=F("number_of_pingbacks"))
>>> Entry.objects.filter(authors__name=F("blog__name")) # span relationships
```

โดย Django นั้น support การใช้ +, -, *, / ร่วมกับ `F()` ด้วย เช่น

```python
>>> Entry.objects.filter(number_of_comments__gt=F("number_of_pingbacks") * 2)
>>> Entry.objects.filter(rating__lt=F("number_of_comments") + F("number_of_pingbacks"))
```

## Complex lookups with Q objects

Keyword argument ที่ส่งเข้าไปใน method `filter()` ทุกตัวจะถูกเอามา generate เป็น `SELECT ... WHERE ... AND ...` เสมอ เช่น 

โดยปกติถ้าเราใช้ `,` ขั้นระหว่าง filter condition จะเป็นการ AND กัน

```sql
-- Entry.objects.filter(headline__contains='Lennon', pub_date__year=2005)
SELECT * FROM entry WHERE headline LIKE '%Lennon%' AND pub_date BETWEEN '2005-01-01' AND '2005-12-31';
```

ในกรณีที่เราต้องการทำการ query ที่ซับซ้อน อาจจะต้องการใช้ `OR` หรือ `NOT` ร่วมด้วย เราจะต้องใช้ `Q objects`

กรณี OR

```python
Entry.objects.filter(Q(headline__startswith="Who") | Q(headline__startswith="What"))
# SELECT ... WHERE headline LIKE 'Who%' OR headline LIKE 'What%'
```

กรณี NOT

```python
Entry.objects.filter(Q(headline__startswith="Who") | ~Q(pub_date__year=2005))
# SELECT ... WHERE headline LIKE 'Who%' OR pub_date NOT BETWEEN '2005-01-01' AND '2005-12-31'; 
```

กรณี nested conditions

```python
Poll.objects.get(
    Q(question__startswith="Who"),
    (Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))),
)
```

แปลงเป็น SQL

```sql
SELECT * from polls WHERE question LIKE 'Who%' 
    AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```
