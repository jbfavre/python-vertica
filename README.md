# vertica-python

[![PyPI version](https://badge.fury.io/py/vertica-python.svg)](https://badge.fury.io/py/vertica-python)
[![License](https://img.shields.io/badge/License-Apache%202.0-orange.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python Version](https://img.shields.io/pypi/pyversions/vertica-python.svg)](https://www.python.org/downloads/)
[![Downloads](https://pepy.tech/badge/vertica-python/week)](https://pepy.tech/project/vertica-python)

vertica-python is a native Python adapter for the Vertica (http://www.vertica.com) database.

:loudspeaker: 08/14/2018: *vertica-python* becomes Vertica’s first officially supported open source database client, see the blog [here](https://my.vertica.com/blog/vertica-python-becomes-verticas-first-officially-supported-open-source-database-client/).

Please check out [release notes](https://github.com/vertica/vertica-python/releases) to learn about the latest improvements.

vertica-python is currently in beta stage; it has been tested for functionality and has a very basic test suite. Please use with caution, and feel free to submit issues and/or pull requests (Read up on our [contributing guidelines](#contributing-guidelines)).

vertica-python has been tested with Vertica 9.1.1 and Python 2.7/3.4/3.5/3.6/3.7/3.8.


## Installation

To install vertica-python with pip:

    pip install vertica-python

To install vertica-python from source, run the following command from the root directory:

    python setup.py install

Source code for vertica-python can be found at:

    https://github.com/vertica/vertica-python

#### Using Kerberos authentication
vertica-python has optional Kerberos authentication support for Unix-like systems, which requires you to install the [kerberos](https://pypi.org/project/kerberos/) package:

    pip install kerberos

Note that `kerberos` is a python extension module, which means you need to install `python-dev`. The command depends on the package manager and will look like

    sudo [yum|apt-get|etc] install python-dev

## Usage


**Create connection**

```python
import vertica_python

conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             # autogenerated session label by default,
             'session_label': 'some_label',
             # default throw error on invalid UTF-8 results
             'unicode_error': 'strict',
             # SSL is disabled by default
             'ssl': False,
             # using server-side prepared statements is disabled by default
             'use_prepared_statements': False,
             # connection timeout is not enabled by default
             # 5 seconds timeout for a socket operation (Establishing a TCP connection or read/write operation)
             'connection_timeout': 5}

# simple connection, with manual close
connection = vertica_python.connect(**conn_info)
# do things
connection.close()

# using with for auto connection closing after usage
with vertica_python.connect(**conn_info) as connection:
    # do things
```

You can pass an `ssl.SSLContext` to `ssl` to customize the SSL connection options. For example,

```python
import vertica_python
import ssl

ssl_context = ssl.SSLContext(ssl.PROTOCOL_SSLv23)
ssl_context.verify_mode = ssl.CERT_REQUIRED
ssl_context.check_hostname = True
ssl_context.load_verify_locations(cafile='/path/to/ca_file.pem')

conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'ssl': ssl_context}
connection = vertica_python.connect(**conn_info)

```

See more on SSL options [here](https://docs.python.org/3.8/library/ssl.html).

In order to use Kerberos authentication, install [dependencies](#using-kerberos-authentication) first, and it is the user's responsibility to ensure that an Ticket-Granting Ticket (TGT) is available and valid. Whether a TGT is available can be easily determined by running the `klist` command. If no TGT is available, then it first must be obtained by running the `kinit` command or by logging in. You can pass in optional arguments to customize the authentication. The arguments are `kerberos_service_name`, which defaults to "vertica", and `kerberos_host_name`, which defaults to the value of argument `host`. For example,

```python
import vertica_python

conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             # The service name portion of the Vertica Kerberos principal
             'kerberos_service_name': 'vertica_krb',
             # The instance or host name portion of the Vertica Kerberos principal
             'kerberos_host_name': 'vcluster.example.com'}

with vertica_python.connect(**conn_info) as conn:
    # do things
```

Logging is disabled by default if you do not pass values to both ```log_level``` and ```log_path```.  The default value of ```log_level``` is logging.WARNING. You can find all levels [here](https://docs.python.org/3.8/library/logging.html#logging-levels). The default value of ```log_path``` is 'vertica_python.log', the log file will be in the current execution directory. If ```log_path``` is set to ```''``` (empty string) or ```None```, no file handler is set, logs will be processed by root handlers. For example,

```python
import vertica_python
import logging

## Example 1: write DEBUG level logs to './vertica_python.log'
conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'log_level': logging.DEBUG}
with vertica_python.connect(**conn_info) as connection:
    # do things

## Example 2: write WARNING level logs to './path/to/logs/client.log'
conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'log_path': 'path/to/logs/client.log'}
with vertica_python.connect(**conn_info) as connection:
   # do things

## Example 3: write INFO level logs to '/home/admin/logs/vClient.log'
conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'log_level': logging.INFO,
             'log_path': '/home/admin/logs/vClient.log'}
with vertica_python.connect(**conn_info) as connection:
   # do things

## Example 4: use root handlers to process logs by setting 'log_path' to '' (empty string) 
conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'log_level': logging.DEBUG,
             'log_path': ''}
with vertica_python.connect(**conn_info) as connection:
    # do things
```

Connection Failover: Supply a list of backup hosts to ```backup_server_node``` for the client to try if the primary host you specify in the connection parameters (```host```, ```port```) is unreachable. Each item in the list should be either a host string (using default port 5433) or a (host, port) tuple. A host can be a host name or an IP address.

```python
import vertica_python

conn_info = {'host': 'unreachable.server.com',
             'port': 888,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'backup_server_node': ['123.456.789.123', 'invalid.com', ('10.20.82.77', 6000)]}
connection = vertica_python.connect(**conn_info)
```

Connection Load Balancing helps automatically spread the overhead caused by client connections across the cluster by having hosts redirect client connections to other hosts. Both the server and the client need to enable load balancing for it to function. If the server disables connection load balancing, the load balancing request from client will be ignored.

```python
import vertica_python

conn_info = {'host': '127.0.0.1',
             'port': 5433,
             'user': 'some_user',
             'password': 'some_password',
             'database': 'vdb',
             'connection_load_balance': True}

# Server enables load balancing
with vertica_python.connect(**conn_info) as conn:
    cur = conn.cursor()
    cur.execute("SELECT NODE_NAME FROM V_MONITOR.CURRENT_SESSION")
    print("Client connects to primary node:", cur.fetchone()[0])
    cur.execute("SELECT SET_LOAD_BALANCE_POLICY('ROUNDROBIN')")

with vertica_python.connect(**conn_info) as conn:
    cur = conn.cursor()
    cur.execute("SELECT NODE_NAME FROM V_MONITOR.CURRENT_SESSION")
    print("Client redirects to node:", cur.fetchone()[0])

## Output
#  Client connects to primary node: v_vdb_node0003
#  Client redirects to node: v_vdb_node0005
```

Another way to set connection properties is passing a connection string to the keyword parameter `dsn` of `vertica_python.connect(dsn='...', **kwargs)`. The connection string is of the form:
```
vertica://(user):(password)@(host):(port)/(database)?(arg1=val1&arg2=val2&...)
```
The connection string would be parsed by `vertica_python.parse_dsn(connection_str)`, and the parsing result (a dictionary of keywords and values) would be merged with _kwargs_. If the same keyword is specified in both the sources, the _kwargs_ value overrides the parsed _dsn_ value. The `(arg1=val1&arg2=val2&...)` section can handle string/numeric/boolean values, blank and invalid value would be ignored.

```python
import vertica_python

connection_str = ('vertica://admin@localhost:5433/db1?connection_load_balance=True&connection_timeout=1.5&'
                  'session_label=vpclient+123%7E456')
print(vertica_python.parse_dsn(connection_str))
# {'user': 'admin', 'host': 'localhost', 'port': 5433, 'database': 'db1',
#  'connection_load_balance': True, 'connection_timeout': 1.5, 'session_label': 'vpclient 123~456'}

additional_info = {
    'password': 'some_password', 
    'backup_server_node': ['10.6.7.123', ('10.20.82.77', 6000)]  # invalid value to be set in a connection string
    }

with vertica_python.connect(dsn=connection_str, **additional_info) as conn:
   # do things
```


**Stream query results**:

```python
cur = connection.cursor()
cur.execute("SELECT * FROM a_table LIMIT 2")

for row in cur.iterate():
    print(row)
# [ 1, 'some text', datetime.datetime(2014, 5, 18, 6, 47, 1, 928014) ]
# [ 2, 'something else', None ]

```
Streaming is recommended if you want to further process each row, save the results in a non-list/dict format (e.g. Pandas DataFrame), or save the results in a file.


**In-memory results as list**:

```python
cur = connection.cursor()
cur.execute("SELECT * FROM a_table LIMIT 2")
cur.fetchall()
# [ [1, 'something'], [2, 'something_else'] ]
```


**In-memory results as dictionary**:

```python
cur = connection.cursor('dict')
cur.execute("SELECT * FROM a_table LIMIT 2")
cur.fetchall()
# [ {'id': 1, 'value': 'something'}, {'id': 2, 'value': 'something_else'} ]
connection.close()
```


**Query using named parameters**:

```python
cur = connection.cursor()
cur.execute("SELECT * FROM a_table WHERE a = :propA b = :propB", {'propA': 1, 'propB': 'stringValue'})

cur.fetchall()
# [ [1, 'something'], [2, 'something_else'] ]
```

**Query using server-side prepared statements**:

Vertica server-side prepared statements let you define a statement once and then run it many times with different parameters. Placeholders in the statement are represented by question marks (?). Server-side prepared statements are useful for preventing SQL injection attacks.

```python
import vertica_python

# Enable using server-side prepared statements at connection level
conn_info = {'host': '127.0.0.1',
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'use_prepared_statements': True,
             }

with vertica_python.connect(**conn_info) as connection:
    cur = connection.cursor()
    cur.execute("CREATE TABLE tbl (a INT, b VARCHAR)")
    cur.execute("INSERT INTO tbl VALUES (?, ?)", [1, 'aa'])
    cur.execute("INSERT INTO tbl VALUES (?, ?)", [2, 'bb'])
    cur.executemany("INSERT INTO tbl VALUES (?, ?)", [(3, 'foo'), (4, 'xx'), (5, 'bar')])
    cur.execute("COMMIT")

    cur.execute("SELECT * FROM tbl WHERE a>=? AND a<=? ORDER BY a", (2,4))
    cur.fetchall()
    # [[2, 'bb'], [3, 'foo'], [4, 'xx']]
```

Vertica does not support executing a command string containing multiple statements using server-side prepared statements. You can set ```use_prepared_statements``` option in ```cursor.execute*()``` functions to override the connection level setting.

```python
import vertica_python

# Enable using server-side prepared statements at connection level
conn_info = {'host': '127.0.0.1',
             'user': 'some_user',
             'password': 'some_password',
             'database': 'a_database',
             'use_prepared_statements': True,
             }

with vertica_python.connect(**conn_info) as connection:
    cur = connection.cursor()
    cur.execute("CREATE TABLE tbl (a INT, b VARCHAR)")

    # Executing compound statements
    cur.execute("INSERT INTO tbl VALUES (?, ?); COMMIT", [1, 'aa'])
    # Error message: Cannot insert multiple commands into a prepared statement

    # Disable prepared statements but forget to change placeholders (?)
    cur.execute("INSERT INTO tbl VALUES (?, ?); COMMIT;", [1, 'aa'], use_prepared_statements=False)
    # TypeError: not all arguments converted during string formatting

    cur.execute("INSERT INTO tbl VALUES (%s, %s); COMMIT;", [1, 'aa'], use_prepared_statements=False)
    cur.execute("INSERT INTO tbl VALUES (:a, :b); COMMIT;", {'a': 2, 'b': 'bb'}, use_prepared_statements=False)

# Disable using server-side prepared statements at connection level
conn_info['use_prepared_statements'] = False
with vertica_python.connect(**conn_info) as connection:
    cur = connection.cursor()

    # Try using prepared statements
    cur.execute("INSERT INTO tbl VALUES (?, ?)", [3, 'foo'])
    # TypeError: not all arguments converted during string formatting

    cur.execute("INSERT INTO tbl VALUES (?, ?)", [3, 'foo'], use_prepared_statements=True)

    # Query using named parameters
    cur.execute("SELECT * FROM tbl WHERE a>=:n1 AND a<=:n2 ORDER BY a", {'n1': 2, 'n2': 4})
    cur.fetchall()
    # [[2, 'bb'], [3, 'foo']]
```
Note: In other drivers, the batch insert is converted into a COPY statement by using prepared statements. vertica-python currently does not support that.

**Insert and commits** :

```python
cur = connection.cursor()

# inline commit (when 'use_prepared_statements' is False)
cur.execute("INSERT INTO a_table (a, b) VALUES (1, 'aa'); commit;")

# commit in execution
cur.execute("INSERT INTO a_table (a, b) VALUES (1, 'aa')")
cur.execute("INSERT INTO a_table (a, b) VALUES (2, 'bb')")
cur.execute("commit;")

# connection.commit()
cur.execute("INSERT INTO a_table (a, b) VALUES (1, 'aa')")
connection.commit()
```


**Copy** :

```python
cur = connection.cursor()
cur.copy("COPY test_copy (id, name) from stdin DELIMITER ',' ",  csv)
```

Where `csv` is either a string or a file-like object (specifically, any object with a `read()` method). If using a file, the data is streamed (in chunks of `buffer_size` bytes, which defaults to 128 * 2 ** 10).

```python
with open("/tmp/binary_file.csv", "rb") as fs:
    cursor.copy("COPY table(field1, field2) FROM STDIN DELIMITER ',' ENCLOSED BY '\"'",
                fs, buffer_size=65536)
```


**Cancel the current database operation** :

`Connection.cancel()` interrupts the processing of the current operation. Interrupting query execution will cause the cancelled method to raise a `vertica_python.errors.QueryCanceled`. If no query is being executed, it does nothing. You can call this function from a different thread/process than the one currently executing a database operation.

```python
from multiprocessing import Process
import time
import vertica_python

def cancel_query(connection, timeout=5):
    time.sleep(timeout)
    connection.cancel()

with vertica_python.connect(**conn_info) as conn:
    cur = conn.cursor()

    # Call cancel() from a different process
    p1 = Process(target=cancel_query, args=(conn,))
    p1.start()

    try:
        cur.execute("<Long running query>")
    except vertica_python.errors.QueryCanceled as e:
        pass

    p1.join()
```


## Rowcount oddities

vertica_python behaves a bit differently than dbapi when returning rowcounts.

After a select execution, the rowcount will be -1, indicating that the row count is unknown. The rowcount value will be updated as data is streamed.

```python
cur.execute('SELECT 10 things')

cur.rowcount == -1  # indicates unknown rowcount

cur.fetchone()
cur.rowcount == 1
cur.fetchone()
cur.rowcount == 2
cur.fetchall()
cur.rowcount == 10
```

After an insert/update/delete, the rowcount will be returned as a single element row:

```python
cur.execute("DELETE 3 things")

cur.rowcount == -1  # indicates unknown rowcount
cur.fetchone()[0] == 3
```

## Nextset

If you execute multiple statements in a single call to execute(), you can use cursor.nextset() to retrieve all of the data.

```python
cur.execute('SELECT 1; SELECT 2;')

cur.fetchone()
# [1]
cur.fetchone()
# None

cur.nextset()
# True

cur.fetchone()
# [2]
cur.fetchone()
# None

cur.nextset()
# None
```

## UTF-8 encoding issues

While Vertica expects varchars stored to be UTF-8 encoded, sometimes invalid strings get into the database. You can specify how to handle reading these characters using the unicode_error connection option. This uses the same values as the unicode type (https://docs.python.org/2/library/functions.html#unicode)

```python
cur = vertica_python.Connection({..., 'unicode_error': 'strict'}).cursor()
cur.execute(r"SELECT E'\xC2'")
cur.fetchone()
# caught 'utf8' codec can't decode byte 0xc2 in position 0: unexpected end of data

cur = vertica_python.Connection({..., 'unicode_error': 'replace'}).cursor()
cur.execute(r"SELECT E'\xC2'")
cur.fetchone()
# �

cur = vertica_python.Connection({..., 'unicode_error': 'ignore'}).cursor()
cur.execute(r"SELECT E'\xC2'")
cur.fetchone()
# 
```

## License

Apache 2.0 License, please see `LICENSE` for details.

## Contributing guidelines

Have a bug or an idea? Please see `CONTRIBUTING.md` for details.

## Acknowledgements

We would like to thank the contributors to the Ruby Vertica gem (https://github.com/sprsquish/vertica), as this project gave us inspiration and help in understanding Vertica's wire protocol. These contributors are:

 * [Matt Bauer](http://github.com/mattbauer)
 * [Jeff Smick](http://github.com/sprsquish)
 * [Willem van Bergen](http://github.com/wvanbergen)
 * [Camilo Lopez](http://github.com/camilo)
