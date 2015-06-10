aiopg-trollius
=======
.. image:: https://travis-ci.org/aio-libs/aiopg.svg?branch=master
    :target: https://travis-ci.org/aio-libs/aiopg
.. image:: https://coveralls.io/repos/aio-libs/aiopg/badge.svg
    :target: https://coveralls.io/r/aio-libs/aiopg

**aiopg** is a library for accessing a PostgreSQL_ database
from the asyncio_ (PEP-3156/tulip) framework. It wraps
asynchronous features of the Psycopg database driver. This
project, **aiopg-trollius** is a port to use trollius
(basically an asyncio-like library for python 2.x +).

Example
-------

::

    from trollius import From, Return
    import trollius as asyncio
    from aiopg_trollius.pool import create_pool
    from contextlib import closing, contextmanager

    dsn = 'dbname=jetty user=nick password=1234 host=localhost port=5432'


    @asyncio.coroutine
    def yield_conn_context(pool):
        """
        Helper to manage the connection context.
        """
        conn = yield From(pool.acquire())

        @contextmanager
        def context():

            try:
                yield conn
            finally:
                pool.release(conn)
        raise Return( context() )


    @asyncio.coroutine
    def yield_cur_context(conn):
        """
        Helper to manage the cursor context.
        """
        cur = yield From(conn.cursor())

        raise Return( closing(cur) )



    @asyncio.coroutine
    def test_select():
        pool = yield From(create_pool(dsn))

        with (yield From(yield_conn_context(pool)) as conn:
            with (yield From(yield_cur_context(conn))) as cur:
                cur = yield From(conn.cursor())
                yield From(cur.execute('SELECT 1'))
                ret = yield From(cur.fetchone())
                assert ret == (1,), ret


    asyncio.get_event_loop().run_until_complete(test_select())


Example of SQLAlchemy optional integration
-------------------------------------------

**Note: SQLAlchemy integration is NOT ported**

::

   import asyncio
   from aiopg.sa import create_engine
   import sqlalchemy as sa


   metadata = sa.MetaData()

   tbl = sa.Table('tbl', metadata,
       sa.Column('id', sa.Integer, primary_key=True),
       sa.Column('val', sa.String(255)))


   @asyncio.coroutine
   def go():
       engine = yield from create_engine(user='aiopg',
                                         database='aiopg',
                                         host='127.0.0.1',
                                         password='passwd')

       with (yield from engine) as conn:
           yield from conn.execute(tbl.insert().values(val='abc'))

           res = yield from conn.execute(tbl.select())
           for row in res:
               print(row.id, row.val)


   asyncio.get_event_loop().run_until_complete(go())

.. _PostgreSQL: http://www.postgresql.org/
.. _asyncio: http://docs.python.org/3.4/library/asyncio.html

**Note: Tests were not ported**

Please use::

   $ python3 runtests.py

for executing project's unittests
