====
MAGE
====

Easy commands set creation

Doc
===

``mage`` allows you to create standalone commands and commands digests. After that you may use that commands easy::

    % python manage.py command_name arg --kwarg=val --kwarg2

or if you defined the command digest::

    % python manage.py digest:command_name arg --kwarg=val --kwarg2

Defining commands digest
------------------------

Subclass ``mage.CommandDigest`` class and append methods with prefix ``command_``. For example I want to create useful command digest for sqlalchemy::

    class SqlaCommands(CommandDigest):
        '''
        This commands allow you to sync/drop/init tables in database
        '''

        def __init__(self, models, init_function=None):
            self.models = models
            self.init_function = init_function

        def command_sync(self, db_name='default'):
            # implementation
            pass

        def command_drop(self, db_name='default'):
            # implementation
            pass

        def command_init(self, db_name='default'):
            # construct session object
            if self.init_function:
                self.init_function(session)

        def command_reset(self, db_name='default'):
            self.command_drop(db_name=db_name)
            self.command_sync(db_name=db_name)
            self.command_init(db_name=db_name)

Note: You can provide your own ``__init__``.
Note: Class docstring and methods docstring becomes help message.

After that just create module with any name, which actualy will be mage :)::

    # ./manage.py
    from models import models_list, initial

    if __name__ == '__main__':
        from sys import argv
        from mage import manage
        manage(dict(
            sqla=SqlaCommands(models_list, initial)
        ), argv)

Note: You use another delimeter instead of ':', just provide kw argement to ``manage`` function. ``manage(commands, argv, delim='.')``
Now you are ready to use commands::

    % python manage.py sqla:sync
    % python manage.py sqla:sync admin_base
    % python manage.py sqla:reset front_base

Defining standalone command
---------------------------

If there is no need in command digest you can create standalone command by callable::

    def cmd(arg, kwarg=None, kwarg2=False):
        assert(arg == '1')
        assert(kwarg == 'val')
        assert(kwarg2 == True)

    # ./manage.py
    if __name__ == '__main__':
        from sys import argv
        from mage import manage
        manage(dict(
            cmd=cmd,
        ), argv)

And after that::

    % python manage.py cmd arg --kwarg=val --kwarg2


On command parametrs
--------------------

Main purpose was to create flexible commands easy way. So, for parametrs we use native python function parametrs declaration, where you can have args, keyword args with it's defaults values. On command line all arguments after command name will become args. Arguments in form of '--arg=value' will become kwargs. Arguments in form of '--arg' will become kwarg with value 'True' (very useful sometimes). So this call means::

    % python manage.py digest:command_name arg --kwargs=val --kwargs2

    command_instance.command_command_name('arg', kwarg='val', kwarg2=True)


Arguments converters
--------------------

``mage`` has smart decorator called ``argconv``. It helps to convert arguments to python types. First parametr of ``argconv`` - argument id. For positiional args it is index number, for keyword args it is arg name (str) (note: as you may know - indexing in python starts from zero). All other positional parametrs are - functions that can convert or validate values::

    class TestCommand(CommandDigest):

        @argconv(1, argconv.to_int)
        @argconv('kwarg', argconv.to_date)
        def command_test(self, arg, kwarg=None, kwarg2=False):
            assert(arg == 1)
            assert(kwarg == datetime.date(2010, 6, 9))
            assert(kwarg2 == True)

    % python manage.py cmd:test 1 --kwarg=9/6/2010 --kwarg2


mage script
-----------

If you install ``mage`` standard way (i.e. distutils, setooptools, pip, distribute) you have script installed in your system's bin directory called ``mage``. This script allows to call commands from modules inplace. For example we have package ``insanities`` (in PYTHON_PATH) with module ``cmd`` with ``mage.CommandDigest`` based commands in it and we want to call command ``project``::

    % mage insanities.cmd:project name_of_project

``mage`` script will look for ``project`` command in ``insanities.cmd`` and if it will find it ``project`` will be called with parameters given to ``mage`` script.
