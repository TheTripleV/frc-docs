# Ramsete Controller

.. warning:: Ramsete Controller has been :term:`deprecated`. Use :doc:`LTV Unicycle Controller <ltv>` which has more intuitive tuning.

The Ramsete Controller is a trajectory tracker that is built in to WPILib. This tracker can be used to accurately track trajectories with correction for minor disturbances for differential drivetrains.

## Constructing the Ramsete Controller Object
The Ramsete controller should be initialized with two gains, namely ``b`` and ``zeta``. Larger values of ``b`` make convergence more aggressive like a proportional term whereas larger values of ``zeta`` provide more damping in the response. These controller gains only dictate how the controller will output adjusted velocities. It does NOT affect the actual velocity tracking of the robot. This means that these controller gains are generally robot-agnostic.

.. note:: Gains of ``2.0`` and ``0.7`` for ``b`` and ``zeta`` have been tested repeatedly to produce desirable results when all units were in meters. As such, a zero-argument constructor for ``RamseteController`` exists with gains defaulted to these values.

.. tab-set-code::
   ```java
   // Using the default constructor of RamseteController. Here
   // the gains are initialized to 2.0 and 0.7.
   RamseteController controller1 = new RamseteController();
   // Using the secondary constructor of RamseteController where
   // the user can choose any other gains.
   RamseteController controller2 = new RamseteController(2.1, 0.8);
   ```

   ```c++
   // Using the default constructor of RamseteController. Here
   // the gains are initialized to 2.0 and 0.7.
   frc::RamseteController controller1;
   // Using the secondary constructor of RamseteController where
   // the user can choose any other gains.
   frc::RamseteController controller2{2.1, 0.8};
   ```

   ```python
   from wpimath.controller import RamseteController
   # Using the default constructor of RamseteController. Here
   # the gains are initialized to 2.0 and 0.7.
   controller1 = RamseteController()
   # Using the secondary constructor of RamseteController where
   # the user can choose any other gains.
   controller2 = RamseteController(2.1, 0.8)
   ```

## Getting Adjusted Velocities
The Ramsete controller returns "adjusted velocities" so that the when the robot tracks these velocities, it accurately reaches the goal point. The controller should be updated periodically with the new reference, which is where it should be at the current time. The reference comprises of a desired pose, desired linear velocity, and desired angular velocity. Furthermore, the current position of the robot should also be updated periodically. The controller uses these four arguments to return the adjusted linear and angular velocity. Users should command their robot to these linear and angular velocities to achieve optimal trajectory tracking.

.. note:: The "reference pose" represents the position that the robot should be at a particular timestep when tracking the trajectory. It does NOT represent the final endpoint of the trajectory, which is the goal.

The controller can be updated using the ``Calculate`` (C++) / ``calculate`` (Java/Python) method. There are two overloads for this method. Both of these overloads accept the current robot position as the first parameter. For the other parameters, one of these overloads takes in the reference as three separate parameters (pose, linear velocity, and angular velocity) whereas the other overload accepts a ``Trajectory.State`` object, which contains information about the reference pose. For its ease, users should use the latter method when tracking trajectories.

.. tab-set-code::

   ```java
   Trajectory.State reference = trajectory.sample(3.4); // sample the trajectory at 3.4 seconds from the beginning
   ChassisSpeeds adjustedSpeeds = controller.calculate(currentRobotPose, reference);
   ```

   ```c++
   const Trajectory::State reference = trajectory.Sample(3.4_s); // sample the trajectory at 3.4 seconds from the beginning
   ChassisSpeeds adjustedSpeeds = controller.Calculate(currentRobotPose, reference);
   ```

   ```python
   reference = trajectory.sample(3.4)  # sample the trajectory at 3.4 seconds from the beginning
   adjustedSpeeds = controller.calculate(currentRobotPose, reference)
   ```

These calculations should be performed at every loop iteration, with an updated robot position and reference.

## Using the Adjusted Velocities
The adjusted velocities are of type ``ChassisSpeeds``, which contains a ``vx`` (linear velocity in the forward direction), a ``vy`` (linear velocity in the sideways direction), and an ``omega`` (angular velocity around the center of the robot frame). Because the Ramsete controller is a controller for non-holonomic robots (robots which cannot move sideways), the adjusted speeds object has a ``vy`` of zero.

The returned adjusted speeds can be converted to usable speeds using the kinematics classes for your drivetrain type. For example, the adjusted velocities can be converted to left and right velocities for a differential drive using a ``DifferentialDriveKinematics`` object.

.. tab-set-code::

   ```java
   ChassisSpeeds adjustedSpeeds = controller.calculate(currentRobotPose, reference);
   DifferentialDriveWheelSpeeds wheelSpeeds = kinematics.toWheelSpeeds(adjustedSpeeds);
   double left = wheelSpeeds.leftMetersPerSecond;
   double right = wheelSpeeds.rightMetersPerSecond;
   ```

   ```c++
   ChassisSpeeds adjustedSpeeds = controller.Calculate(currentRobotPose, reference);
   DifferentialDriveWheelSpeeds wheelSpeeds = kinematics.ToWheelSpeeds(adjustedSpeeds);
   auto [left, right] = kinematics.ToWheelSpeeds(adjustedSpeeds);
   ```

   ```python
   adjustedSpeeds = controller.calculate(currentRobotPose, reference)
   wheelSpeeds = kinematics.toWheelSpeeds(adjustedSpeeds)
   left = wheelSpeeds.left
   right = wheelSpeeds.right
   ```

Because these new left and right velocities are still speeds and not voltages, two PID Controllers, one for each side may be used to track these velocities. Either the WPILib PIDController ([C++](https://github.wpilib.org/allwpilib/docs/release/cpp/classfrc_1_1_p_i_d_controller.html), [Java](https://github.wpilib.org/allwpilib/docs/release/java/edu/wpi/first/math/controller/PIDController.html), :external:py:class:`Python <wpimath.controller.PIDController>`) can be used, or the Velocity PID feature on smart motor controllers such as the TalonSRX and the SPARK MAX can be used.

## Ramsete in the Command-Based Framework
For the sake of ease for users, a ``RamseteCommand`` class is built in to WPILib. For a full tutorial on implementing a path-following autonomous using RamseteCommand, see :ref:`docs/software/pathplanning/trajectory-tutorial/index:Trajectory Tutorial`.

