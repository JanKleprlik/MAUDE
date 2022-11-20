topic: algebraic specifications in Maude

deadline: 23.11.2022

# TASK
```
Intensive Care Unit
===================
task 1: create algebraic specification of a control system at intensive care unit (ICU)
	- what you should capture in your model (specification, prototype):
		many dynamic characteristics (properties) of a human body (physiology) are continuously monitored (heart rate, blood pressure, temperature, breathing frequency, neural/brain signals, etc)
		in case of a problem observed by some of the sensors and monitors, the control system quickly decides that some machine will respond by some action (for example, the dropper will immediately start to deliver more fluid of some kind)
	- general principle: the control system of ICU receives inputs from various sensors, produces some outputs (reports), and performs certain actions

task 2: document your solution
	- explain key decisions and high-level design

task 3: prepare some test cases (scenarios, inputs)
	- common scenarios that may occur in a real hospital
```	

# Solution

ICU was implemented in two parts. `Sensors` and `Devices` groupped in the `ICU`. 

Each sensor has its own minimum and maximum values that are considered acceptable. (I ommited identification of such sensors by ID for brevity.) We can set the current value of each sensor as well as we can obtain current value. In real life those values would be hardware pulses. In our case we update them manually. The main operations is `check` which "specialises" on sensors via defined equations.(all sensors are `GenericSensors`).

The ICU is made of devices. Each device has one sensor for which it is responsible and reacts to its changes. Here I have tried to make only generic device which would specialise by the sensor it contains (see commented part in `device.maude` where the idea is that the operation `getDeviceStatus` would get resolved by the `chceck` operation but I have quickly realised that this is not how it works).

The most important operations are `getPatientCondition` and `reactToSensors`. The first one reports the condition of the patient which is connected to the sensors -> if any of the sensors is outside the bonds the conidtion is considered unstable, stable otherwise. The latter operation performs adequate action to the first failing sensor that it finds. Here I have tried the conditional equation approach as opposed to the mixin `if_then_else_` as in the `sensor.maude` file. Possible improvement here could be to react to all failed sensors and not only to the first one.

It may be worth noting that I have struggled finding an issue how "hiearchy" works, because I have swapped left and right sides of the subset definition.

# Test cases
In the file `tests.maude` I have prepared a few contstant ICU units which have different configurations of sensors. A reduce functions can be used on expressions such as:

```
    reduce getPatientCondition(icuAllGood) .
    reduce getPatientCondition(icuLowHeartRate) .
    reduce getPatientCondition(icuLowPressure) .
    reduce getPatientCondition(icuHighBreathing) .
    reduce getPatientCondition(icuMultipleHigh) .
    reduce getPatientCondition(icuMultipleLow) .
    reduce getPatientCondition(icuHighAndLow) .

    reduce reactToSensors(icuAllGood) .
    reduce reactToSensors(icuLowHeartRate) .
    reduce reactToSensors(icuLowPressure) .
    reduce reactToSensors(icuHighBreathing) .
    reduce reactToSensors(icuMultipleHigh) .
    reduce reactToSensors(icuMultipleLow) .
    reduce reactToSensors(icuHighAndLow) .
```

I have tested them manually and all have resolved as expected.
