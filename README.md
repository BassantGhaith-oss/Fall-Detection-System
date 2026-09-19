# Fall Detection System

A simple fall detection system designed to detect when a person may have fallen.

## Idea

The system detects a possible fall by monitoring the person's acceleration.

The detection process has three stages:

* **Normal / Stable:** The person is in a normal state.
* **Fall Suspected:** A high acceleration is detected, which may indicate a fall.
* **Fall Confirmed:** The high acceleration is followed by a period of stillness, which confirms that a fall likely happened.

## How It Works

The system continuously monitors acceleration and checks for a sudden increase followed by stillness.

This helps distinguish a possible fall from normal movement.

## Simulation

The project was designed and tested using Wokwi.

[Wokwi Project](https://wokwi.com/projects/472542265678315521)

[Project presentation](https://canva.link/7i7f3o2zd2aiqll)
