.. image:: https://img.shields.io/pypi/pyversions/PyAthenaJDBC.svg
    :target: https://pypi.python.org/pypi/PyAthenaJDBC/

.. image:: https://circleci.com/gh/laughingman7743/PyAthenaJDBC.svg?style=shield
    :target: https://circleci.com/gh/laughingman7743/PyAthenaJDBC

.. image:: https://codecov.io/gh/laughingman7743/PyAthenaJDBC/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/laughingman7743/PyAthenaJDBC

.. image:: https://img.shields.io/pypi/l/PyAthenaJDBC.svg
    :target: https://github.com/laughingman7743/PyAthenaJDBC/blob/master/LICENSE


PyAthenaJDBC
============

PyAthenaJDBC is a Python `DB API 2.0 (PEP 249)`_ compliant wrapper for `Amazon Athena JDBC driver`_.

.. _`DB API 2.0 (PEP 249)`: https://www.python.org/dev/peps/pep-0249/
.. _`Amazon Athena JDBC driver`: http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html

Requirements
------------

* Python

  - CPython 2.6, 2,7, 3,4, 3.5

* Java

  - Java >= 7

Installation
------------

.. code:: bash

    $ pip install PyAthenaJDBC

Usage
-----

Basic usage
~~~~~~~~~~~

.. code:: python

    from pyathenajdbc import connect

    conn = connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/',
                   region_name='us-west-2')
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT * FROM one_row
            """)
            print(cursor.description)
            print(cursor.fetchall())
    finally:
        conn.close()

Cursor iteration
~~~~~~~~~~~~~~~~

.. code:: python

    from pyathenajdbc import connect

    conn = connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/',
                   region_name='us-west-2')
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT * FROM many_rows LIMIT 10
            """)
            for row in cursor:
                print(row)
    finally:
        conn.close()

Query with parameter
~~~~~~~~~~~~~~~~~~~~

.. code:: python

    from pyathenajdbc import connect

    conn = connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/',
                   region_name='us-west-2')
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT col_int FROM one_row_complex where col_int = {0}
            """, 2147483647)
            print(cursor.fetchall())

            cursor.execute("""
            SELECT col_string FROM one_row_complex where col_string = {param}
            """, param='a string')
            print(cursor.fetchall())
    finally:
        conn.close()

Minimal example for Pandas DataFrame
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    from pyathenajdbc import connect
    import pandas as pd

    conn = connect(access_key='YOUR_ACCESS_KEY_ID',
                   secret_key='YOUR_SECRET_ACCESS_KEY',
                   s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/',
                   region_name='us-west-2',
                   jvm_path='/path/to/jvm')  # optional, as used by JPype
    df = pd.read_sql("SELECT * FROM many_rows LIMIT 10", conn)

As Pandas DataFrame
~~~~~~~~~~~~~~~~~~~

.. code:: python

    import contextlib
    from pyathenajdbc import connect
    from pyathenajdbc.util import as_pandas

    with contextlib.closing(
            connect(s3_staging_dir='s3://YOUR_S3_BUCKET/path/to/'
                    region_name='us-west-2'))) as conn:
        with conn.cursor() as cursor:
            cursor.execute("""
            SELECT * FROM many_rows
            """)
            df = as_pandas(cursor)
    print(df.describe())

Examples
--------

Redash_ query runner example
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See `examples/redash/athena.py`_

.. _Redash: https://github.com/getredash/redash
.. _`examples/redash/athena.py`: examples/redash/athena.py

Credential
----------

Support `AWS CLI credentials configuration`_.

.. _`AWS CLI credentials configuration`: http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html

Credential Files
~~~~~~~~~~~~~~~~

~/.aws/credentials

.. code:: cfg

    [default]
    aws_access_key_id=YOUR_ACCESS_KEY_ID
    aws_secret_access_key=YOUR_SECRET_ACCESS_KEY

~/.aws/config

.. code:: cfg

    [default]
    region=us-west-2
    output=json

Environment variables
~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    $ export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
    $ export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY
    $ export AWS_DEFAULT_REGION=us-west-2

Additional environment variable:

.. code:: bash

    $ export AWS_ATHENA_S3_STAGING_DIR=s3://YOUR_S3_BUCKET/path/to/

Testing
-------

Depends on the AWS CLI credentials and the following environment variables:

~/.aws/credentials

.. code:: cfg

    [default]
    aws_access_key_id=YOUR_ACCESS_KEY_ID
    aws_secret_access_key=YOUR_SECRET_ACCESS_KEY

Environment variables

.. code:: bash

    $ export AWS_DEFAULT_REGION=us-west-2
    $ export AWS_ATHENA_S3_STAGING_DIR=s3://YOUR_S3_BUCKET/path/to/

Run test
~~~~~~~~

.. code:: bash

    $ pip install pytest awscli
    $ scripts/upload_test_data.sh
    $ py.test
    $ scripts/delete_test_data.sh

Run test multiple Python versions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    $ pip install tox awscli
    $ scripts/upload_test_data.sh
    $ pyenv local 2.6.9 2.7.12 3.4.5 3.5.2
    $ tox
    $ scripts/delete_test_data.sh
