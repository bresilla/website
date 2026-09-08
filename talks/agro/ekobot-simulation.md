---
title: "EKOBOT: **field simulation**"
sub_title: ROS 2, Isaac Sim, and synthetic agricultural data
author: Trim Bresilla & Anouk Leunissen
date: 2026-09-08
theme:
  name: dark
  override:
    default:
      colors:
        foreground: "e8edf5"
        background: "03060b"
    palette:
      colors:
        accent: "4f78a6"
        chromatic: "c65d21"
    slide_title:
      colors:
        foreground: "4f78a6"
options:
  incremental_lists: false
---

Why this simulation effort
==========================

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

![image:width:100%](ekobot-simulation/ekobot-current.jpg)

<!-- column: 1 -->

* Move the robot software from ROS 1 to ROS 2 without stopping field development.
* Test weed-removal behaviour before hardware and crop rows are available.
* Use simulation to compare tool geometry, sensing, and control changes.
* Produce labelled field imagery for perception experiments.

<!-- reset_layout -->

The aim is a repeatable engineering environment, not a photorealistic demo.

<!-- speaker_note: EKOBOT has Swedish engineering roots and a current Dutch ownership connection. The robot combines an autonomous electric carrier, cameras, an AI perception system, and a mechanical weeding tool. This presentation proposes where ROS 2 and simulation can help, while keeping field tests as the final acceptance test. Photo: Nieuwe Oogst, 14 May 2025. -->

<!-- end_slide -->

The EKOBOT system
=================

```faqe:graph
title = "One field operation, several coupled systems"
columns = 4
rows = 2
[[nodes]]
id = "crop"
title = "crop row"
subtitle = "plants · weeds · soil"
column = 1
row = 1
tone = "positive"
[[nodes]]
id = "sense"
title = "camera system"
subtitle = "3D view · localisation"
column = 2
row = 1
tone = "accent"
[[nodes]]
id = "decide"
title = "perception"
subtitle = "plant class · position"
column = 3
row = 1
[[nodes]]
id = "tool"
title = "weeding tool"
subtitle = "timing · path · force"
column = 4
row = 1
tone = "warning"
[[nodes]]
id = "carrier"
title = "electric carrier"
subtitle = "navigation · safety"
column = 2
row = 2
[[nodes]]
id = "record"
title = "field record"
subtitle = "events · outcomes"
column = 3
row = 2
[[edges]]
from = "crop"
to = "sense"
[[edges]]
from = "sense"
to = "decide"
[[edges]]
from = "decide"
to = "tool"
[[edges]]
from = "carrier"
to = "sense"
[[edges]]
from = "tool"
to = "record"
[[edges]]
from = "carrier"
to = "record"
```

Tool accuracy depends on the complete chain, including vehicle motion and timestamp alignment.

<!-- speaker_note: EKOBOT's public material describes three main subsystems: the autonomous carrier, the mechanical tool, and the AI/camera system. For simulation work, it is useful to add the field state and the recorded outcome. A small error in any upstream transform or timestamp can become a crop strike at the tool. -->

<!-- end_slide -->

What must be proved
===================

```faqe:table
variant = "comparison"
columns = ["Question", "Measure", "Final evidence"]
[[rows]]
cells = ["Did we remove the weed?", "removal rate by class", "labelled field plot"]
[[rows]]
cells = ["Did we protect the crop?", "crop contacts and damage", "post-pass inspection"]
[[rows]]
cells = ["Did the tool track its target?", "tip error and timing", "high-speed video"]
[[rows]]
cells = ["Can the system keep working?", "misses, stops, recovery", "full-row run"]
```

Simulation supplies controlled trials and diagnostics. Field measurements decide whether the model was useful.

<!-- speaker_note: Agree on measurements before building scenes. A visually convincing simulator can still omit the error source that limits field performance. The acceptance set should separate weed removal, crop protection, geometric tracking, and operational recovery. -->

<!-- end_slide -->

ROS 2 as the experiment boundary
================================

* Keep message and service contracts shared between simulation and the robot.
* Put simulator adapters at hardware boundaries rather than inside application nodes.
* Record commands, observations, transforms, and outcomes in the same bag format.
* Run the same launch descriptions and parameter sets where practical.
* Version the scene, robot asset, calibration, software, and random seed together.

ROS 2 is most useful here as a replaceable boundary between product software and its environment.

<!-- speaker_note: The migration goes beyond API translation. It gives us a chance to define which interfaces remain stable when cameras, drives, and tools are replaced by simulated counterparts. Keep simulation-specific code in drivers and adapters so application nodes can run unchanged. -->

<!-- end_slide -->

The simulation graph
====================

```faqe:graph
title = "ROS 2 software in the loop"
columns = 4
rows = 2
[[nodes]]
id = "world"
title = "Isaac Sim"
subtitle = "USD · PhysX · sensors"
column = 1
row = 1
tone = "accent"
[[nodes]]
id = "bridge"
title = "ROS 2 bridge"
subtitle = "topics · services · clock"
column = 2
row = 1
[[nodes]]
id = "stack"
title = "robot stack"
subtitle = "perception · control"
column = 3
row = 1
[[nodes]]
id = "act"
title = "actuator commands"
subtitle = "carrier · tool"
column = 4
row = 1
tone = "warning"
[[nodes]]
id = "bag"
title = "rosbag2"
subtitle = "replay · comparison"
column = 2
row = 2
[[nodes]]
id = "score"
title = "evaluator"
subtitle = "events · metrics"
column = 4
row = 2
tone = "positive"
[[edges]]
from = "world"
to = "bridge"
[[edges]]
from = "bridge"
to = "stack"
[[edges]]
from = "stack"
to = "act"
[[edges]]
from = "act"
to = "world"
[[edges]]
from = "bridge"
to = "bag"
[[edges]]
from = "world"
to = "score"
[[edges]]
from = "act"
to = "score"
```

<!-- speaker_note: Isaac Sim's ROS 2 bridge exposes publishers, subscribers, services, and clock integration through its graph system. Bridge nodes are active while simulation is playing. Keep a separate evaluator connected to simulator ground truth, because product code must not see perfect labels. -->

<!-- end_slide -->

Time, QoS, and repeatability
============================

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "time"
title = "Use simulated clock"
bullets = ["publish /clock", "set use_sim_time", "timestamp every observation", "test resets and pauses"]
tone = "accent"
[[items]]
eyebrow = "delivery"
title = "Match the real interfaces"
bullets = ["sensor-data QoS", "bounded queues", "late subscriber behaviour", "intentional loss tests"]
tone = "warning"
[[items]]
eyebrow = "replay"
title = "Capture the run"
bullets = ["scene revision", "random seed", "parameter files", "software commit"]
tone = "positive"
[[items]]
eyebrow = "speed"
title = "Measure RTF"
bullets = ["physics rate", "render rate", "ROS callback load", "GPU saturation"]
```

Real-time factor is simulated elapsed time divided by wall-clock elapsed time.

<!-- speaker_note: A repeatable run needs more than a random seed. Record the complete set of inputs that can change an outcome. Sensor QoS should resemble the deployed system, otherwise an application may pass only because the simulator delivered data more reliably. Track real-time factor separately from correctness. -->

<!-- end_slide -->

One test ladder
===============

```faqe:timeline
[[items]]
title = "node tests"
body = "messages · transforms · algorithms"
[[items]]
title = "headless SIL"
body = "fixed scenes · regression metrics"
tone = "accent"
[[items]]
title = "rendered SIL"
body = "camera models · perception"
[[items]]
title = "hardware loop"
body = "controller · latency · I/O"
tone = "warning"
[[items]]
title = "field plot"
body = "soil · plants · weather · wear"
tone = "positive"
```

Move a failure downward only when the cheaper level cannot represent its cause.

<!-- speaker_note: This is not a maturity staircase where every test must use the most expensive level. Geometry and timing regressions should remain in fast software-in-the-loop tests. Hardware-loop and field tests cover effects the virtual model does not reproduce. -->

<!-- end_slide -->

Why Isaac Sim
=============

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "assets"
title = "OpenUSD scene"
body = "Layer robot, field, materials, sensors, and experiment variants without duplicating the base asset."
tone = "accent"
[[items]]
eyebrow = "motion"
title = "Physics stepping"
body = "Articulations, contacts, joint drives, and sensor timing share the simulation clock."
[[items]]
eyebrow = "sensing"
title = "Rendered and physics sensors"
body = "Cameras, depth, IMU, contact, effort, and joint state can be generated together."
tone = "positive"
[[items]]
eyebrow = "data"
title = "Replicator"
body = "Randomize scenes and write synchronized images, labels, poses, and metadata."
tone = "warning"
```

Isaac Sim is the scene and sensor runtime. ROS 2 carries the product interfaces.

<!-- speaker_note: OpenUSD is useful because the vehicle, tool, field, materials, and experiment changes can be separate layers. Isaac Sim currently provides a ROS 2 bridge, URDF import, physics, sensors, and Replicator data generation. Version-pin the simulator because extensions and APIs change between releases. -->

<!-- end_slide -->

From URDF to a checked USD asset
================================

```faqe:progress
title = "asset acceptance"
max = 1.0

[[items]]
label = "checked"
value = 0.82
display = "6 / 7"
text = "import · axes · joints · collisions · mass · drives · sensors"
tone = "accent"
```

1. Import links, joints, meshes, and limits from URDF.
2. Check scale, axes, joint directions, and frame names.
3. Replace visual meshes with stable collision geometry.
4. Measure mass, centre of mass, inertia, limits, and actuator response.
5. Add camera and tool frames only after the mechanism is correct.

The imported asset is a starting point. It is not yet a digital twin.

<!-- speaker_note: NVIDIA's URDF importer converts the robot into USD and can create an articulation. Imported values still need review. Collision meshes, joint drives, materials, mass properties, and frame conventions are common sources of believable but incorrect motion. Keep a short asset acceptance test that runs after every model change. -->

<!-- end_slide -->

Split the carrier from the tool
===============================

```faqe:graph
title = "Composed USD assets"
columns = 3
rows = 2
[[nodes]]
id = "carrier"
title = "carrier.usd"
subtitle = "wheels · chassis · safety"
column = 1
row = 1
tone = "accent"
[[nodes]]
id = "mount"
title = "tool mount"
subtitle = "datum · interface · compliance"
column = 2
row = 1
[[nodes]]
id = "tool"
title = "tool variant"
subtitle = "geometry · joints · blades"
column = 3
row = 1
tone = "warning"
[[nodes]]
id = "sensors"
title = "sensor rig"
subtitle = "camera · IMU · encoders"
column = 1
row = 2
[[nodes]]
id = "field"
title = "field scene"
subtitle = "rows · plants · terrain"
column = 3
row = 2
tone = "positive"
[[edges]]
from = "carrier"
to = "mount"
[[edges]]
from = "mount"
to = "tool"
[[edges]]
from = "sensors"
to = "carrier"
[[edges]]
from = "field"
to = "tool"
```

A stable mount contract lets tool variants change without rebuilding the carrier model.

<!-- speaker_note: The composition boundary should follow the physical and software boundary. Define the mount transform, connection points, expected loads, I/O, and calibration procedure. This makes it possible to compare tools and sensor rigs against the same vehicle and field run. -->

<!-- end_slide -->

Model sensors as measurements
=============================

```faqe:table
variant = "comparison"
columns = ["Sensor", "Start with", "Add after comparison"]
[[rows]]
cells = ["RGB camera", "intrinsics, pose, exposure", "blur, glare, rolling shutter"]
[[rows]]
cells = ["Depth", "range, occlusion, alignment", "dropout, bias, quantisation"]
[[rows]]
cells = ["Encoders", "joint state and rate", "latency, noise, slip"]
[[rows]]
cells = ["IMU", "pose and dynamics", "bias, drift, vibration"]
[[rows]]
cells = ["Tool contact", "contact and effort", "compliance, wear, soil effects"]
```

Noise parameters should come from paired measurements, not visual judgement.

<!-- speaker_note: Isaac Sim can provide ideal outputs and post-processed noise. Begin with calibrated geometry and timing. Add noise only after comparing virtual and physical samples, otherwise noise can hide a model error. Product nodes should receive the same ROS message types and frame conventions in both environments. -->

<!-- end_slide -->

The tool is a timed mechanism
=============================

```faqe:graph
title = "From detected plant to tool contact"
columns = 5
rows = 1
[[nodes]]
id = "detect"
title = "detect"
subtitle = "class · pixel"
column = 1
row = 1
tone = "accent"
[[nodes]]
id = "locate"
title = "locate"
subtitle = "3D · field frame"
column = 2
row = 1
[[nodes]]
id = "predict"
title = "predict"
subtitle = "vehicle motion"
column = 3
row = 1
[[nodes]]
id = "command"
title = "command"
subtitle = "joint trajectory"
column = 4
row = 1
[[nodes]]
id = "contact"
title = "contact"
subtitle = "tip · soil · plant"
column = 5
row = 1
tone = "warning"
[[edges]]
from = "detect"
to = "locate"
label = "camera time"
[[edges]]
from = "locate"
to = "predict"
[[edges]]
from = "predict"
to = "command"
[[edges]]
from = "command"
to = "contact"
label = "actuation delay"
```

Measure latency and tool-tip error at each boundary instead of scoring only the final contact.

<!-- speaker_note: A mechanical tool can miss even with correct plant classification. Camera timestamps, transform lookup, vehicle velocity estimation, trajectory generation, actuator delay, compliance, and soil resistance all contribute to the contact point. Simulation gives access to every intermediate value. -->

<!-- end_slide -->

A tool-design test matrix
=========================

```faqe:table
variant = "comparison"
columns = ["Variable", "Range", "Measure"]
[[rows]]
cells = ["forward speed", "slow to operating limit", "tip error, throughput"]
[[rows]]
cells = ["plant spacing", "isolated to clustered", "crop clearance"]
[[rows]]
cells = ["mount offset", "tolerance and drift", "calibration sensitivity"]
[[rows]]
cells = ["actuator delay", "nominal plus tails", "late contacts"]
[[rows]]
cells = ["tool geometry", "candidate variants", "coverage and clearance"]
[[rows]]
cells = ["terrain profile", "flat to bounded roughness", "height tracking"]
```

Run the same scenario set for each tool revision and keep the raw event log.

<!-- speaker_note: Start with variables whose values can be measured on the physical robot. Use a designed set of combinations rather than randomizing everything at once. The result should rank the designs and explain why their performance differs. -->

<!-- end_slide -->

Where soil models stop helping
==============================

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "reasonable first model"
title = "Geometry and contact"
bullets = ["tool clearance", "collision timing", "height following", "kinematic reach"]
tone = "positive"
[[items]]
eyebrow = "needs field calibration"
title = "Resistance and disturbance"
bullets = ["compaction", "moisture", "roots", "soil flow", "tool wear"]
tone = "warning"
```

Use contact simulation for geometry and timing first. Treat agronomic outcome as a measured field property.

<!-- speaker_note: Rigid-body contact can answer useful design questions, but it does not reproduce every soil and root interaction. A more detailed soil model should be added only when a decision depends on it and physical data is available for calibration. Crop damage and successful root removal still need field evidence. -->

<!-- end_slide -->

Software-in-the-loop boundaries
===============================

```faqe:graph
title = "Product code stays outside the simulator"
columns = 4
rows = 2
[[nodes]]
id = "sim"
title = "Isaac Sim"
subtitle = "world · sensors · physics"
column = 1
row = 1
tone = "accent"
[[nodes]]
id = "drivers"
title = "sim adapters"
subtitle = "ROS messages · commands"
column = 2
row = 1
[[nodes]]
id = "product"
title = "product nodes"
subtitle = "same executables"
column = 3
row = 1
tone = "positive"
[[nodes]]
id = "supervisor"
title = "supervisor"
subtitle = "start · stop · recovery"
column = 4
row = 1
[[nodes]]
id = "truth"
title = "ground truth"
subtitle = "private evaluation path"
column = 2
row = 2
tone = "warning"
[[nodes]]
id = "report"
title = "test report"
subtitle = "metrics · traces · video"
column = 4
row = 2
[[edges]]
from = "sim"
to = "drivers"
[[edges]]
from = "drivers"
to = "product"
[[edges]]
from = "product"
to = "supervisor"
[[edges]]
from = "sim"
to = "truth"
[[edges]]
from = "truth"
to = "report"
[[edges]]
from = "supervisor"
to = "report"
```

<!-- speaker_note: Avoid importing simulator APIs into the production perception and control nodes. A simulation adapter publishes the same camera, depth, transform, joint, and status interfaces as the real drivers. The evaluator may use perfect simulator state, but that channel must be inaccessible to product logic. -->

<!-- end_slide -->

Build a scenario catalogue
==========================

```faqe:grid
columns = 3
variant = "cards"
[[items]]
eyebrow = "crop"
title = "Plant state"
bullets = ["species", "growth stage", "spacing", "occlusion", "disease"]
tone = "positive"
[[items]]
eyebrow = "field"
title = "Operating state"
bullets = ["row geometry", "soil colour", "roughness", "residue", "moisture proxy"]
[[items]]
eyebrow = "light"
title = "Image conditions"
bullets = ["sun angle", "cloud cover", "exposure", "shadows", "specular leaves"]
tone = "accent"
[[items]]
eyebrow = "robot"
title = "Motion"
bullets = ["speed", "yaw error", "vibration", "wheel slip", "tool delay"]
tone = "warning"
[[items]]
eyebrow = "fault"
title = "Degradation"
bullets = ["camera loss", "stale transform", "blocked joint", "missed deadline"]
[[items]]
eyebrow = "record"
title = "Identity"
bullets = ["scenario ID", "asset version", "seed", "software commit"]
```

Every run should be reproducible from one manifest.

<!-- speaker_note: A scenario is more than a scene file. It includes robot state, environmental parameters, injected faults, software versions, and expected measurements. Keep named regression scenarios fixed, then add randomized sweeps around them. -->

<!-- end_slide -->

Data products from one run
==========================

```faqe:table
variant = "comparison"
columns = ["Product", "Used by", "Content"]
[[rows]]
cells = ["rosbag2", "software debugging", "sensor topics, commands, TF, events"]
[[rows]]
cells = ["ground truth", "evaluation", "plant IDs, poses, contacts, trajectories"]
[[rows]]
cells = ["render output", "perception", "RGB, depth, masks, boxes"]
[[rows]]
cells = ["run manifest", "reproduction", "versions, seed, parameters, scene"]
[[rows]]
cells = ["summary", "review", "metrics, failures, selected frames"]
```

Synchronised records make it possible to trace a damaged crop back through detection, timing, and contact.

<!-- speaker_note: Separate the data that production code consumes from the ground truth used for evaluation. Preserve raw outputs before reducing them to summary metrics. A failed run should be inspectable without launching the original simulator version. -->

<!-- end_slide -->

Replicator and Cosmos do different work
=======================================

```faqe:table
variant = "comparison"
columns = ["Aspect", "Isaac Replicator", "Cosmos Transfer"]
[[rows]]
cells = ["Input", "structured USD scene", "video plus control signal"]
[[rows]]
cells = ["Output", "render and exact labels", "transformed photorealistic video"]
[[rows]]
cells = ["Strength", "geometry and annotation", "appearance and domain variation"]
[[rows]]
cells = ["Constraint", "asset realism", "control adherence and training domain"]
[[rows]]
cells = ["EKOBOT use", "test and labelled source data", "appearance transfer for perception"]
```

Keep exact simulator labels when Cosmos changes the appearance of the corresponding frames.

<!-- speaker_note: Replicator can randomize a known scene and write synchronized annotations. Cosmos Transfer changes the visual domain using a source video and a structural control such as depth. The useful combination is structured source generation followed by appearance transfer, with explicit checks that labels remain valid. -->

<!-- end_slide -->

The agricultural Cosmos recipe
==============================

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

![image:width:100%](ekobot-simulation/cosmos-soybean.gif)

<!-- column: 1 -->

* NVIDIA's example post-trains Cosmos Transfer 2.5 on agricultural robot video.
* It uses depth as the structural control for single-view video transfer.
* The training set covers soybean, cotton, and tomato fields.
* The reported downstream experiment combines mostly synthetic imagery with 1% real data.

<!-- reset_layout -->

Treat this as a reproducible case study. The cookbook now receives limited maintenance, so pin every dependency.

<!-- speaker_note: The recipe reports autonomous field weeding using a perception model trained with one percent real data and the remainder synthetic. That is the recipe authors' result, not a guaranteed EKOBOT outcome. The current Cosmos Cookbook repository points future work toward Cosmos 3, so we should reproduce the Transfer 2.5 baseline before considering a model-family update. -->

<!-- end_slide -->

The agricultural domain gap
===========================

```faqe:graph
title = "Depth preserves structure while appearance changes"
columns = 4
rows = 2
[[nodes]]
id = "usd"
title = "3D field"
subtitle = "plants · robot · camera"
column = 1
row = 1
tone = "accent"
[[nodes]]
id = "render"
title = "source render"
subtitle = "RGB + exact labels"
column = 2
row = 1
[[nodes]]
id = "depth"
title = "depth control"
subtitle = "normalised consistently"
column = 2
row = 2
tone = "warning"
[[nodes]]
id = "transfer"
title = "post-trained transfer"
subtitle = "crop and field appearance"
column = 3
row = 1
[[nodes]]
id = "data"
title = "training sample"
subtitle = "realistic RGB + labels"
column = 4
row = 1
tone = "positive"
[[nodes]]
id = "audit"
title = "label audit"
subtitle = "alignment · artifacts"
column = 4
row = 2
[[edges]]
from = "usd"
to = "render"
[[edges]]
from = "usd"
to = "depth"
[[edges]]
from = "render"
to = "transfer"
[[edges]]
from = "depth"
to = "transfer"
[[edges]]
from = "transfer"
to = "data"
[[edges]]
from = "data"
to = "audit"
```

<!-- speaker_note: Appearance transfer is useful only if the transformed plant boundaries and positions remain compatible with the simulator labels. Depth provides structural guidance, but the output still needs an automated and sampled human audit for geometry drift, duplicated objects, and temporal artifacts. -->

<!-- end_slide -->

Why zero-shot transfer was insufficient
=======================================

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

**3D source render**

![image:width:100%](ekobot-simulation/image3drender.webp)

<!-- column: 1 -->

**Base Transfer 2.5 checkpoint**

![image:width:100%](ekobot-simulation/BaseCosmosTransfer25Checkpoint.webp)

<!-- reset_layout -->

The recipe reports generic vegetation, weak crop morphology, and implausible field texture before agricultural post-training.

Post-training teaches the model the target visual domain; depth keeps the scene arrangement constrained.

<!-- speaker_note: This comparison comes from the NVIDIA recipe. The base checkpoint can improve visual realism without knowing enough about row crops, leaves, weeds, and tilled soil. The point of post-training is domain adaptation, not changing the robot trajectory or plant layout. -->

<!-- end_slide -->

The source data
===============

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

[Agricultural fleet clip](ekobot-simulation/aigen_robot_cotton.mp4)

<!-- column: 1 -->

```faqe:grid
columns = 1
variant = "cards"
[[items]]
eyebrow = "volume"
title = "About 3,000 clips"
body = "Roughly ten seconds each from an agricultural robot fleet."
tone = "accent"
[[items]]
eyebrow = "training format"
title = "480p at 10 FPS"
body = "Original 720p clips were prepared for the training configuration."
[[items]]
eyebrow = "coverage"
title = "Three crop families"
body = "Soybean, cotton, and tomato across field states and views."
tone = "positive"
```

<!-- reset_layout -->

<!-- speaker_note: Fleet video provides real agricultural appearance and motion. For EKOBOT, the equivalent collection should include the deployed camera pose, relevant crops, working speeds, lighting changes, tool states, and known hard negatives. Consent, ownership, and retention rules belong in the collection plan. -->

<!-- end_slide -->

Each clip needs searchable context
==================================

```faqe:table
variant = "comparison"
columns = ["Field", "Role", "EKOBOT example"]
[[rows]]
cells = ["sample_id", "stable identity", "field-04_row-12_clip-008"]
[[rows]]
cells = ["video_path", "RGB sequence", "video/clip-008.mp4"]
[[rows]]
cells = ["depth_path", "structural control", "depth/clip-008.mp4"]
[[rows]]
cells = ["crop_type", "domain filter", "yellow onion"]
[[rows]]
cells = ["field_state", "visual condition", "emerged, damp soil"]
[[rows]]
cells = ["camera_view", "geometry filter", "tool-facing oblique"]
[[rows]]
cells = ["weather", "optional context", "bright broken cloud"]
```

Captions should state camera angle, crop, field state, and visible conditions in consistent language.

<!-- speaker_note: The NVIDIA recipe requires sample identity, video, and depth paths, and recommends crop type, field state, and camera view. Weather is optional. Their captioning pipeline used a structured template and Gemini 2.5 Flash for visible-scene description. Store generated captions with the model and prompt revision so they can be audited. -->

<!-- end_slide -->

Depth control worked better than edges
======================================

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

![image:width:100%](ekobot-simulation/deepcontrol.webp)

<!-- column: 1 -->

```faqe:grid
columns = 1
variant = "cards"
[[items]]
eyebrow = "edge failure"
title = "Illumination becomes structure"
body = "Hard shadows, exposure changes, and wet-leaf highlights can create false boundaries."
tone = "warning"
[[items]]
eyebrow = "depth advantage"
title = "Geometry remains explicit"
body = "Depth is less sensitive to illumination and better preserves rows and plant placement."
tone = "accent"
[[items]]
eyebrow = "requirement"
title = "One normalisation rule"
body = "Training and inference must map depth values in exactly the same way."
tone = "positive"
```

<!-- reset_layout -->

<!-- speaker_note: This is the recipe's agricultural finding, not a universal rule for every task. Synchronized measured or simulated depth is preferred. The recipe notes that Depth Anything V2 can estimate it when synchronized depth is unavailable. Check the error pattern before using estimated depth as a control signal. -->

<!-- end_slide -->

The reported training setup
===========================

```faqe:table
variant = "comparison"
columns = ["Setting", "Recipe value", "Reason given"]
[[rows]]
cells = ["hardware", "8 × A100 GPUs", "distributed post-training"]
[[rows]]
cells = ["iterations", "4,000", "agricultural domain adaptation"]
[[rows]]
cells = ["checkpoint interval", "500", "visual comparison over time"]
[[rows]]
cells = ["state_t", "16", "low temporal variability"]
[[rows]]
cells = ["video frames", "61", "short field clips"]
[[rows]]
cells = ["inference depth guidance", "0.8", "reported control strength"]
```

The loss flattened before visual quality stopped changing, so checkpoint selection included visual review.

<!-- speaker_note: These are reproduction values from the Transfer 2.5 recipe, not defaults for a new model or dataset. Record exact software, container, weights, and GPU configuration. The recipe warns that training loss alone did not identify the best visual checkpoint. Use a fixed validation set and a written image-quality rubric. -->

<!-- end_slide -->

Condition changes without new 3D assets
=======================================

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

**Healthy and bright**

![image:width:100%](ekobot-simulation/healthly-bright.webp)

<!-- column: 1 -->

**Prompted disease condition**

![image:width:100%](ekobot-simulation/Diseased.webp)

<!-- reset_layout -->

The recipe reports prompt-driven disease appearance even though disease-specific clips were not in its training set.

For EKOBOT, such samples belong in a stress-test set until field data confirms that they improve detection.

<!-- speaker_note: Generative variation can expose a perception model to conditions missing from the simulator assets. It can also invent symptoms, alter boundaries, or create unrealistic correlations. Keep generated disease conditions separate from validated field data and review them with agronomy expertise. -->

<!-- end_slide -->

What post-training changed
==========================

<!-- column_layout: [1, 1, 1, 1] -->
<!-- column: 0 -->

**Base**

![image:width:100%](ekobot-simulation/BaseCosmosTransfer25Checkpoint.webp)

<!-- column: 1 -->

**Post-trained**

![image:width:100%](ekobot-simulation/Aigenpost-trainedCosmosTransfer25.webp)

<!-- column: 2 -->

**Baseline pair**

![image:width:100%](ekobot-simulation/Baselinesim2real.webp)

<!-- column: 3 -->

**Adapted pair**

![image:width:100%](ekobot-simulation/aigen-posttrained.webp)

<!-- reset_layout -->

The recipe reports stronger crop morphology, tilled soil, shadows, weed diversity, and depth adherence after post-training.

Some transfer to unseen crops was reported, but it was less consistent than performance in the training domain.

<!-- speaker_note: Review these images at full size in the source recipe. For our evaluation, image preference is not enough. Compare downstream segmentation or detection metrics across real-only, synthetic-only, and mixed datasets, then inspect failure categories. -->

<!-- end_slide -->

An EKOBOT data loop
===================

```faqe:timeline
[[items]]
title = "capture"
body = "paired field video, depth, calibration, outcomes"
[[items]]
title = "simulate"
body = "tool motion, plant geometry, exact labels"
tone = "accent"
[[items]]
title = "transfer"
body = "field appearance under depth control"
tone = "warning"
[[items]]
title = "train"
body = "fixed real and synthetic data ratios"
[[items]]
title = "measure"
body = "offline metrics and field-row outcomes"
tone = "positive"
```

```faqe:progress
title = "first study"
max = 1.0

[[items]]
label = "starting point"
value = 0.2
display = "scope"
text = "contracts · asset · scenarios · paired data · baseline · transfer · field test"
tone = "warning"
```

Start with one crop, one camera view, one tool configuration, and a held-out physical field plot.

<!-- speaker_note: The first study should be small enough to invalidate quickly. Establish a real-only perception baseline, then add controlled synthetic ratios. Use the same held-out physical dataset for comparison. Separately run the ROS 2 tool-control stack in fixed simulator scenarios so perception gains are not confused with control changes. -->

<!-- end_slide -->

Decisions for the first phase
=============================

1. Select the ROS 2 distribution and define the ROS 1 coexistence boundary.
2. Choose one EKOBOT carrier, tool, crop, camera view, and measurable field task.
3. Build the asset acceptance test and ten fixed software-in-the-loop scenarios.
4. Record paired field data with calibration, depth, operating state, and outcome labels.
5. Reproduce the published Transfer 2.5 recipe before testing a newer Cosmos model.

**Primary references**

[EKOBOT](https://www.ekobot.se/) · [RISE profile](https://www.ri.se/en/agriculture/story/ekobots-unique-robot-contributes-to-sustainable-agriculture) · [2024 ownership announcement](https://view.news.eu.nasdaq.com/view?id=b092d6b2e3f307646230f3910732a4871&lang=en&src=micro) · [2025 photo](https://www.nieuweoogst.nl/nieuws/2025/05/14/ekobot-gaat-wieden-met-lasers-van-escarda)

[Isaac Sim ROS 2](https://docs.isaacsim.omniverse.nvidia.com/latest/ros2_tutorials/ros2_landing_page.html) · [URDF import](https://docs.isaacsim.omniverse.nvidia.com/latest/importer_exporter/import_urdf.html) · [Replicator](https://docs.isaacsim.omniverse.nvidia.com/latest/replicator_tutorials/tutorial_replicator_isaac_randomizers.html) · [Cosmos recipe](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/post_training/transfer2_5/agtec_scenarios_single_view/post_training.html)

<!-- speaker_note: Additional sources: the RISE profile of EKOBOT's EIP-Agri development and tool evaluation; the 3 June 2024 Nasdaq announcement confirming the sale to HH Agriculture Investments B.V.; Isaac Sim documentation for sensors, physics, the ROS 2 bridge, synthetic-data recording, and real-time factor; and the Cosmos Cookbook repository notice about limited maintenance. End by agreeing on the first measurable task and who owns the robot asset, field data, ROS interfaces, and validation set. -->
