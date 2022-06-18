# Syntax and Basic Functionality

## Example Subsystem

As an example of how the syntax of the SMF should be used, consider the diagram for the behaviors of a turret, as shown below.

![Example Turret subsystem logic (created in Miro)](<.gitbook/assets/22O Logic - Frame 2.jpg>)

In the above example, different colors represent different parts of the SMF. They are as follows:

* Black: A normal state
* Gray: A flag state
* Dark yellow: The transition by which the subsystem determines its state (Note: As you'll see later, this is syntactically the same as a normal transition)
* Light pink: Normal transitions
* Green: Continuous commands that will run whilst the subsystem is in a certain state
* Light Yellow: The condition for a flag state to trigger

## The Subsystem Class

The example turret described above will have a subsystem class that looks something like this

{% code title="Turret.java" %}
```java
public class Turret extends StatedSubsystem<Turret.TurretState> {


    public Turret() {
        super(TurretState.class);
    }
    
    public enum TurretState {
        Undetermined, Idle, Odometry, Active, LockedIn
    }

}
```
{% endcode %}

The use of the generic in the class declaration as well as in the constructor ensures that only that enum class can be passed as values for the subsystem. Technically, the same enum could be used for different subsystems, but I would not recommend doing that.

{% hint style="info" %}
You'll notice that the subsystem has an enum that describes all the states in which the subsystem can be. Although `LockedIn` is going to become a flag state, it is still defined in the same enum as the other states.
{% endhint %}

## Defining Transitions

{% hint style="info" %}
All behavioral setup for the subsystem (defining transitions, continuous commands, and flag states) should be done in the constructor. While you could add other transitions while the robot code is already running, I would not recommend it, as it could lead to unexpected behavior)
{% endhint %}

There are two types of transitions a subsystem has. It has **exactly one** determination transition, and any number of normal transitions. A determination transition can be defined as follows:

{% code title="Turret.java Constructor" %}
```java
addTransition(Idle, Active, new EnableLimelightCommand());

//Alternate method version
addTransition(Idle, Active, Odometry, new LimelightCommand());
```
{% endcode %}

The first argument of `addTransition` is the starting state of the subsystem, the second argument is the state the subsystem will be in after running the command, and the third argument, is the command that the subsystem will run, here just an example.

The alternate method version provides another argument for a "cancel state", which is the state that the subsystem will revert to in case of the transition being canceled. For example, a subsystem could revert back to the undetermined state if interrupted (because perhaps the transition could leave the subsystem in an odd state if interrupted)

To define how a subsystem should determine its state, use the `addDetermination` method instead. It defines a normal transition, but it also defines important variables that will be used by the subsystem to determine its own state.

{% hint style="info" %}
The `addDetermination` method is mandatory for every subsystem
{% endhint %}

## Defining Continuous Commands

Continuous commands will also be defined in the constructor with the following method:

{% code title="Turret.java constructor" %}
```java
setContinuousCommand(Active, new LimelightTrackingCommand());
```
{% endcode %}

Here the first argument is the state during which the continuous command should run, and the second argument is the command to be run.

## Instance-Based States

Instance-Based states can be defined with the following method:

{% code title="Turret.java constructor" %}
```java
addInstanceBasedState(Active);
```
{% endcode %}

## Defining Flag States

Flag states are defined with the following method in the constructor.

{% code title="Turret.java constructor" %}
```java
addFagState(Active, LockedIn, () -> true);
```
{% endcode %}

Here, the first argument is the parent state, the second argument is the flag state, and the third argument is a `BooleanSupplier` that indicates whether the flag state should be active or not

{% hint style="info" %}
Multiple flag states can be defined for the same parent state, but if multiple of them have suppliers that return true at once, only one flag state will be activated. This is based on the order in which the flag states are defined. The states later in the list will overwrite the current flag state when the `periodic` method checks for flag states every 20ms.
{% endhint %}

## update()

The `update` method is the replacement for the `periodic` method to be used in actual subsystems. This is because the `periodic` method needs to be used for subsystem management.

## Full Example

Here is what the entire class would look like (with example commands):

{% code title="Turret.java" %}
```java
public class Turret extends StatedSubsystem<Turret.TurretState> {

    public Turret() {
        super(TurretState.class);

        addDetermination(Undetermined, Idle, new InstantCommand());

        addTransition(Idle, Active, new InstantCommand()); //Enable limelight
        addTransition(Active, Idle, new InstantCommand()); //Disable limelight
        addTransition(Idle, Odometry, new InstantCommand());
        addTransition(Odometry, Idle, new InstantCommand());
        addTransition(Odometry, Active, new InstantCommand()); //Enable limelight
        addTransition(Active, Odometry, new InstantCommand()); //Disable limelight

        setContinuousCommand(Active, new RunCommand(() -> {})); //Actively track with limelight
        setContinuousCommand(Odometry, new RunCommand(() -> {})); //Enable odometry

        addFlagState(Active, LockedIn, () -> true); //Indicate if the turret is locked in

    }

    public enum TurretState {
        Undetermined, Idle, Odometry, Active, LockedIn
    }

    @Override
    public void update() {

    }
}
```
{% endcode %}

## Causing State Transitions

Using the subsystem's `requestTransition` method is the most basic way to make a state transition. It is a raw method, not enclosed in a command. If a current transition is ongoing, it will be canceled.

{% hint style="info" %}
When requesting a transition, if a flag state is inputted as the desired state, the subsystem will find that flag state's parent state and request to transition to that (as flag states cannot be directly transitioned to, but only become active when a condition is true).
{% endhint %}

### waitForState()

A factory that returns a command that will run continuously until the inputted state is reached. If interrupted, the command will cancel the subsystem's current transition (if one is occuring). Additionally, the inputted state can be a flag state, and the command will run until that flag state is reached.

### goToState()

The command is the same as `waitForState`, but it also requests a transition in the `init()` method of the command
