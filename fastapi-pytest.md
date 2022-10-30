---
title: 'fastbooks (part 2) - fastapi, pytest, heroku'
date: '2021-11-07'
---

## Using pytest with FastAPI 

In the [last post](/posts/sqlmodel) we saw how to make a simple `FastAPI` app using the new `sqlmodel` to handle database conenction (and data validation). We also set up `alembic` to handle database migrations, which will come in handy when we deploy to [Heroku](https://www.heroku.com/) later on. We'll start with integrating `pytest`
It's not necessary to have the same code as our project here, but it might make more sense if the project structure is similar:

```bash
.
├── Procfile
├── alembic
│   ├── README
│   ├── env.py
│   ├── script.py.mako
│   └── versions
│       ├── 0f599158e6f3_rename_acccess_token_to_hashed_password.py
│       ├── 41790a17a9dc_add_descrption_column.py
│       ├── 649a10b6cd01_.py
│       └── 6d7727a40b30_remove_access_token_column.py
├── alembic.ini
├── app
│   ├── __init__.py
│   ├── database.py
│   ├── main.py
│   ├── models.py
│   ├── routes
│   │   ├── books.py
│   │   └── users.py
│   ├── settings.py
│   ├── static
│   │   ├── css
│   │   │   ├── docs.css
│   │   │   └── theme.css
│   │   └── img
│   │       ├── books.png
│   │       ├── cloud.png
│   │       └── favicon.ico
│   └── templates
│       ├── book
│       │   ├── addbook.html
│       │   └── updatebook.html
│       ├── home
│       │   └── index.html
│       ├── shared
│       │   └── layout.html
│       └── users
│           └── login.html
├── bookdatabase.db
└── requirements.txt
```

The important things are:

- There is a directory `app` which has all the code required to run the app, which is a python module (it has
  an `__init__.py` file)

We'll be using `pytest` to handle our tests, so make sure it's installed (`pip install pytest ` if not). Make a folder
next to `app` called `tests`. This name is important because `pytest` will automatically look for a folder with this name to collect the tests.

FastAPI allows for really easy integration with `pytest`, by providing a `TestClient`. Make a file
called `tests/test_main.py` with the following content

```python
import pytest
from fastapi import TestClient
from app.main import app


@pytest.fixture(name="client")
def testclient():
    yield TestClient(app)


@pytest.fixture(name='session')
def session():
    with Session(db) as session:
        yield session

def get_session_override():
	db_url = "sqlite:///"

def test_create_book(session):
    book = Book(title="My first book about WORDS", author="Pytest McPyface")
    session.add(book)


	session.refresh(book)
	assert book.id
	assert book.title == "My first book about WORDS"
	assert book.author ==  "Pytest McPyface"


def test_homepage(client):
    homepage = client.get("/")
    assert homepage.status_code == 200
    assert "" != homepage.content

```

This file will define two basic tests to test the functionality of creating a book in the database and also that the homepage returns the correct status code.
This can now be run with
```bash
pytest
```

### Pytest Fixtures
The pytest fixtures that we've defined in `test_main.py` (that functions with the `@pytest.fixture()` decorators) are a very convenient way of defining things that are used in mulitple tests. We are using them here to define a `session` to connect to the database and a `client` to make our requests against. Note that we have also overridden the `get_session` dependency so that we aren't using the same database file for the tests as we are for developing.
The important thing is that pytest will take care of creating the fixtures and taking them down after each test (or each testing session, depending on the scope). 


## Deploying to Heroku

Create a `Procfile` in the base directory with the following content

```
web: gunicorn app.main:app -w 4 -k uvicornWorkers
on_start: alembic upgrade
```

