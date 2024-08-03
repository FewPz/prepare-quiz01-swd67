## 🏑 - Field Type Model

ใน Django, การใช้งาน Field types และ options ใน model นั้นมีความหลากหลายตามที่ต้องการใช้งาน โดยมีรายละเอียดดังนี้

### BooleanField
```python
class MyModel(models.Model):
    is_active = models.BooleanField(default=True)
```
* `default`: กำหนดค่าเริ่มต้นให้กับ field นี้
### CharField
```python
class MyModel(models.Model):
    name = models.CharField(max_length=100, blank=True)
```
* `max_length`: กำหนดความยาวสูงสุดของ string
* `blank`: กำหนดให้สามารถเว้นว่าง field นี้ได้
### EmailField
```python
class MyModel(models.Model):
    email = models.EmailField(max_length=254, unique=True)
```
* `max_length`: กำหนดความยาวสูงสุดของ email
* `unique`: กำหนดให้ค่าใน field นี้ห้ามซ้ำ
### URLField
```python
class MyModel(models.Model):
    website = models.URLField(max_length=200)
```
* `max_length`: กำหนดความยาวสูงสุดของ URL
### UUIDField
```python
import uuid

class MyModel(models.Model):
    uuid = models.UUIDField(default=uuid.uuid4, editable=False, unique=True)
```
* `default`: กำหนดค่าเริ่มต้นให้เป็น UUID ใหม่
* `editable`: กำหนดว่า field นี้ไม่สามารถแก้ไขได้
* `unique`: กำหนดให้ค่าใน field นี้ห้ามซ้ำ
### TextField
```python
class MyModel(models.Model):
    description = models.TextField(null=True, blank=True)
```
* `null`: กำหนดให้ค่าใน field นี้สามารถเป็น NULL ได้
* `blank`: กำหนดให้สามารถเว้นว่าง field นี้ได้
### DateField (วัน)
```python
class MyModel(models.Model):
    birth_date = models.DateField(auto_now_add=True)
```
* `auto_now`: กำหนดให้ field นี้บันทึกค่า `datetime.now()` ทุกครั้งที่มีการแก้ไข (INSERT + UPDATE)
* `auto_now_add`: กำหนดให้ field นี้บันทึกค่า `datetime.now()` ตอนที่สร้างใหม่ (INSERT)
### DateTimeField (วันและเวลา)
```python
class MyModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```
* `auto_now`: กำหนดให้ field นี้บันทึกค่า `datetime.now()` ทุกครั้งที่มีการแก้ไข (INSERT + UPDATE)
* `auto_now_add`: กำหนดให้ field นี้บันทึกค่า `datetime.now()` ตอนที่สร้างใหม่ (INSERT)
### TimeField (เวลา)
```python
class MyModel(models.Model):
    start_time = models.TimeField(auto_now=False, auto_now_add=False)
```
* `auto_now`: กำหนดให้ field นี้บันทึกค่า `datetime.now()` ทุกครั้งที่มีการแก้ไข (INSERT + UPDATE)
* `auto_now_add`: กำหนดให้ field นี้บันทึกค่า `datetime.now()` ตอนที่สร้างใหม่ (INSERT)
### FileField
```python
class MyModel(models.Model):
    upload = models.FileField(upload_to="uploads/%Y/%m/%d/")
```
* `upload_to`: กำหนด path ที่จะ save file
### ImageField
```python
class MyModel(models.Model):
    image = models.ImageField(upload_to="images/", height_field=None, width_field=None)
```
* `upload_to`: กำหนด path ที่จะ save image file
* `height_field`: กำหนด field ที่จะเก็บข้อมูลความสูงของภาพ
* `width_field`: กำหนด field ที่จะเก็บข้อมูลความกว้างของภาพ
### DecimalField
```python
class MyModel(models.Model):
    price = models.DecimalField(max_digits=10, decimal_places=2)
```
* `max_digits`: กำหนดจำนวนหลักสูงสุดของตัวเลข
* `decimal_places`: กำหนดจำนวนหลักทศนิยมสูงสุด
### IntegerField
```python
class MyModel(models.Model):
    quantity = models.IntegerField(default=0)
```
* `default`: กำหนดค่าเริ่มต้นให้กับ field นี้
### PositiveIntegerField
```python
class MyModel(models.Model):
    positive_quantity = models.PositiveIntegerField(default=0)
```
* `default`: กำหนดค่าเริ่มต้นให้กับ field นี้
### JSONField
```python
class MyModel(models.Model):
    data = models.JSONField(default=dict)
```
* `default`: กำหนดค่าเริ่มต้นให้กับ field นี้เป็น empty dictionary

### Field Options:
* `primary_key`: กำหนดว่า field นี้เป็น primary key ของ table
* `unique`: กำหนดให้ค่าใน field นี้ห้ามซ้ำ
* `null`: กำหนดให้ค่าใน field นี้สามารถเป็น NULL ได้
* `blank`: กำหนดให้สามารถเว้นว่าง field นี้ได้
* `default`: กำหนดค่าเริ่มต้นให้กับ field นี้
* `choices`: กำหนด ENUM ให้เลือกเฉพาะค่าที่กำหนด
* `db_index`: กำหนดให้สร้าง index ใน database สำหรับ field นี้

ยกตัวอย่างการใช้งาน choices
```python
class Student(models.Model):
    FRESHMAN = "FR"
    SOPHOMORE = "SO"
    JUNIOR = "JR"
    SENIOR = "SR"
    GRADUATE = "GR"
    YEAR_IN_SCHOOL_CHOICES = [
        (FRESHMAN, "Freshman"),
        (SOPHOMORE, "Sophomore"),
        (JUNIOR, "Junior"),
        (SENIOR, "Senior"),
        (GRADUATE, "Graduate"),
    ]
    
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )
    
    def is_upperclass(self):
        return self.year_in_school in {self.JUNIOR, self.SENIOR}
```
* ⚠️ การเรียก Query ที่เป็นลักษณะ choice นั้นจะต้องดูชื่อ field ดีๆ จะสังเกตุว่าชื่อ FRESHMAN จะเป็นตัว `FR` แต่เวลาแสดงผลจะเป็น `Freshman` แทน
