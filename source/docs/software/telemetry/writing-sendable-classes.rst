# Writing Your Own Sendable Classes

Since the ``Sendable`` interface only has one method, writing your own classes that implement ``Sendable`` (and thus automatically log values to and/or consume values from the dashboard) is extremely easy: just provide an implementation for the overridable ``initSendable`` method, in which setters and getters for your class's fields are declaratively bound to key values (their display names on the dashboard).

For example, here is the implementation of ``initSendable`` from WPILib's ``BangBangController``:

.. tab-set-code::

    .. remoteliteralinclude:: https://raw.githubusercontent.com/wpilibsuite/allwpilib/v2024.1.1/wpimath/src/main/java/edu/wpi/first/math/controller/BangBangController.java
        :language: java
        :lines: 151-159
        :lineno-match:

    .. remoteliteralinclude:: https://raw.githubusercontent.com/wpilibsuite/allwpilib/v2024.1.1/wpimath/src/main/native/cpp/controller/BangBangController.cpp
        :language: c++
        :lines: 55-69
        :lineno-match:

To enable the automatic updating of values by WPILib "in the background", ``Sendable`` data names are bound to getter and setter functions rather than specific data values.  If a field that you wish to log has no defined setters and getters, they can be defined inline with a lambda expression.

## The SendableBuilder Class

As seen above, the ``initSendable`` method takes a single parameter, ``builder``, of type ``SendableBuilder`` ([Java](https://github.wpilib.org/allwpilib/docs/release/java/edu/wpi/first/util/sendable/SendableBuilder.html), [C++](https://github.wpilib.org/allwpilib/docs/release/cpp/classwpi_1_1_sendable_builder.html), :external:py:class:`Python <wpiutil.SendableBuilder>`).  This builder exposes methods that allow binding of getters and setters to dashboard names, as well as methods for safely ensuring that values consumed *from* the dashboard do not cause unsafe robot behavior.

### Databinding with addProperty Methods

Like all WPILib dashboard code, ``Sendable`` fields are ultimately transmitted over :ref:`NetworkTables <docs/software/networktables/networktables-intro:What is NetworkTables>`, and thus the databinding methods provided by ``SendableBuilder`` match the supported NetworkTables data types:

* ``boolean``: ``addBooleanProperty``
* ``boolean[]``: ``addBooleanArrayProperty``
* ``double``: ``addDoubleProperty``
* ``double[]``: ``addDoubleArrayProperty``
* ``string``:  ``addStringProperty``
* ``string[]``: ``addStringArrayProperty``
* ``byte[]``: ``addRawProperty``

### Ensuring Safety with setSafeState and setActuator

Since ``Sendable`` allows users to consume arbitrary values from the dashboard, it is possible for users to pipe dashboard controls directly to robot actuations.  This is extremely unsafe if not done with care; dashboards are not a particularly good interface for controlling robot movement, and users generally do not expect the robot to move in response to a change on the dashboard.

To help users ensure safety when interfacing with dashboard values, ``SendableBuilder`` exposes a ``setSafeState`` method, which is called to place any ``Sendable`` mechanism that actuates based on dashboard input into a safe state.  Any potentially hazardous user-written ``Sendable`` implementation should call ``setSafeState`` with a suitable safe state implementation.  For example, here is the implementation from the WPILib ``PWMMotorController`` class:

.. tab-set-code::

    .. remoteliteralinclude:: https://raw.githubusercontent.com/wpilibsuite/allwpilib/v2024.1.1-beta-4/wpilibj/src/main/java/edu/wpi/first/wpilibj/motorcontrol/PWMMotorController.java
        :language: java
        :lines: 120-126
        :lineno-match:

    .. remoteliteralinclude:: https://raw.githubusercontent.com/wpilibsuite/allwpilib/v2024.1.1-beta-4/wpilibc/src/main/native/cpp/motorcontrol/PWMMotorController.cpp
        :language: c++
        :lines: 56-62
        :lineno-match:

Additionally, users may call ``builder.setActuator(true)`` to mark any mechanism that might move as a result of ``Sendable`` input as an actuator.  Currently, this is used by :ref:`Shuffleboard <docs/software/dashboards/shuffleboard/getting-started/shuffleboard-tour:Tour of Shuffleboard>` to disable actuator widgets when not in :ref:`LiveWindow <docs/controls-overviews/control-system-software:LiveWindow>` mode.
