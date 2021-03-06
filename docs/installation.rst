Installation
============

Dependencies
------------

EvaP is written in Python using the Django framework and you need at least
Python 2.6 to run it. Apart from Python and Django there are some other
dependencies that are listed in the file ``requirements.txt``. The file is 
in a format that `pip <http://www.pip-installer.org/en/latest/installing.html>`_
can use to automatically install all the requirements. This can be performed on
the command line with ``pip install -r requirements.txt``.

*For Windows users:* Some dependencies have native code, such as lxml and PIL 
(Python Imaging Library). As compiling these with your installed compiler 
usually fails, you need to install the precompiled packages from
`lfd.uci.edu <http://www.lfd.uci.edu/~gohlke/pythonlibs/>`_.

*For Windows users:* For the translation of texts gettext is used.
This can be downloaded as a precompiled package from
`the GnuWin project <http://sourceforge.net/projects/gnuwin32/files/gettext/>`_.

Filesystem Structure
--------------------

We recommend that you install the application into the directory ``/opt/evap``
according to the filesystem hierarchy standard. In the remaining steps, we will 
assume that this is your installation directory.

Clone the repository or copy the files into that directory. 
The installation should be correct if the settings
file has the path ``/opt/evap/evap/settings.py``. Make sure that all files and
directories are readable by the Apache web server. Additionally please make sure
that the directory ``/opt/evap/evap/upload`` is writable by the web server.

Settings
--------

The configuration of the application is done by modifying the ``settings.py`` 
in the ``evap`` folder. For a production environment you should change the 
following settings:

- Choose an appropriate database and modify the ``default`` entry in the 
  ``DATABASES`` settings. Please make sure that you use a database that 
  supports transactions.
- Change the ``DEFAULT_FROM_EMAIL`` to an administrative mail address that is 
  used as the sender of the mails generated by the system.
- Change ``MEDIA_ROOT`` to a directory that is writable by the web application.
  This directory will hold user-uploaded files like photos.
- You might want to change the ``SECRET_KEY``.
- Modify the ``LOGGING`` configuration so that it suits your needs.
- Finally, set ``DEBUG`` to ``False``.

Database Initialization
-----------------------

To initialize the database during the installation you have to perform the 
following command in ``/opt/evap`` directory:

- ``python manage.py syncdb`` to initialize the database.

File Refresh
------------

You have to run some additional commands during the installation and whenever
you upgrade the software. Perform these steps in the ``/opt/evap``
directory after you have upgraded the files:

- ``python manage.py migrate`` to perform any potential database updates.
- ``python manage.py collectstatic`` to collect all files that the front-end
  webserver should serve.
- ``python manage.py compilemessages`` to update the binary translation catalog.

Finally, restart the Apache web server.

Development Environment: Setup and Server Running
-------------------------------------------------

The manual creation of a superuser is just recommended for development environments.
In productive environments a Kerberos authentification system or similar should
be used.

To create a superuser perform in the ``/opt/evap`` directory the command
``python manage.py createsuperuser``.

A login in a development environment can be simulated by setting the variable 
``REMOTE_USER`` to your previously created user.

The server for this environment can be started from within the ``/opt/evap``
directory with ``python manage.py runserver``.

Productive Environment: Apache 2 Configuration
----------------------------------------------

We recommend the following Apache configuration:

::

        WSGIScriptAlias / /opt/evap/handler.wsgi
        <Location /login>
                AuthName "Kerberos Domain Login"
                AuthType Kerberos
                KrbAuthRealms DOMAIN.EXAMPLE.COM
                KrbMethodNegotiate On
                KrbMethodK5Passwd On
                KrbVerifyKDC off

                Require valid-user
        </Location>

        Alias /static /opt/evap/evap/staticfiles
        <Location /static>
            ExpiresActive On
            ExpiresDefault "access plus 1 month"
        </Location>
        Alias /media /opt/evap/evap/upload

Productive Environment: Cron Configuration
------------------------------------------

EvaP has components which need to react to timed events.
This behavior is implemented by running two cronjobs, which in turn trigger
a management command. One should be run hourly and the other one daily

For example you could use a /etc/cron.hourly/evap like

::

    #!/bin/sh
    
    pushd  /opt/evap
    /usr/bin/python manage.py run_tasks
    popd

And a /etc/cron.daily/evap like

::

    #!/bin/sh
    
    pushd  /opt/evap
    /usr/bin/python manage.py run_tasks daily
    popd
