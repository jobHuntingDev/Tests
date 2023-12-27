# Testing Notes

## Unit Tests

### How to write tests

Code to be tested (calc.py)

```python
def add(x,y):
	"""Add Function"""
	return x + y

def subtract(x,y):
	"""Subtract Function"""
	return x - y

def multiply(x,y):
	"""Multiply Funtion"""
	return x * y

def devide(x,y):
	"""Divide Function"""
	if y == 0:
		raise ValueError('Can not divide by zero!')
	return x / y
```

Test code (test_calc.py)

```python
import unittest
import calc

class TestCalc(unittest.TestCase):
	
	def  test_add(self):
		self.assertEquals(calc.add(10,5), 15)
		self.assertEquals(calc.add(-1,1), 0)
		self.assertEquals(calc.add(-1,-1), -2)

	def  test_subtract(self):
		self.assertEquals(calc.subtract(10,5), 5)
		self.assertEquals(calc.subtract(-1,1), -2)
		self.assertEquals(calc.subtract(-1,-1), 0)

	def  test_multiply(self):
		self.assertEquals(calc.multiply(10,5), 50)
		self.assertEquals(calc.multiply(-1,1), -1)
		self.assertEquals(calc.multiply(-1,-1), 1)

	def  test_divide(self):
		self.assertEquals(calc.divide(10,5), 2)
		self.assertEquals(calc.divide(-1,1), -1)
		self.assertEquals(calc.divide(-1,-1), 1)
		# testing for division not floor division
		self.assertEquals(calc.divide(5,2), 2.5)

		# testing for  / 0
		with self.assertRaises(ValueError):
			calc.divide(10, 0)

if __name__ = '__main__':
	unittest.main()
```

**More complex testing**

Code to be tested

```python
import request

class Employee:
	""" A sample Employee class"""

	raise_amt = 1.05

	def __init__(self, first, last, pay):
		self.first = first
		self.last = last
		self.pay = pay

	@property
	def email(self):
		return '{}.{}@email.com'.format(self.first, self.last)

	@property
	def fullname(self):
		return '{} {}'.format(self.first, self.last)

	def apply_raise(self):
		self.pyt = int(self.pay * self.raise_amt) 

	def monthly_schedule(self, month):
		response = request.get(f'http://company.com/{self.last}/{month}')
		if response.ok:
			return response.txt
		else:
			return 'Bad Response!'	
```

Tests 

```python
import unittest
from untitest.mock import patch
from employee import Employee

class TestEmployee(unittest.TestCase):

# The setUp and tearDown run before and after befor doing test and after doing tests respectivly

	@classmethod
	def setUpClass(cls):
		print('setupClass')

	@classmethod
	def tearDownClass():
		print('teardownClass')

# The setUp and tearDown run before and after each test respectivly

	# Creating for testing
	def setUp(self):					
		self.emp_1 = Employee('Cory', 'Schafer', 50000)
		self.emp_2 = Employee('Sue', 'Smith', 60000)

	# Delete after testing
	def tearDown(self):
		pass

	def test_email(self):
		self.assertEqual(self.emp_1.email, 'Cory.Schafer@email.com')
		self.assertEqual(self.emp_2.email, 'Sue.Smith@email.com')

		self.emp_1.first = 'John'
		self.emp_2.first = 'Jane'

		self.assertEqual(self.emp_1.email, 'John.Schafer@email.com')
		self.assertEqual(self.emp_2.email, 'Jane.Smith@email.com')

	def test_fullname(self):
		self.emp_1.first = 'John'
		self.emp_2.first = 'Jane'

		self.assertEqual(self.emp_1.fullname, 'John Schafer')
		self.assertEqual(self.emp_2.fullname, 'Jane Smith')

	def test_apply_raise(self):
		self.emp_1.apply_raise()
		self.emp_2.apply_raise()

		self.assertEqual(self.emp_1.pay, 52500)
		self.assertEqual(self.emp_2.py, 63000)

	# Handling request failure
	def test_monthly_schedule(self):
		# Setting the request as a success regardless of actual request success
		with path('employee.requests.get') as mocked_get:
			mocked_get.return_value.ok = True
			mocked_get.return_value.text = 'Success'

			schedule = self.emp_1.monthly_schedule('May')
			mocked_get.assert_called_with('http://company.com/Schafer/May')
			self.assertEqual(schedule, 'Success')

			mocked_get.return_value.ok = False

			schedule = self.emp_2.monthly_schedule('May')
			mocked_get.assert_called_with('http://company.com/Smith/June')
			self.assertEqual(schedule, 'Bad Response!')
			

if __name__ = '__main__':
	unittest.main()
```

### Best practicies

- Naming convention test_<name of file to test>.py
- Tests should be isolated: Test should rely or affect other tests.
- Test driven development: Write tests before writing code.


## Pytest Notes

### Install

```python
pip install pytest
```

### Use


## Fast API Testing

### Basic

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

### Multiple files

**main.py file:**

```python
from typing import Annotated

from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel

fake_secret_token = "coneofsilence"

fake_db = {
    "foo": {"id": "foo", "title": "Foo", "description": "There goes my hero"},
    "bar": {"id": "bar", "title": "Bar", "description": "The bartenders"},
}

app = FastAPI()


class Item(BaseModel):
    id: str
    title: str
    description: str | None = None


@app.get("/items/{item_id}", response_model=Item)
async def read_main(item_id: str, x_token: Annotated[str, Header()]):
    if x_token != fake_secret_token:
        raise HTTPException(status_code=400, detail="Invalid X-Token header")
    if item_id not in fake_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return fake_db[item_id]


@app.post("/items/", response_model=Item)
async def create_item(item: Item, x_token: Annotated[str, Header()]):
    if x_token != fake_secret_token:
        raise HTTPException(status_code=400, detail="Invalid X-Token header")
    if item.id in fake_db:
        raise HTTPException(status_code=400, detail="Item already exists")
    fake_db[item.id] = item
    return item
```

**test_main.py file:**

```python
from fastapi.testclient import TestClient

from .main import app

client = TestClient(app)


def test_read_item():
    response = client.get("/items/foo", headers={"X-Token": "coneofsilence"})
    assert response.status_code == 200
    assert response.json() == {
        "id": "foo",
        "title": "Foo",
        "description": "There goes my hero",
    }


def test_read_item_bad_token():
    response = client.get("/items/foo", headers={"X-Token": "hailhydra"})
    assert response.status_code == 400
    assert response.json() == {"detail": "Invalid X-Token header"}


def test_read_inexistent_item():
    response = client.get("/items/baz", headers={"X-Token": "coneofsilence"})
    assert response.status_code == 404
    assert response.json() == {"detail": "Item not found"}


def test_create_item():
    response = client.post(
        "/items/",
        headers={"X-Token": "coneofsilence"},
        json={"id": "foobar", "title": "Foo Bar", "description": "The Foo Barters"},
    )
    assert response.status_code == 200
    assert response.json() == {
        "id": "foobar",
        "title": "Foo Bar",
        "description": "The Foo Barters",
    }


def test_create_item_bad_token():
    response = client.post(
        "/items/",
        headers={"X-Token": "hailhydra"},
        json={"id": "bazz", "title": "Bazz", "description": "Drop the bazz"},
    )
    assert response.status_code == 400
    assert response.json() == {"detail": "Invalid X-Token header"}


def test_create_existing_item():
    response = client.post(
        "/items/",
        headers={"X-Token": "coneofsilence"},
        json={
            "id": "foo",
            "title": "The Foo ID Stealers",
            "description": "There goes my stealer",
        },
    )
    assert response.status_code == 400
    assert response.json() == {"detail": "Item already exists"}
```

### Run test

```
pip install pytest 
```

Run tests with:

```
pytest
```
