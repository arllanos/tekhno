# Exploring FastAPI
## Setup
### Install virtualenv and create env for the app
```bash
#install virtualenv
pip3 install virtualenv
# create environment
virtualenv env
```
### Activate the environment
```
source env/bin/activate
```
### Install requirements
```
pip3 install fastapi
pip3 install uvicorn
```

## Basic FastAPI App
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"hello": "world"}
```
Run with:
```
uvicorn main:app --reload
````
The application should be reachable at [http://localhost:8000/](http://localhost:8000/)

## Diving Deeper
```python
from fastapi import FastAPI
from pydantic import BaseModel

class MyItem(BaseModel):
    name: str
    info: str = None
    price: float
    qty: int

app = FastAPI()

@app.get("/")
def read_root():
    return {"hello": "world"}

@app.get("/course/{course_id}")
def my_course(course_id: int):
    return {"course": course_id}

@app.get("/my/page/items/")
async def read_items(page: int = 0, limit: int = 0, skip: int = 1):
    dummy_data = [i for i in range(100)]
    return dummy_data[page*10: page*10 + limit: skip]

@app.post("/purchase/item/")
async def create_item(item: MyItem):
    return {"amount": item.qty*100, "sucess": True}
```
### Finalize
```
pip3 freeze > requirements.txt
```


### References
[Tutorial](https://app.pluralsight.com/guides/explore-python-libraries:-fastapi)
