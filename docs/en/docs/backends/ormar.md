When generating routes, the `OrmarCRUDRouter` will automatically tie into your database
using your [ormar](https://collerek.github.io/ormar/) models. To use it, simply pass your ormar database model as the schema.

## Simple Example

Below is an example assuming that you have already imported and created all the required
models.

```python
router = OrmarCRUDRouter(
    schema=MyOrmarModel,
    create_schema=Optional[MyPydanticCreateModel],
    update_schema=Optional[MyPydanticUpdateModel]
)

app.include_router(router)
```

!!! note
    The `create_schema` should not include the *primary id* field as this be
    generated by the database.

## Full Example

```python
# example.py
import databases
import ormar
import sqlalchemy
import uvicorn
from fastapi import FastAPI

from fastapi_crudrouter import OrmarCRUDRouter

DATABASE_URL = "sqlite:///./test.db"
database = databases.Database(DATABASE_URL)
metadata = sqlalchemy.MetaData()

app = FastAPI()


@app.on_event("startup")
async def startup():
    await database.connect()


@app.on_event("shutdown")
async def shutdown():
    await database.disconnect()


class BaseMeta(ormar.ModelMeta):
    metadata = metadata
    database = database


def _setup_database():
    # if you do not have the database run this once
    engine = sqlalchemy.create_engine(DATABASE_URL)
    metadata.drop_all(engine)
    metadata.create_all(engine)
    return engine, database


class Potato(ormar.Model):
    class Meta(BaseMeta):
        pass

    id = ormar.Integer(primary_key=True)
    thickness = ormar.Float()
    mass = ormar.Float()
    color = ormar.String(max_length=255)
    type = ormar.String(max_length=255)


app.include_router(
    OrmarCRUDRouter(
        schema=Potato,
        prefix="potato",
    )
)

if __name__ == "__main__":
    uvicorn.run("example:app", host="127.0.0.1", port=5000, log_level="info")
```
