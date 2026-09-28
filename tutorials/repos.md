# The ARCUS repos

The ARCUS project is divided in many repos on Github. This section is intended as a brief overview of each repo to make you understand better where each part of the code is located and where to add you own!

## [arcus-documentation](https://github.com/robotique-udes/arcus-documentation)

This is the repo you are currently in right now! It is where we store important information and procedures regarding the project as a whole. It includes the tutorials and explanation on logic/handling of our main algorithms.

## [arcus-vesc](https://github.com/robotique-udes/arcus-vesc)

`arcus-vesc` stores code and configuration of our [VESC](https://en.wikipedia.org/wiki/Electronic_speed_control). The VESC acts as the bridge between our controller and the different motors. The repo is used mainly for configuration and we rarely change its code.

## [arcus-simulation](https://github.com/robotique-udes/arcus-simulation)

`arcus-simulation` contains code relating to the **RViz** simulation and the scripts that are only used on our local computers (not needed on the car/Jetson). For example, all of our GUI is in this repo.

## [arcus](https://github.com/robotique-udes/arcus)

`arcus` contains the core code located directly on the car (Jetson). It contains mostly our autonomous driving algorithms and nodes.