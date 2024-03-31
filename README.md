# PWMServoWrapper

Servo wrapper class written for easier control of PWM servo motors! Includes methods for driving servos with non-blocking code!

This class uses the Adafruit PWM Servo Driver library for sending pwm values to the servos and includes multiple methods for controlling servos.

This class is intended to be used with a PC9685 Servo Driver.

## Control

As of writing, there are three main ways to control a servo:

- `void directDrive(int degree)` - Drives a servo directly to a given degree
- `void update()` - This method is used in your main `loop()` function and will call one of the two functions:
  - `void updateWithInterval()` - Non-blocking function which will move a servo to a target position based on a set interval (ex. 30ms between each step). Call `setMovement(int degree, int stepInterval)` **_once_** to set a new target and interval. Call this function again to change target and interval as needed.
  - `void updateWithDuration()` - Non-blocking function which will move a servo to a target position within a set duration (ex. Move to 50deg within 2 seconds). Call `setMoveWithDuration(int targetDegree, int durationMillis)` **_once_** to set a new target and duration. Call this function again to change target and duration as needed.

## Usage

This class is structured to control one servo using a `Adafruit_PWMServoDriver` object for sending pwm signals. This allows us to make multiple `PWMServoWrapper` objects for specific control of a servo. Each servo can have their own min and max pulse widths defined, as well as being labeled for "inverse" control.

### Instantion

The constructor takes 5 parameters:
`PWMServoWrapper(int pin, int maxPulse, int minPulse, bool inverse, Adafruit_PWMServoDriver *pwm)`

- `pin`: This refers to the pin on the PC9685 driver board
- `maxPulse`, `minPulse`: Min and max pulse length count for _this_ particular servo
- `inverse`: Inverts degree inputs which can be useful for servos physically arranged in particular ways (inverse is based on servo's limits of 0-180 deg, so the inverse of 135deg would be 45deg)
- `*pwm`: Reference to a `Adafruit_PWMServoDriver` object which should be defined in your main code

### Important!

This classes uses an internal variable `pos` as the starting position for `update()`. By default, this variable is set to `0`! Once the servo has moved to a target position, `pos` will be the last known target and will become the new starting point. **It is recommended to use `setStartPos(int degree)` in `setup()` or some other startup function to clearly define the starting point of the servo on first startup!**

### Example

```c++
#include <Adafruit_PWMServoDriver.h>
#include "PWMServoWrapper.hpp"

#define SERVO_FREQ 50

Adafruit_PWMServoDriver pwm = Adafruit_PWMServoDriver(); // pwm object
PWMServoWrapper myServo(0, 490, 100, false, &pwm);  // object for our servo

void setup()
{
    Serial.begin(9600);
    // setup for pwm (checkout Adafruit for more detail on this)
    pwm.begin();
    pwm.setOscillatorFrequency(27000000);
    pwm.setPWMFreq(SERVO_FREQ);

    myServo.setStartPos(90);       // set start position to 90 degrees
    myServo.setMovement(45, 50);   // set target to 45 degrees with 50ms interval
}

void loop()
{
    /* Update our servo every loop. The servo will move incrementally to the target position. This is
    safe to call every loop due to the implementation of non-blocking code in the class. */
    myServo.update();
}
```
