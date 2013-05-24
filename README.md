newrelic_plugin_agent
=====================

An agent that polls supported backend systems and submits the results to the
NewRelic platform. Currently supported backend systems are:

- Apache HTTP Server
- CouchDB
- Edgecast CDN "Realtime" API
- Memcached
- Nginx
- pgBouncer
- PostgreSQL
- RabbitMQ
- Redis
- Riak

Installation Instructions
-------------------------
1. Unzip the archive
2. Ensure that setuptools is up to date and installed on your system.
3. In the archive directory:

    python setup.py install

4. Copy the configuration file example from /opt/newrelic_plugin_agent/etc/example.yml to /etc/newrelic_plugin_agent.yml and edit the configuration in that file.
5. Run the app:

    newrelic_plugin_agent -c PATH-TO-CONF-FILE [-f]

* - Where -f is to run it in the foreground instead of as a daemon.

Sample configuration and init.d script are installed in /opt/newrelic_plugin_agent

Installing Additional Requirements
----------------------------------

To use the pgBouncer or PostgreSQL plugin you must install the psycopg2 library. To easily do
this, make sure you have the latest version of pip installed (http://www.pip-installer.org/). This should be done after installing the agent itself.

Once installed, from inside the source directory run the following command:

    pip install -e .[pgbouncer]

or

    pip install -e .[postgresql]

Apache HTTPd Installation Nodes
-------------------------------
Enable the HTTPd server status page in the default virtual host. The following example configuration snippet for Apache HTTPd demonstrates how to do this:

    <Location /server-status>
        SetHandler server-status
        Order deny,allow
        Deny from all
        Allow from all
    </Location>

Nginx Installation Notes
------------------------
Enable the nginx stub_status setting on the default site in your configuration. The following example configuration snippet for Nginx demonstates how to do this:

      location /nginx_stub_status {
        stub_status on;
      }

pgBouncer Installation Notes
----------------------------
The user specified must be a stats user.

PostgreSQL Installation Notes
-----------------------------
The user specified must have the ability to query the PostgreSQL system catalog.

RabbitMQ Installation Notes
---------------------------
The user specified must have access to all virtual hosts you wish to monitor and should have either the Administrator tag or the Monitor tag.

Configuration Example
---------------------

    %YAML 1.2
    ---
    Application:
      license_key: VALUE
      poll_interval: 60

      apache_httpd:
        name: hostname
        host: localhost
        port: 80
        path: /server-status

      couchdb:
        name: localhost
        host: localhost
        port: 5984

      edgecast:
        name: My Edgecase Account
        account: ACCOUNT_NUMBER
        token: API_TOKEN

      memcached:
        name: localhost
        host: localhost
        port: 11211

      nginx:
        name: hostname
        host: localhost
        port: 80
        path: /nginx_stub_status

      pgbouncer:
        host: localhost
        port: 6000
        user: stats

      postgresql:
        host: localhost
        port: 5432
        user: postgres
        dbname: postgres

      rabbitmq:
        name: rabbitmq@localhost
        host: localhost
        port: 15672
        username: guest
        password: guest

      redis:
        - name: localhost
          host: localhost
          port: 6379
          db_count: 16

      riak:
        name: localhost
        host: localhost
        port: 8098

    Daemon:
      pidfile: /var/run/newrelic_plugin_agent.pid

    Logging:
      formatters:
        verbose:
          format: '%(levelname) -10s %(asctime)s %(process)-6d %(processName) -15s %(threadName)-10s %(name) -25s %(funcName) -25s L%(lineno)-6d: %(message)s'
      handlers:
        file:
          class : logging.handlers.RotatingFileHandler
          formatter: verbose
          filename: /tmp/newrelic_plugin_agent.log
          maxBytes: 10485760
          backupCount: 3
      loggers:
        newrelic_plugin_agent:
          level: INFO
          propagate: True
          handlers: [console, file]
        requests:
          level: ERROR
          propagate: True
          handlers: [console, file]

Troubleshooting
---------------
If the installation does not install the "newrelic_plugin_agent" application in /usr/bin then it is likely that setuptools or distribute is not up to date. The following commands can be run to install distribute and pip for installing the application:

    curl http://python-distribute.org/distribute_setup.py | python
    curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | python

If the application installs but doesn't seem to be submitting status, check the logfile which at /tmp/newrelic_plugin_agent.log if the default example logging configuration is used.
