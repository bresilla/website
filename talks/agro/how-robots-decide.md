---
title: How Robots Decide
sub_title: "Robot architectures and finite state machines · FTE-36306"
author: Trim Bresilla
date: 2026-11-16
theme:
  name: light
  override:
    default:
      colors:
        foreground: "002b47"
        background: "ffffff"
    palette:
      colors:
        accent: "004d00"
        chromatic: "97c600"
    slide_title:
      colors:
        foreground: "004d00"
options:
  incremental_lists: false
---

Today
=====

**How should a robot connect sensing, thinking and acting?** Four steps, from the building blocks to a state machine you program yourself.

![Roadmap: building blocks, three paradigms, state machines, and the Webots assignment](how-robots-decide/roadmap.svg)

*Main reading: Kortenkamp, Simmons & Brugali (2016), Robotic systems architectures and programming, Springer Handbook of Robotics*

<!-- end_slide -->

<!-- jump_to_middle -->

Building blocks
===============

What every robot program is made of

![](how-robots-decide/section-both.svg)

<!-- end_slide -->

Sense, plan, act
================

Every robot program is built from three primitives (Murphy, 2019).

![The three robot primitives: sense turns sensor data into information, plan turns information into directives, act turns them into actuator commands](how-robots-decide/primitives.svg)

<!-- end_slide -->

Two big ideas
=============

How do we put SENSE, PLAN and ACT together? Two ideas: the **architecture**, how the program is built (Kortenkamp et al., 2016), and the **paradigm**, how SENSE, PLAN and ACT are wired (Murphy, 2019).

![Two big ideas. Architecture: structure and style, time and task. Paradigms: hierarchical, reactive and hybrid](how-robots-decide/hub.svg)

<!-- end_slide -->

<!-- jump_to_middle -->

Architecture
============

How a robot program is built

![](how-robots-decide/section-arch.svg)

<!-- end_slide -->

Structure & style
=================

![Architecture, structure and style: which parts, and how they talk. Topics: structure, style](how-robots-decide/div-ss.svg)

<!-- end_slide -->

Structure: which parts, and which are connected
===============================================

![image:width:100%](how-robots-decide/crumb-ss-1.svg)

**Structure** is the boxes and the lines. The same sensors and motors can be wired very differently, and that choice is what the three paradigms will disagree about.

![Two structures for the same sensors and motors. A: GPS and distance sensor feed one world model and one planner, which drives both motors. B: GPS feeds follow path, the distance sensor feeds avoid obstacles, and both feed coordinate motion, which drives the motors](how-robots-decide/structure.svg)

<!-- end_slide -->

Style: how connected parts talk
===============================

![image:width:100%](how-robots-decide/crumb-ss-2.svg)

**Style** is what a line means: how data travels along a connection, and above all **who waits for whom**.

![Three communication styles with timelines. Publish-subscribe: the sender sends messages at its own rate and nobody waits. Call and wait: the caller is blocked while the planner thinks. Shared memory: the battery writes the latest value into a store and the executive reads the newest value when it wants](how-robots-decide/style.svg)

<!-- end_slide -->

Time & task
===========

![Architecture, time and task: how the jobs are split into layers. Topics: time, task](how-robots-decide/div-tt.svg)

<!-- end_slide -->

A robot does many jobs at once
==============================

![image:width:100%](how-robots-decide/crumb-tt-0.svg)

So which parts should the program have? Look at what the robot does: many **jobs at the same time**, some answering within milliseconds, others with minutes to think.

![A field robot in a crop row with six jobs around it, each with how fast it must answer: avoid obstacles, follow path and coordinate motion in milliseconds; headland turn and pick next row in seconds to minutes; plan coverage route in minutes to hours](how-robots-decide/many-jobs.svg)

Architectures **decompose** these jobs into layers, in two ways: by **time** and by **task**.

<!-- end_slide -->

Split by time: slow on top, fast below
======================================

![image:width:100%](how-robots-decide/crumb-tt-2.svg)

By **time**: sort the jobs by how often they must answer. Each layer runs about **10× slower** and is **more abstract** than the one below.

![Loop rates of a field robot grouped into planning, sequencing and control layers on a logarithmic time axis; each layer is slower and more abstract than the one below](how-robots-decide/split-time.svg)

<!-- end_slide -->

Split by task: big jobs become small steps
==========================================

![image:width:100%](how-robots-decide/crumb-tt-3.svg)

By **task**: a job on one layer is done by calling jobs on the layer below, **in order**.

![Task decomposition: scout the field calls move to start, navigate row and headland turn in order; navigate row calls follow path and avoid obstacles, which feed coordinate motion and the motors](how-robots-decide/split-task.svg)

Open question: where does **PLAN** sit, and does **ACT** have to wait for it? Robotics has tried three answers.

<!-- end_slide -->

<!-- jump_to_middle -->

Paradigms
=========

Three ways to connect sense, plan and act

![](how-robots-decide/section-par.svg)

<!-- end_slide -->

Hierarchical
============

![Paradigms, hierarchical: sense, plan, act. Topics: Shakey, world model, limits](how-robots-decide/div-hier.svg)

<!-- end_slide -->

Shakey: sense, then plan, then act
==================================

![image:width:100%](how-robots-decide/crumb-hier-1.svg)

<!-- column_layout: [3, 2, 2] -->

<!-- column: 0 -->

* **Shakey** (SRI, late 1960s): the first robot that reasoned about its own actions
* Camera, rangefinder and bump sensors; differential drive
* Strictly in turn: **sense** the world, **plan** on a world model, **act**

<!-- column: 1 -->

![Hierarchical paradigm: sense, then plan on a world model, then act](how-robots-decide/paradigm-hierarchical.svg)

<!-- column: 2 -->

![Shakey the robot with labels for antenna, camera, range finder, bump detectors and drive wheels](how-robots-decide/shakey.jpg)

<!-- reset_layout -->

*Photo: SRI International, CC BY-SA 3.0, via Wikimedia Commons*

<!-- end_slide -->

Plan on a world model
=====================

![image:width:100%](how-robots-decide/crumb-hier-2.svg)

<!-- column_layout: [1, 1] -->

<!-- column: 0 -->

* All sensor data goes into one **world model**
* A **planner** searches the model for the best sequence of actions
* **A\*** on a grid map was invented for Shakey (Hart, Nilsson & Raphael, 1968)
* Optimal and predictable, **if** the model is right

<!-- column: 1 -->

![A* search on a grid map: expanded cells shaded and the shortest path from S to G in green](how-robots-decide/astar.svg)

<!-- reset_layout -->

<!-- end_slide -->

The world does not wait
=======================

![image:width:100%](how-robots-decide/crumb-hier-3.svg)

* Planning takes **long**, and the robot is **blocked** while it waits
* The plan runs **without sensing**, which is dangerous in a dynamic world

![Timeline: the robot senses once, plans for twelve seconds, then executes blind while a person walks into the row](how-robots-decide/blind.svg)

<!-- end_slide -->

Reactive
========

![Paradigms, reactive: sense, act. Topics: behaviours, subsumption, motor schemas, limits](how-robots-decide/div-react.svg)

<!-- end_slide -->

Cut out the planner
===================

![image:width:100%](how-robots-decide/crumb-react-1.svg)

![Left: a pipeline of functional modules between sensors and actuators. Right: parallel task-achieving layers that each connect sensors to actuators](how-robots-decide/decomposition.svg)

Brooks (1986): instead of one pipeline of functions, many **behaviours**, each a direct link from SENSE to ACT, all running at once.

<!-- end_slide -->

Subsumption: layers of behaviours
=================================

![image:width:100%](how-robots-decide/crumb-react-2.svg)

![Subsumption for a field robot: level 2 suppresses level 1, level 1 suppresses level 0, and the halt reflex goes straight to the wheels](how-robots-decide/subsumption.svg)

Layers run **in parallel**. A higher layer **suppresses** (S) the command of the layer below: arbitration by priority (Brooks, 1986).

<!-- end_slide -->

Motor schemas: blend behaviours
===============================

![image:width:100%](how-robots-decide/crumb-react-3.svg)

<!-- column_layout: [2, 3] -->

<!-- column: 0 -->

* Arkin (1989): every behaviour outputs a **vector**
  * *move-to-goal* pulls towards the goal
  * *avoid-obstacle* pushes away, harder when closer
* The robot follows the **sum**
* Subsumption **picks** one behaviour, motor schemas **blend** them all

<!-- column: 1 -->

![Motor schemas: a move-to-goal vector and an avoid-obstacle vector add up and the path bends around a rock to the goal](how-robots-decide/schemas.svg)

<!-- reset_layout -->

<!-- end_slide -->

Fast, but short-sighted
=======================

![image:width:100%](how-robots-decide/crumb-react-4.svg)

<!-- column_layout: [2, 3] -->

<!-- column: 0 -->

* Responds in milliseconds, needs no world model
* But:
  * **No memory**: which rows are already done?
  * **No foresight**: when should it go and charge?
  * **Hard to predict**: behaviour emerges
  * **Gets stuck**: forces can cancel out

<!-- column: 1 -->

![A U-shaped obstacle: attraction and repulsion cancel inside the cup and the robot gets stuck short of the goal](how-robots-decide/trap.svg)

<!-- reset_layout -->

<!-- end_slide -->

Hybrid
======

![Paradigms, hybrid: plan, then sense and act. Topics: three layers, executive](how-robots-decide/div-hybrid.svg)

<!-- end_slide -->

Plan on top, react below
========================

![image:width:100%](how-robots-decide/crumb-hybrid-1.svg)

![Three-layer architecture: planning, executive and behavioural control with their time scales and state; commands flow down and failures flow up](how-robots-decide/three-layer.svg)

The time and task layers from the start, each with one job (Firby; 3T; Gat, 1998): behaviours below, a planner on top, and an **executive** in between.

<!-- end_slide -->

The executive switches behaviours
=================================

![image:width:100%](how-robots-decide/crumb-hybrid-2.svg)

![The executive layer as a finite state machine: the active state switches on one behaviour below it, and events from the behaviours trigger transitions](how-robots-decide/executive-fsm.svg)

The executive turns the plan into **one behaviour at a time** and switches when an **event** happens. Most robots today are hybrid, and this executive is often a **finite state machine**: what you will build.

<!-- end_slide -->

<!-- jump_to_middle -->

State machines
==============

Programming the executive layer

![](how-robots-decide/section-fsm.svg)

<!-- end_slide -->

Anatomy of a state machine
==========================

The robot's battery as a state machine: it scouts until the battery runs low, drives back to the dock, and charges.

![Anatomy of a state machine on the robot battery example: initial state, active state, states, events and transitions](how-robots-decide/fsm-anatomy.svg)

<!-- end_slide -->

States can have several transitions
===================================

The robot must also stop for people: one new state, **Wait**. *Scout rows* now has **two** ways out, and the event decides which one is taken.

![Left: battery state machine with scout rows, return to dock and charge. Right: the same machine with a wait state, entered on person ahead and left on path clear](how-robots-decide/fsm-battery.svg)

<!-- end_slide -->

CropScout: three behaviours, three states
=========================================

<!-- column_layout: [1, 1] -->

<!-- column: 0 -->

A field robot that scouts a crop row after row, with three behaviours:

![CropScout's three behaviours in the field: move to the start of a row, navigate through the row, make a headland turn into the next row](how-robots-decide/cropscout-paths.svg)

<!-- column: 1 -->

What does the FSM look like?

<!-- pause -->

![CropScout state machine: move to start of row, navigate through row and make headland turn, with self-loops while a state is still busy](how-robots-decide/fsm-cropscout.svg)

<!-- reset_layout -->

<!-- end_slide -->

One state at a time
===================

![A CropScout run over time: the active state switches from move to start, to navigate row, to headland turn and back, only when an event fires](how-robots-decide/fsm-trace.svg)

<!-- end_slide -->

Every arrow is one row
======================

![CropScout transition table drawn as rows of state, event and next state, with self-loops greyed out](how-robots-decide/fsm-table.svg)

<!-- end_slide -->

How an FSM runs, cycle after cycle
==================================

![The FSM control loop: read the sensors, check the events of the current state, switch state if an event fired, run the state's behaviour, repeat](how-robots-decide/fsm-loop.svg)

<!-- end_slide -->

Your turn
=========

<!-- column_layout: [1, 1] -->

<!-- column: 0 -->

![Five steps to build a state machine: behaviours to states, events to transitions, draw the diagram, write the table, code the loop and test](how-robots-decide/fsm-recipe.svg)

<!-- column: 1 -->

1. Notebook *t08_state_machine*
2. Webots assignment: *t08_state_machine_webots_assignment.pdf* and *t08_webots.zip*

[Expected behaviour](robot-architectures/expected-behaviour.mp4)

<!-- reset_layout -->
