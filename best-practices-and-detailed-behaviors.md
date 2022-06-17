# Best Practices and Detailed Behaviors

## Orchestrating Complex Behaviors

Complex commands that require interactions between several subsystems can be accomplished through the use of normal command groups along with the `goToState` and `waitForState` command factories. These transitions should be robust with command groups like `ParallelRaceGroup`, although I wouldn't necessarily recommend their use unless absolutely necessary.

If you require more granular control of multiple subsystems transitioning and the included command groups do not meet your needs, you would also be able to manually call the `requestTransition`, `isInState`, and `getCurrentState` methods on the subsystem

## Behavior Explanations

{% hint style="warning" %}


All "error" messages from attempts to do invalid things (i.e. transitions that already exist, identical command objects, etc.). Details about what went wrong will be printed in the console, but the robot code will still run. Any attempts to register invalid things (transitions, continuous commands, and flag states), will not be registered until the issue is corrected.
{% endhint %}

### Transition to undetermined state

Your transitions cannot go to or from the undetermined state. However, the state a transition goes to when interrupted can be

### Command Requirements

Commands both in transitions and continuous commands should not have any requirements or should require only the subsystem for which you are registering them.

### Flag States vs. Normal States

Flag states and normal states are mutually exclusive. The following rules apply when using them

* Transitions cannot include start, end, or interruption states that are flag states
* Flag states cannot include any state that is registered in a transition
* Flag states cannot be parent states and vice versa
* Flag states cannot have continual commands

### Canceling Transitions

Primarily, canceling transitions can be accomplished by calling the `cancelTransition` method on a subsystem. This cancels any transition that is currently running and returns `true` or `false` based on whether a transition was actually canceled.

### Continuous Commands

A subsystem may only have one continuous command for any state.

## Best Practices

### Don't Expose Methods

Subsystem methods that cause changes to the subsystem state should be `private` wherever possible. This would include methods to, for example, lower an intake

