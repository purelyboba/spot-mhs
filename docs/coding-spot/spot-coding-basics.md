---
title: Spot Coding Basics
---

???+ abstract "Objective"
    You will be introduced to some basic concepts that may be useful when coding more complex programs with Spot.

If you haven't already, please follow the setup instructions for the Spot SDK (Software Development Kit) [here](https://github.com/boston-dynamics/spot-sdk/blob/master/docs/python/quickstart.md). This page will go over some of the most important parts of Spot setup. The SDK will allow you to communicate with Spot with custom code.

## Connecting to spot

Spot emits its own WiFi signal, which can be connected to from your computer.

Once you are connected with WiFi, you can run the following command in your terminal to confirm the connection:

```py
python3 -m bosdyn.client 192.168.80.3 id
```

This command will get the ID of Spot. If Spot is connected, it should provide information about the Spot ID and Software version.

## Using Git

Git is a useful tool for code management and keeping track of changes you add to your code. This is formally known as a version control system (VCS).

You will be using Git to download the software development kit (SDK ) for Spot. The SDK repository is [here](https://github.com/boston-dynamics/spot-sdk).

You can learn about Git [here](https://git-scm.com/doc).

You can learn about GitHub [here](https://docs.github.com/en/get-started/quickstart/hello-world).

## Using the SDK

Once you have Git set up and the SDK cloned, most of your coding will be in the ```./python/examples``` folder. This is where pre-written examples of the SDK are for you to learn from.

## E-Stop

"E-Stop" refers to an "Emergency Stop" service that is constantly being run in the background of your code. Your code **will not run** without an E-Stop. 

The Emergency Stop system is designed to rapidly shut down the robot's motors and other critical functions to prevent harm to humans or damage to the robot. It can be triggered manually by a human operator in emergencies or can be activated automatically by various safety sensors or software conditions.

You will be able to run a GUI (or text-based) version of the E-Stop that you can manually use through your code.


Setup and verification of estop:
```py
# Verify that Spot is NOT estopped and has an estop client configured
assert not robot.is_estopped(), 'Robot is estopped, Please use an external E-Stop client, ' \ 
'such as the estop SDK example, to configure E-Stop.'
```

Implementation of estop control:
```py
def stop(self): # stop Spot
    self.estop_keep_alive.stop()
def allow(self): # remove the stop and allow Spot to move again
    self.estop_keep_alive.allow()
def settle_then_cut(self): # Spot will attempt to sit down first before cutting motor power
    self.estop_keep_alive.settle_then_cut()
```

## Lease

A "lease"  is used to manage and allocate resources among multiple controllers or software components that may need to interact with the robot simultaneously while avoiding conflicts and ensuring safe and coordinated operation.

**There can only be one lease owner** at any time. The owner of the lease is able to give the robot instructions, however an application can delegate the lease to other systems to perform various tasks. In most basic situations, only one program will have the lease for the entire duration of the mission.

```py
lease_client = robot.ensure_client(bosdyn.client.lease.LeaseClient.default_service_name)
    with bosdyn.client.lease.LeaseKeepAlive(lease_client, must_acquire=True, return_at_exit=True):
        # all robot commands go here
```

## Basic code format

The basic format of the code is simple. The following components are needed to start a "template" python file to build off of when coding Spot:

- *import* commands to import all necessary packages for controlling Spot

    ```py
    import argparse
    import os
    import sys
    import time
    import bosdyn.client
    import bosdyn.client. lease
    import bosdyn.client.util
    import bosdyn.geometry
    from bosdyn.api import trajectory_pb2
    from bosdyn.api.spot import robot_command_pb2 as spot_command_pb2
    ```

- A main function to call any other helper functions and get command line arguments as parameters

    ```py
    def main(argv):
        parser = argparse. ArgumentParser() # Set up parsing system for command-line parameters
        bosdyn.client.util.add_base_arguments (parser)
        parser.add_argument( # Add acceptable arguments that can be used with the file
            '-S', '--save', action='store_true', help= 'Save the image captured by Spot to the working directory. To chose the save location, use --save_path instead. '
        )
        options = parser.parse_args(argv)
        try:
            hello_spot (options) # If the Python file is run correct, the hello_spot() helper function will run with all the necessary commands
            return True
        except Exception as exc:
            logger = bosdyn.client.util.get_logger()
            Logger.error('Hello, Spot! threw an exception: %r', exc) # If there is an error with the input/syntax, throw the exception into the logger
            return False
    ```

- An entry point to call the main function

    ```py
    if __name__ == '__main__': # This belongs at the END of your python file
        if not main(sys.argv[1:]):
            sys.exit(1)
    ```

## Running a file

To run a Python file for Spot, use the following command syntax:

```py
python3 <file_name> 192.168.80.3
```

192.168.80.3 is the local IP address of Spot on its WiFi signal.