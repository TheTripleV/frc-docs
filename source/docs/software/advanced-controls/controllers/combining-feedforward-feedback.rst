.. include:: <isonum.txt>

# Combining Feedforward and PID Control

.. todo:: link to conceptual article when available

.. note:: This article covers the in-code implementation of combined feedforward/PID control with WPILib's provided library classes.  Documentation describing the involved concepts in more detail is forthcoming.

Feedforward and feedback controllers can each be used in isolation, but are most effective when combined together.  Thankfully, combining these two control methods is *exceedingly* straightforward - one simply adds their outputs together.

## Using Feedforward with a PIDController

Users may add any feedforward they like to the output of the controller before sending it to their motors:

.. tab-set-code::

  ```java
  // Adds a feedforward to the loop output before sending it to the motor
  motor.setVoltage(pid.calculate(encoder.getDistance(), setpoint) + feedforward);
  ```

  ```c++
  // Adds a feedforward to the loop output before sending it to the motor
  motor.SetVoltage(pid.Calculate(encoder.GetDistance(), setpoint) + feedforward);
  ```

  ```python
  # Adds a feedforward to the loop output before sending it to the motor
  motor.setVoltage(pid.calculate(encoder.getDistance(), setpoint) + feedforward)
  ```

Moreover, feedforward is a separate feature entirely from feedback, and thus has no reason to be handled in the same controller object, as this violates separation of concerns.  WPILib comes with several helper classes to compute accurate feedforward voltages for common FRC\ |reg| mechanisms - for more information, see :ref:`docs/software/advanced-controls/controllers/feedforward:Feedforward Control in WPILib`.

## Using Feedforward Components with PID

.. note:: Since feedforward voltages are physically meaningful, it is best to use the ``setVoltage()`` ([Java](https://github.wpilib.org/allwpilib/docs/release/java/edu/wpi/first/wpilibj/motorcontrol/MotorController.html#setVoltage(double)), [C++](https://github.wpilib.org/allwpilib/docs/release/cpp/classfrc_1_1_motor_controller.html#a613c23a3336e103876e433bcb8b5ad3e), :external:py:meth:`Python <wpilib.interfaces.MotorController.setVoltage>`) method when applying them to motors to compensate for "voltage sag" from the battery.

What might a more complete example of combined feedforward/PID control look like?  Consider the :ref:`drive example <docs/software/advanced-controls/controllers/feedforward:Using Feedforward to Control Mechanisms>` from the feedforward page.  We can easily modify this to include feedback control (with a ``SimpleMotorFeedforward`` component):

.. tab-set-code::

  ```java
  public void tankDriveWithFeedforwardPID(double leftVelocitySetpoint, double rightVelocitySetpoint) {
    leftMotor.setVoltage(feedforward.calculate(leftVelocitySetpoint)
        + leftPID.calculate(leftEncoder.getRate(), leftVelocitySetpoint));
    rightMotor.setVoltage(feedForward.calculate(rightVelocitySetpoint)
        + rightPID.calculate(rightEncoder.getRate(), rightVelocitySetpoint));
  }
  ```

  ```c++
  void TankDriveWithFeedforwardPID(units::meters_per_second_t leftVelocitySetpoint,
                                   units::meters_per_second_t rightVelocitySetpoint) {
    leftMotor.SetVoltage(feedforward.Calculate(leftVelocitySetpoint)
        + leftPID.Calculate(leftEncoder.getRate(), leftVelocitySetpoint.value()));
    rightMotor.SetVoltage(feedforward.Calculate(rightVelocitySetpoint)
        + rightPID.Calculate(rightEncoder.getRate(), rightVelocitySetpoint.value()));
  }
  ```

  ```python
  def tank_drive_with_feedforward_PID(
      left_velocity_setpoint: float,
      right_velocity_setpoint: float,
  ) -> None:
      leftMotor.setVoltage(
          feedforward.calculate(left_velocity_setpoint)
          + leftPID.calculate(leftEncoder.getRate(), left_velocity_setpoint)
      )
      rightMotor.setVoltage(
          feedforward.calculate(right_velocity_setpoint)
          + rightPID.calculate(rightEncoder.getRate(), right_velocity_setpoint)
      )
  ```

Other mechanism types can be handled similarly.
