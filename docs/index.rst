.. currentmodule:: chrononaut

Chrononaut
==========

`Chrononaut`_ is a simple package to provide versioning, change tracking, and record
locking for applications using `Flask-SQLAlchemy`_. It currently supports Postgres as
a database backend.

.. _Chrononaut: https://github.com/onecodex/chrononaut
.. _Flask-SQLAlchemy: http://flask-sqlalchemy.pocoo.org/2.1/

Getting started
---------------

Getting started with Chrononaut is a simple two step process. First, replace your :class:`FlaskSQLAlchemy`
database object with a Chrononaut :class:`VersionedSQLAlchemy` database connection::


    from flask_sqlalchemy import SQLAlchemy
    from chrononaut import VersionedSQLAlchemy

    # A standard, FlaskSQLAlchemy database connection without support
    # for automatic version tracking
    db = SQLAlchemy(app)

    # A Chrononaut database connection with automated versioning
    # for any models with a `Versioned` mixin
    db = VersionedSQLAlchemy(app)


After that, simply add the :class:`Versioned` mixin object to your standard Flask-SQLAlchemy models::

    # A simple User model with versioning to support tracking of, e.g.,
    # email and name changes.
    class User(db.Model, Versioned):
        __tablename__ = 'appuser'
        __chrononaut_untracked__ = ['login_count']
        __chrononaut_hidden__ = ['password']

        id = db.Column(db.Integer, primary_key=True)
        name = db.Column(db.String(80), unique=False)
        email = db.Column(db.String(255), unique=True)
        password = db.Column(db.Text())
        ...
        login_count = db.Column(db.Integer())


This creates an ``appuser_history`` table that provides prior record values, along with JSON ``change_info``
and a ``changed`` microsecond-level timestamp.


Using model history
-------------------
Chrononaut automatically generates a history table for each model into which you mixin :class:`Versioned`. This history table facilitates::

    # See if the user has changed their email
    # since they first signed up
    user = User.query.first()
    original_user_info = user.versions()[0]
    if user.email == original_user_info.email:
        print('User email matches!')
    else:
        print('The user has updated their email!')


Trying to access fields that are untracked or hidden raises an exception::

    print(original_user_info.password)     # Raises a HiddenAttributeError
    print(original_user_info.login_count)  # Raises an UntrackedAttributeError

For more information on fetching specific version records see :meth:`Versioned.versions`.

Fine-grained versioning
-----------------------
By default, Chrononaut will automatically version every column in a model.

In the above example, we do not want to retain past user passwords in our history table, so we add ``password`` to the model's ``__chrononaut_hidden__`` property. Changes to a user's password will now result in a new model version and creation of a history record, but the automatically generated ``appuser_history`` table will not have a ``password`` field and will only note that a hidden column was changed in its ``change_info`` JSON column.

Similarly, Chrononaut's ``__chrononaut_untracked__`` property allows us to specify that we do not want to track a field at all. This is useful for changes that are regularly incremented, toggled, or otherwise changed but do not need to be tracked. A good example would be a ``starred`` property on an object or other UI state that might be persisted to the database between application sessions.


Migrations
----------
Chrononaut automatically generates a SQLAlchemy model (and corresponding table) for each :class:`Versioned` mixin. By default, this table is named ``tablename_history`` where ``tablename`` is the name of the table for the model. A custom table name may be specified by using the ``__chrononaut_tablename__`` property in the model.

In order to use Chrononaut, it's important to keep your ``*_history`` tables in sync with your main tables. We recommend using `Alembic`_ for migrations which should automatically generate the ``*_history`` tables when you first add the :class:`Versioned` mixins and subsequent updates to your models.

.. _Alembic: http://alembic.zzzcomputing.com/en/latest/


More details
------------

More in-depth information on Chrononaut's API is available below:

.. toctree::
   :maxdepth: 2

   basics
