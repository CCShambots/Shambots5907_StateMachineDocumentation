---
description: Easily Enable, Disable, and Determine Subsystems
---

# Subsystem Manager

The `SubsystemManager` class is a singleton object meant to help with enabling, disabling, and determining the state of all subsystems in your codebase

### Adding Subsystems to the Subsystem Manager

{% code title="RobotContainer.java " %}
```java
private final ExampleSubsystem s = new ExampleSubsystem();

public RobotContainer() {
    SubsystemManager.getInstance().registerSubsystem()
}

```
{% endcode %}

{% hint style="info" %}
These can really be added anywhere, but I would recommend constructing all your subsystems before `RobotContainer.java`'s constructor, and register all your subsystems in the constructor of `RobotContainer.java`
{% endhint %}

### Enabling All Subsystems

```java
SubsystemManager.getInstance().enableAllSubsystems();
```

### Disabling All Subsystems

```java
SubsystemManager.getInstance().disableAllSubsystems();
```

### Determine State of All Subsystems

```java
SubsystemManager.getInstance().determineAllSubsystems();
```

This method returns a command that will run the transition to enable any and all subsystems that are still in their undetermined state.

### Prep Subsystems

This is a convenience method provided that will return the command from `determineAllSubsystems()`, but will also call `enableAllSubsystems()`.
