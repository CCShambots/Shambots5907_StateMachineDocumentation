---
description: A Framework for Managing the State of Subsystems Developed by FRC Team 5907
---

# Introduction

## What is SMF?

The state machine framework is a way to manage the states of subsystems in Java WPILib. It runs on top of the existing Command-Based framework. It provides an easy syntax to define which states a subsystem is currently in, the valid ways that subsystem can change states, and which commands should be run to transition through states and to maintain that state.

## Why is SMF Necessary?

While developing with the normal command-based framework, it can become difficult to maintain the states of every subsystem, especially as more and more is expected of a robot and all the interweaving tasks it must execute. When our team wrote code for the 2022 season, we found that we needed many commands to run at different times, and each time we needed a command to run, we had to check progressively more flags in different subsystems to make sure that command could run. Occasionally, some flags would not get set correctly, causing strange issues with understanding the states of different subsystems. Our only good solution to this problem was to restart the robot's code because we had no effective way to understand the whole state of each subsystem.

Our solution to this problem is to create a framework that effectively manages the state of each subsystem, so that every subsystem's state is easily viewed and controlled. Team RUSH (FRC 27) already does something similar, controlling the state of each subsystem. However, at least as of their 2019 code release, it does not appear to be in a framework that would be easy for other teams to adapt. We hope to make our version of our state machine implementation easy not only for our own team to use, but for other teams to use as well.
