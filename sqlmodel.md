---
title: 'fastbooks - FastAPI, sqlmodel, alembic and Jinja'
date: '2021-10-28'
---


We will be building a `fastapi` app using `sqlmodel` to handle a connection to the database, `alembic` for migrations, and `Jinja2` templates for the front end. Let's get straight into it!

To see the full code, see this [git repo](https://github.com/idjotherwise/fastbook-blog).

## Package requirements

We will start by installing all the packages we will need into a virtual environment.

```bash
python3 -m venv fastenv
source fastenv/bin/activate
pip install fastapi sqlmodel uvicorn Jinja2 aiofiles
```

Those packages are enough to get off the ground for local development. We will be using an `sqlite` database for local development, but when we deploy to Heroku we'll use `Postgres` so let's install some additional things we need for later.

```bash
pip install psycopg2 alembic gunicorn
```

We also installed `alembic` for database migrations, and `gunicorn` to use as a production server. The `psycopg2` package allows `sqlmodel` (which uses `SQLAlchemy` underneath) to talk to `Postgres` databases.

Those are all the packages we are going to need, so let's save them to a `requirements.txt` file before we forget.

```bash
pip freeze > requirements.txt
```


## Fastapi app

Start by making the files `main.py` and `__init__.py` inside a directory called `app`, so that the file structure looks like this:

```bash
.
├── app
│   ├── __init__.py
│   ├── main.py
├── bookenv
│   ├── bin
│   ├── include
│   ├── lib
│   └── pyvenv.cfg
└── requirements.txt
```

```python
# app/main.py
from fastapi import FastAPI
import uvicorn

app = FastAPI()

@app.get("/")
def home_page():
    return {"message": "Hello!"}

if __name__=="__main__":
    uvicorn.run('app.main:app', reload=True)
```

Running this with `python -m app.main` should start up a server on port `8000` on your `localhost`, so going to `127.0.0.1:8000` should show you the message
```json
{"message": "Hello!"}
```

## Making it look nice
Before setting up our database and models, let's make the first page look nice using Jinja templates.

```python {4-8, 12-13, 15-17, 20, 23} showLineNumbers
from fastapi import FastAPI
import uvicorn

from starlette.staticfiles import StaticFiles
from starlette.templating import Jinja2Templates
from starlette.requests import Request

templates = Jinja2Templates('app/templates')

app = FastAPI()

def configure():
    app.mount('/static', StaticFiles(directory='app/static'), name='static')

@app.get("/")
def home_page(request: Request):
    return templates.TemplateResponse('index.html', {'request': request})

if __name__=="__main__":
    configure()
    uvicorn.run('app.main:app', reload=True)
else:
    configure()
```

 

After importing the required packages, we set up the templates with `Jinja2Templates('app/templates')` and also mount the static files so that `fastapi` can find them with `app.mount('/static', StaticFiles(directory='app/static', name='static')`, so we need to make 2 directories:

```bash
mkdir app/static
mkdir app/templates
```

Inside templates, we'll create the following `index.html` file,

```html
<!DOCTYPE html>

<html lang="en-us">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="A website showcasing fastapi and sqlmodel">
    <meta name="author" content="idjotherwise">

    <title>Fastsqlmodel showcase</title>

    <link href="//oss.maxcdn.com/libs/twitter-bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet">
</head>

<body>
    <h1>
        A super stylish home page
    </h1>
    <p>
        Really interesting content, full of buzzwords like AI and MACHINE LEARNING!
    </p>
</body>
</html>
```

Now we can run the app to check that things are working so far,
```bash
python -m main.app
```
If everything has gone well, after opening up `localhost:8000`, you should be greeted with

![You should be greeted with this lovely webpage](/images/fastbooks_screenshot.png)

We'll refactor the `jinja` templates later, but that will do for now.

Next we'll set up some database stuff, including some models to hold our data.

## Database: sqlmodel

In a new file called `database.py`, we'll set up the database engine and session handler.

```python
# app/database.py

from sqlmodel import SQLModel, create_engine, Session

from .settings import settings

connect_args = {"check_same_thread": False}
engine = create_engine(settings.database_url), connect_args=connect_args)


def create_db_and_tables():
    SQLModel.metadata.create_all(engine)


def get_session():
    with Session(engine) as session:
        yield session
```

and in the file `settings.py` we'll define the `DATABASE_URL`,

```python
# app/settings.py

from pydantic import BaseSettings

class Settings(BaseSettings):
    database_url: str = 'sqlite:///database.db'
```

This may seem uncessarily complex, but it will make things easier later. When your class `Settings` inherits from `BaseSettings`, it will first try to get the attributes from your environment but if it doesn't find anything it will set it to the default you've given. So in our case (assuming you haven't set `DATABASE_URL` to anything else in your terminal) it will simply be the `sqlite` database url. On Heroku, environment variables like `DATABASE_URL` are set automatically (if you have installed a `Postgres` add-on).

Before we actually create the database, we should create a model that will go into the database. In a new file `models.py`,

```python
# app/models.py

from sqlmodel import SQLModel, Field
from typing import Optional


class BookBase(SQLModel):
    title: str
    author: Optional[str]


class Book(BookBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)

```

In `sqlmodel`, models can be treated as both `Pydantic` models and `SQLAlchemy` models. That's all it takes to create the table called `Book` in the database. Now let's actually create the table, we add some code to `main.py`

```python {3, 10-12, 20, 23-26} showLineNumbers
# app/main.py

from fastapi import FastAPI, Depends
import uvicorn

from starlette.staticfiles import StaticFiles
from starlette.templating import Jinja2Templates
from starlette.requests import Request

from sqlmodel import Session, select
from .database import create_db_and_tables, get_session
from .models import Book

templates = Jinja2Templates('app/templates')

app = FastAPI()


def configure():
    create_db_and_tables()
    app.mount('/static', StaticFiles(directory='app/static'), name='static')

@app.get("/")
def home_page(*, session: Session = Depends(get_session), request: Request):
    books = session.exec(select(Book)).all()
    return templates.TemplateResponse('index.html', {'request': request})

if __name__=="__main__":
    configure()
    uvicorn.run('app.main:app', reload=True)
else:
    configure()
```

Finally add the following bit of code to the `index.html` file (see the [Jinja documentation](https://jinja.palletsprojects.com/en/3.0.x/templates/) for more information),

```html {23-29} showLineNumbers
<!DOCTYPE html>

<html lang="en-us">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="A website showcasing fastapi and sqlmodel">
    <meta name="author" content="idjotherwise">

    <title>Fastsqlmodel showcase</title>

    <link href="//oss.maxcdn.com/libs/twitter-bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet">
</head>

<body style="text-align: center">
    <h1>
        A super stylish home page
    </h1>
    <p>
        Really interesting content, full of buzzwords like AI and MACHINE LEARNING!
    </p>
    {% if books %}
    <ul>
        {% for book in books %}
        <li key="{{book.id}}">Title: {{book.title}}, Author: {{book.author}}</li>
        {% endfor %}
    </ul>
    {% endif %}
</body>
</html>
```

At this point, if you navigate `localhost:8000`, you won't see anything has changed - that's because our database is empty. But if the database _did_ have books in it, then in the `get` request the `books = session.exec(select(Book)).all()` would contain a list of `book` items, and so our Jinja template `index.html` would get passed a non-empty list.

There are many ways to add data into our database. You could use [SQLite Browser](https://sqlitebrowser.org/) to open the `database.db` file, PyCharm's own database reader, or any other database browser. Instead, we will add a `POST` route so that we can add in data from the browser.

Add the following code the `main.py`, just below the `@app.get('/')` route.

```python
@app.post("/book")
def add_book(*, session: Session = Depends(get_session), request: Request, book: BookCreate) -> BookRead:
    db_book = Book.from_orm(book)
    session.add(db_book)
    session.commit()
    session.refresh(db_book)
    return db_book
```
We've added in two new models here, `BookCreate` and `BookRead`. Add them to the `models.py` file.

```python {15-16, 19-20} showLineNumbers
# app/models.py
from sqlmodel import SQLModel, Field
from typing import Optional


class BookBase(SQLModel):
    title: str
    author: Optional[str]


class Book(BookBase, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)


class BookCreate(BookBase):
    pass


class BookRead(BookBase):
    id: int
```

Now, navigate to `localhost:8000/docs` in your browser, and you should see the `POST` method right there:

![You should see this in the docs](/images/fastbooks_openapi_screenshot.png)

Click on the post request and add a few books in. These will now be saved into the database.
 Thanks to the way we wrote the `index.html` file, if you navigate again to the home page (`localost:8000`), you should see the books that you just added.
Since we didn't implement `@app.delete` or `@app.patch` methods, we can't delete or update the entries. To start afresh, you can just delete the `database.db` file and re-add the books.

## Adding a `description` column with Alembic

Now let's say that we want to add a `description` field to the book items. We could just delete the `database.db` file, make the changes to the `Book` model, and restart the app. However, if we were in the situation where the database had a bunch of entries that you don't want to lose, then using migrations is the way to go. We'll be using `alembic`, see [the documentation](https://alembic.sqlalchemy.org/en/latest/) for more information. We need to do a few changes to the setup since we are using `sqlmodel` and not `sqlalchemy`. [TestDriven.io](https://testdriven.io/blog/fastapi-sqlmodel/#alembic) has a nice tutorial showing how to do this, so we'll follow that here (although we won't be using the `async` version)
Start by going to the terminal, from root directory of the project, run

```bash
alembic init alembic
```

This will make a directory called `alembic` with some files that we need to modify. The project structure should now look like this

```bash
.
|-- alembic
|   |-- README
|   |-- env.py
|   |-- script.py.mako
|   `-- versions
|-- alembic.ini
|-- app
|   |-- __init__.py
|   |-- database.py
|   |-- main.py
|   |-- models.py
|   |-- settings.py
|   `-- static/
|   `-- templates
|       `-- index.html
|-- database.db
`-- requirements.txt
```

In the `script.py.mako` file, we need to `import sqlmodel`:

```python {6}
# alembic/script.py.mako
# code above ommited

from alembic import op
import sqlalchemy as sa
import sqlmodel
${imports if imports else ""}

# code below ommited
```

and finally in the `alembic/env.py` add the following imports,

```python {8-10, 16, 26} showLineNumbers
from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

from sqlmodel import SQLModel
from app.settings import settings
from app.models import Book

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

config.set_main_option("sqlalchemy.url", settings.database_url)

# Interpret the config file for Python logging.
# This line sets up loggers basically.
fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support

# target_metadata = mymodel.Base.metadata
target_metadata = SQLModel.metadata

# ---- code below omitted ---
```

All we've done here is import the `SQLModel` from `sqlmodel`, which holds all the metadata about our database (after also importing our `Book` model), then we've grabbed the `database_url` from the settings file.

Now delete the `database.db` file, and then generate the first alembic migration with `alembic revision -autogenerate -m "Add book table"` to verify that everything sets up properly. If successful, you will find a new file in the `alembic/revisions` directory which looks something like this (the revision ID might be different):

```python showLineNumbers
"""Add book table

Revision ID: 7071875a0907
Revises:
Create Date: 2021-09-12 22:16:26.595677

"""
from alembic import op
import sqlalchemy as sa
import sqlmodel


# revision identifiers, used by Alembic.
revision = '7071875a0907'
down_revision = None
branch_labels = None
depends_on = None


def upgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.create_table('book',
    sa.Column('title', sqlmodel.sql.sqltypes.AutoString(), nullable=False),
    sa.Column('author', sqlmodel.sql.sqltypes.AutoString(), nullable=True),
    sa.Column('id', sa.Integer(), nullable=True),
    sa.PrimaryKeyConstraint('id')
    )
    op.create_index(op.f('ix_book_author'), 'book', ['author'], unique=False)
    op.create_index(op.f('ix_book_id'), 'book', ['id'], unique=False)
    op.create_index(op.f('ix_book_title'), 'book', ['title'], unique=False)
    # ### end Alembic commands ###


def downgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_index(op.f('ix_book_title'), table_name='book')
    op.drop_index(op.f('ix_book_id'), table_name='book')
    op.drop_index(op.f('ix_book_author'), table_name='book')
    op.drop_table('book')
    # ### end Alembic commands ###

```

Apply the migration (which just creates the table in the database) with `alembic upgrade head`.

Now to add a new column to the `book` table in the database, add the following to the `Book` model in `app/models.py`,

```python {5} showLineNumbers
...
class BookBase(SQLModel):
    title: str
    author: Optional[str]
    description: Optional[str]
...
```

and then generate another migration file with

```bash
alembic revision --autogenerate -m "Add description column"
```
This gives us the following migration file,

```python
"""Add description column

Revision ID: 0b26b04f3af4
Revises: 7071875a0907
Create Date: 2021-09-12 22:55:29.481379

"""
from alembic import op
import sqlalchemy as sa
import sqlmodel

# revision identifiers, used by Alembic.
revision = '0b26b04f3af4'
down_revision = '7071875a0907'
branch_labels = None
depends_on = None


def upgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.add_column('book', sa.Column('description', sqlmodel.sql.sqltypes.AutoString(), nullable=True))
    op.create_index(op.f('ix_book_description'), 'book', ['description'], unique=False)
    # ### end Alembic commands ###


def downgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_index(op.f('ix_book_description'), table_name='book')
    op.drop_column('book', 'description')
    # ### end Alembic commands ###

```

Apply the migration with `alembic upgrade head`.

## Update Jinja template


Now we are going to modify the jinja template in `index.html` to use the new `description` column. While we're at it, lets add some more styling to the page with a header and a footer. For this, we'll create a new file `templates/layout.html`. In there we will put all the things from `index.html` that should go on every page (if there were more pages).

```html {14, 18-22, 28-38, 41-53} showLineNumbers
<!DOCTYPE html>

<html lang="en-us">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="A website showcasing fastapi and sqlmodel">
    <meta name="author" content="idjotherwise">

    <title>Fastsqlmodel showcase</title>

    <link href="//oss.maxcdn.com/libs/twitter-bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet">
    <link href="/static/css/styling.css" rel="stylesheet">
</head>

<body >
<div class="content-wrapper">
    <div class="container">
        <div class="row">
                    <h1>
                          Book list
                    </h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-10">
            <div>
                {% block content %}
                    <div class="content">
                                THIS PAGE HAS NO CONTENT
                    </div>
                {% endblock %}
            </div>
        </div>
    </div>
    <div class="footer">
        <div class="footer links">
            <ul>
                <li><i class="glyphicon glyphicon-cog icon-muted"></i><a
                                href="https://github.com/idjotherwise"
                                target="_blank">Github</a>
                </li>
                <li><i class="glyphicon glyphicon-globe icon-muted"></i><a href="https://twitter.com/johnstonifan"
                                                                                   target="_blank">Twitter</a>
                </li>
            </ul>
        </div>
    </div>
</div>


</body>
</html>
```

Now our `app/templates/index.html` will extend the `app/templates/layout.html` file:

```html {15} showLineNumbers
{% extends "layout.html" %}
{% block content %}

    <h1>
        A super stylish home page
    </h1>
    <p>
        Really interesting content, full of buzzwords like AI and MACHINE LEARNING!
    </p>
    {% if books %}
    <ul style="text-align: left">
        {% for book in books %}
        <li key="{{book.id}}"><b>Title</b>: {{book.title}}<br/>
        <b>Author</b>: {{book.author}}</li>
        <p style="padding-left: 6rem;">{{book.description}}</p>
        {% endfor %}
    </ul>
    {% endif %}

{% endblock %}
```

Finally, we'll add some simple styling in the file `app/static/css/styling.css`. You can also just put in your own style file here if you like.

```css
@import url(//fonts.googleapis.com/css?family=Open+Sans:300,400,600,700);

body {
    font-family: "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
    font-weight: 300;
    color: black;
    background: lightcyan;
    width: 100%;
}

h1,
h2,
h3,
h4,
h5,
h6 {
    font-family: "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
    font-weight: 400;
}

p {
    font-weight: 300;
}

.content-wrapper {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    margin: 2rem auto 0 auto;
    height: 85vh;
    text-align: center;
    width: 56rem;
    justify-content: center;
}
```

Opening up the browser again to `localhost:8000` should greet you with something like

![This fancy looking webpage](/images/fastbooks_jinjad.png)

As you can see, I used the `locahost:8000/docs` page to add a _very interesting_ sounding book to the database and it showed up on the home page. Try adding a bunch more books to see the page filling up. Also go ahead and modify the styling to make it look _actually_ stylish!

## Conclusion
We've done quite a bit here! In another post, we'll clean things
up a bit by adding tests with `pytest` and then deploy to Heroku. We'll also refactor some of the endpoints (and add some more) to use
the powerful [response model](https://sqlmodel.tiangolo.com/tutorial/fastapi/response-model/) stuff from fastapi/sqlmodel. 


Thanks for reading!
