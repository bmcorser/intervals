intervals
=========

|Build Status| |Version Status| |Downloads|

Python tools for handling intervals (ranges of comparable objects).


Interval initialization
-----------------------


.. code-block:: python

    >>> from intervals import IntInterval

    >>> # All integers between 1 and 4
    >>> interval = IntInterval([1, 4])
    >>> interval.lower
    1
    >>> interval.upper
    4
    >>> interval.lower_inc
    True
    >>> interval.upper_inc
    True


Open, half-open and closed intervals
------------------------------------

Intervals can be either open, half-open or closed. Properties ``lower_inc`` and
``upper_inc`` denote whether or not given endpoint is included (open) or not.

* An open interval is an interval where both endpoints are open.

  .. code-block:: python

      >>> interval = IntInterval((1, 4))
      >>> interval.open
      True
      >>> interval.lower_inc
      False
      >>> interval.upper_inc
      False

* Half-open interval has one of the endpoints as open

  .. code-block:: python

      >>> interval = Interval('[1, 4)')
      >>> interval.open
      False
      >>> interval.lower_inc
      True
      >>> interval.upper_inc
      False

* Closed interval includes both endpoints

  .. code-block:: python

      >>> interval = Interval([1, 4])
      >>> interval.closed
      True
      >>> interval.lower_inc
      True
      >>> interval.upper_inc
      True


Interval types
--------------

Each interval encapsulates a type. Interval is not actually a class. Its a
convenient factory that generates ``AbstractInterval`` subclasses. Whenever you
call ``Interval()`` the ``IntervalFactory`` tries to guess to best matching
interval for given bounds.

.. code-block:: python

    >>> from datetime import date
    >>> from infinity import inf

    >>> interval = Interval([1, 4])
    >>> interval
    IntInterval('[1, 4]')
    >>> interval.type.__name__
    'int'

    >>> interval = Interval(['a', 'd'])
    >>> interval
    CharacterInterval('[a, d]')
    >>> interval.type.__name__
    'str'

    >>> interval = Interval([1.5, 4])
    >>> interval
    FloatInterval('[1.5, 4.0]')
    >>> interval.type
    <type 'float'>

    >>> interval = Interval([date(2000, 1, 1), inf])
    >>> interval
    DateInterval('[2000-01-01,]')
    >>> interval.type.__name__
    'date'


You can also create interval subtypes directly (this is also faster than using
``Interval``).

.. code-block:: python

    >>> from intervals import FloatInterval, IntInterval
    >>> IntInterval([1, 4])
    IntInterval('[1, 4]')
    >>> FloatInterval((1.4, 2.7))
    FloatInterval('(1.4, 2.7)')

Currently provided subtypes are:

* ``IntInterval``
* ``CharacterInterval``
* ``FloatInterval``
* ``DecimalInterval``
* ``DateInterval``
* ``DateTimeInterval``


Properties
----------

* ``radius`` gives the half-length of an interval

  .. code-block:: python

      >>> IntInterval([1, 4]).radius
      1.5

* ``length`` gives the length of an interval.

  .. code-block:: python

      >>> IntInterval([1, 4]).length
      3

* ``centre`` gives the centre (midpoint) of an interval

  .. code-block:: python

      >>> IntInterval([-1, 1]).centre
      0.0

* Interval :math:`[a, b]` is ``degenerate`` if :math:`a = b`

  .. code-block:: python

      >>> IntInterval([1, 1]).degenerate
      True
      >>> IntInterval([1, 2]).degenerate
      False


Emptiness
---------

An interval is empty if it contains no points:

.. code-block:: python

    >>> IntInterval('(1, 1]').empty
    True


Data type coercion
------------------

Interval evaluates as ``True`` if its non-empty

.. code-block:: python

    >>> bool(IntInterval([1, 6]))
    True
    >>> bool(IntInterval([0, 0]))
    True
    >>> bool(IntInterval('(1, 1]'))
    False

Integer intervals can be coerced to integer if they contain only one point,
otherwise passing them to ``int()`` throws a ``TypeError``

.. code-block:: python

    >>> int(IntInterval([1, 6]))
    Traceback (most recent call last):
        ...
    TypeError: Only intervals containing single point can be coerced to integers

    >>> int(IntInterval('[1, 1]'))
    1


Operators
---------


Operator coercion rules
^^^^^^^^^^^^^^^^^^^^^^^

All the operators and arithmetic functions use special coercion rules. These
rules are made for convenience.

So for example when you type:

.. code-block:: python

    IntInterval([1, 5]) > IntInterval([3, 3])

Its actually the same as typing:

.. code-block:: python

    IntInterval([1, 5]) > [3, 3]

Which is also the same as typing:

.. code-block:: python

    IntInterval([1, 5]) > 3


Comparison operators
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    >>> IntInterval([1, 5]) > IntInterval([0, 3])
    True
    >>> IntInterval([1, 5]) == IntInterval([1, 5])
    True
    >>> IntInterval([2, 3]) in IntInterval([2, 6])
    True
    >>> IntInterval([2, 3]) in IntInterval([2, 3])
    True
    >>> IntInterval([2, 3]) in IntInterval((2, 3))
    False


Intervals are hashable
^^^^^^^^^^^^^^^^^^^^^^

Intervals are hashed on the same attributes that affect comparison: the values
of the upper and lower bounds, ``lower_inc`` and ``upper_inc``, and the
``type`` of the interval. This enables the use of intervals as keys in dict()
objects.

.. code-block:: python

    >>> IntInterval([3, 7]) in {IntInterval([3, 7]): 'zero to ten'}
    True
    >>> IntInterval([3, 7]) in set([IntInterval([3, 7])])
    True
    >>> IntInterval((3, 7)) in set([IntInterval([3, 7])])
    False
    >>> IntInterval([3, 7]) in set([FloatInterval([3, 7])])
    False


Discrete intervals
------------------

.. code-block:: python

    >>> IntInterval([2, 4]) == IntInterval((1, 5))
    True


Using interval steps
^^^^^^^^^^^^^^^^^^^^

You can assign given interval to use optional ``step`` argument. By default
``IntInterval`` uses ``step=1``. When the interval encounters a value that is
not a multiplier of the ``step`` argument it tries to round it to the nearest
multiplier of the ``step``.

.. code-block:: python

    >>> from intervals import IntInterval

    >>> interval = IntInterval([0, 5], step=2)
    >>> interval.lower
    0
    >>> interval.upper
    6

You can also use steps for ``FloatInterval`` and ``DecimalInterval`` classes.
Same rounding rules apply here.

.. code-block:: python

    >>> from intervals import FloatInterval

    >>> interval = FloatInterval([0.2, 0.8], step=0.5)
    >>> interval.lower
    0.0
    >>> interval.upper
    1.0


Arithmetics
-----------


Arithmetic operators
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    >>> Interval([1, 5]) + Interval([1, 8])
    IntInterval('[2, 13]')

    >>> Interval([1, 4]) - 1
    IntInterval('[0, 3]')

Intersection:

.. code-block:: python

    >>> Interval([2, 6]) & Interval([3, 8])
    IntInterval('[3, 6]')

Union:

.. code-block:: python

    >>> Interval([2, 6]) | Interval([3, 8])
    IntInterval('[2, 8]')


Arithmetic functions
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    >>> interval = IntInterval([1, 3])

    >>> # greatest lower bound
    >>> interval.glb(IntInterval([1, 2]))
    IntInterval('[1, 2]')

    >>> # least upper bound
    >>> interval.lub(IntInterval([1, 2]))
    IntInterval('[1, 3]')

    >>> # infimum
    >>> interval.inf(IntInterval([1, 2]))
    IntInterval('[1, 2]')

    >>> # supremum
    >>> interval.sup(IntInterval([1, 2]))
    IntInterval('[1, 3]')


.. |Build Status| image:: https://travis-ci.org/kvesteri/intervals.png?branch=master
   :target: https://travis-ci.org/kvesteri/intervals
.. |Version Status| image:: https://pypip.in/v/intervals/badge.png
   :target: https://crate.io/packages/intervals/
.. |Downloads| image:: https://pypip.in/d/intervals/badge.png
   :target: https://crate.io/packages/intervals/
