---
title: "ROS 1 → **ROS 2**"
sub_title: A hands-on migration workshop for a ROS 1 team, with Rust and Zenoh where they earn their place
author: Trim Bresilla & Anouk Leunissen
date: 2026-09-03
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

What we need from today
=======================

By the end, we should be able to answer four questions for **your** system:

1. What can be ported mechanically?
2. What must be redesigned because ROS 2 changes the contract?
3. How do we migrate without stopping product delivery?
4. Which new capabilities earn a place now, and which can wait?

<!-- speaker_note: This is a working session for a team that already ships ROS 1. We are here to make migration decisions, not tour every ROS 2 feature. -->

<!-- end_slide -->

Why move now?
=============

```faqe:grid
columns = 3
variant = "metrics"

[[items]]
eyebrow = "ROS Noetic"
title = "EOL"
body = "Official ROS 1 support ended on 31 May 2025."
tone = "negative"

[[items]]
eyebrow = "after EOL"
title = "0"
body = "No new features, security fixes, official patches, or updated binaries."
tone = "warning"

[[items]]
eyebrow = "migration reality"
title = "now"
body = "Old binaries remain, but the risk and maintenance burden move to you."
tone = "accent"
```

> ROS 1 does not suddenly stop running. It becomes **your distribution to maintain**.

<!-- speaker_note: Existing robots will keep booting. What changed is ownership: security work, dependency drift, compiler trouble, and unsupported hardware now land on your team. -->

<!-- end_slide -->

Choose the target deliberately
==============================

```faqe:table
title = "Supported ROS 2 distributions in September 2026"
variant = "comparison"
columns = ["Distribution", "Base platform", "Upstream EOL", "Workshop recommendation"]

[[rows]]
cells = ["Jazzy Jalisco", "Ubuntu 24.04", "May 2029", "Mature LTS when Noble is your platform"]
tones = ["positive", "neutral", "neutral", "positive"]

[[rows]]
cells = ["Kilted Kaiju", "Ubuntu 24.04", "December 2026", "Do not begin a long migration here"]
tones = ["warning", "neutral", "negative", "negative"]

[[rows]]
cells = ["Lyrical Luth", "Ubuntu 26.04", "May 2031", "New LTS for a new Resolute platform"]
tones = ["accent", "neutral", "positive", "accent"]

[[rows]]
cells = ["Rolling Ridley", "moving target", "continuous", "Development and upstream contribution only"]
tones = ["warning", "warning", "warning", "neutral"]
```

Choose around OS certification, vendor drivers, and the required support window. “Newest” is not a deployment requirement.

<!-- speaker_note: Freeze the target distribution early. Jazzy remains an excellent migration target on Ubuntu 24.04. Lyrical is the new long-lived target on Ubuntu 26.04. Validate vendor support before choosing. -->

<!-- end_slide -->

How we will spend the day
=========================

```faqe:timeline
[[items]]
title = "Part I · change the mental model"
meta = "graph · middleware · QoS"
body = "Separate architectural changes from simple renames."
tone = "accent"

[[items]]
title = "Part II · port the application"
meta = "APIs · build · launch · lifecycle"
body = "Translate packages while adopting ROS 2 execution semantics."
tone = "positive"

[[items]]
title = "Part III · migrate safely"
meta = "bridge · test · deploy"
body = "Run mixed systems, measure parity, and retire ROS 1 in slices."
tone = "warning"

[[items]]
title = "Part IV · go further"
meta = "Rust · Zenoh · world models"
body = "Use ROS 2 as a platform once the migration basics are sound."
tone = "negative"

[[items]]
title = "Part V · workshop plan"
meta = "labs · backlog · 90 days"
body = "Leave with concrete owners, gates, and first migrations."
tone = "accent"
```

<!-- speaker_note: The first half is the migration core. Rust, Zenoh, and world models are optional, though knowing about them now may change where we draw boundaries. -->

<!-- end_slide -->

Keep the robot working
======================

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "keep"
title = "Domain logic"
bullets = ["algorithms", "calibration", "kinematics", "state machines", "hardware protocols", "test vectors"]
body = "Move valuable logic behind clean ROS-neutral boundaries."
tone = "positive"

[[items]]
eyebrow = "reconsider"
title = "System contracts"
bullets = ["delivery semantics", "discovery", "startup", "configuration", "concurrency", "failure recovery"]
body = "These are where ROS 2 is genuinely different."
tone = "accent"
```

Port the behavior you already trust. Redesign only when ROS 2 offers a better contract or the old assumption no longer holds.

<!-- speaker_note: A big-bang rewrite piles three risks together: framework, behavior, and architecture. Keep the algorithms and test vectors steady while you replace the integration layer. -->

<!-- end_slide -->

The ROS 1 mental model
======================

```faqe:graph
title = "Central coordination, mostly point-to-point transport"
columns = 5
rows = 3

[[nodes]]
id = "master"
title = "ROS master"
subtitle = "names · registration · lookup"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "param"
title = "parameter server"
subtitle = "global configuration tree"
column = 3
row = 2
tone = "accent"

[[nodes]]
id = "a"
title = "node A"
subtitle = "publisher"
column = 1
row = 3
tone = "positive"

[[nodes]]
id = "b"
title = "node B"
subtitle = "subscriber"
column = 3
row = 3
tone = "positive"

[[nodes]]
id = "c"
title = "node C"
subtitle = "service"
column = 5
row = 3
tone = "neutral"

[[edges]]
from = "a"
to = "master"
label = "register"
dashed = true

[[edges]]
from = "b"
to = "master"
label = "lookup"
dashed = true

[[edges]]
from = "master"
to = "c"
label = "resolve"
dashed = true

[[edges]]
from = "a"
to = "b"
label = "TCPROS / UDPROS"
tone = "positive"

[[edges]]
from = "param"
to = "b"
label = "global read"
tone = "accent"
```

The master coordinates discovery; it is not normally on the topic data path.

<!-- speaker_note: Be precise about ROS 1. The master is a central control-plane dependency, not a message broker. Nodes negotiate direct transport after lookup. -->

<!-- end_slide -->

The ROS 2 mental model
======================

```faqe:graph
title = "A distributed graph over a replaceable middleware layer"
columns = 5
rows = 3

[[nodes]]
id = "a"
title = "node A"
subtitle = "publisher · client"
column = 1
row = 1
tone = "positive"

[[nodes]]
id = "b"
title = "node B"
subtitle = "subscriber · server"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "c"
title = "node C"
subtitle = "action server"
column = 5
row = 1
tone = "neutral"

[[nodes]]
id = "rmw"
title = "RMW"
subtitle = "middleware abstraction"
column = 3
row = 2
tone = "accent"

[[nodes]]
id = "transport"
title = "DDS or Zenoh"
subtitle = "discovery · serialization · transport"
column = 3
row = 3
tone = "warning"

[[edges]]
from = "a"
to = "b"
label = "QoS contract"
tone = "positive"

[[edges]]
from = "b"
to = "c"
label = "distributed graph"

[[edges]]
from = "a"
to = "rmw"

[[edges]]
from = "b"
to = "rmw"

[[edges]]
from = "c"
to = "rmw"

[[edges]]
from = "rmw"
to = "transport"
label = "runtime selection"
tone = "accent"
```

<!-- speaker_note: There is no ROS master. The selected RMW distributes discovery and graph state. We lose one central dependency and gain a middleware configuration problem that operations must understand. -->

<!-- end_slide -->

No master does not mean no infrastructure
=========================================

```faqe:table
title = "Control-plane responsibilities move"
variant = "comparison"
columns = ["Concern", "ROS 1 default", "ROS 2 default", "Production choice"]

[[rows]]
cells = ["Discovery", "ROS master", "distributed middleware discovery", "multicast, discovery server, or Zenoh router"]

[[rows]]
cells = ["Parameters", "global server", "per-node services", "configuration files plus node ownership"]

[[rows]]
cells = ["Process placement", "roslaunch machine tags", "launch and external orchestration", "systemd, containers, Kubernetes, or fleet manager"]

[[rows]]
cells = ["Security", "network trust", "middleware security and enclaves", "identity, permissions, keys, and rotation"]
```

Distributed defaults still need an explicit production topology.

<!-- speaker_note: ROS 2 did not make infrastructure disappear. Pick a discovery setup, process supervisor, and security policy instead of inheriting accidental defaults. -->

<!-- end_slide -->

ROS 2 is layered
================

```faqe:graph
title = "Application code is insulated from transport"
columns = 5
rows = 1

[[nodes]]
id = "app"
title = "application"
subtitle = "your nodes"
column = 1
row = 1
tone = "positive"

[[nodes]]
id = "client"
title = "client library"
subtitle = "rclcpp · rclpy · rclrs"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "rcl"
title = "rcl"
subtitle = "common C layer"
column = 3
row = 1
tone = "neutral"

[[nodes]]
id = "rmw"
title = "rmw"
subtitle = "middleware interface"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "wire"
title = "implementation"
subtitle = "Fast DDS · Cyclone · Zenoh"
column = 5
row = 1
tone = "negative"

[[edges]]
from = "app"
to = "client"

[[edges]]
from = "client"
to = "rcl"

[[edges]]
from = "rcl"
to = "rmw"

[[edges]]
from = "rmw"
to = "wire"
label = "select"
```

Design against ROS semantics; qualify the exact RMW and configuration you ship.

<!-- speaker_note: The abstraction is real; the implementations still behave differently in production. Test the exact RMW, version, configuration, network, and workload you plan to ship. -->

<!-- end_slide -->

Middleware is a product decision
================================

```faqe:table
title = "Common ROS 2 RMW choices"
variant = "comparison"
columns = ["RMW", "Strength", "Watch"]

[[rows]]
cells = ["Fast DDS", "default, broad support, discovery-server option", "XML profiles and discovery behavior"]

[[rows]]
cells = ["Cyclone DDS", "lean DDS implementation and deterministic focus", "network-interface configuration"]

[[rows]]
cells = ["Connext DDS", "commercial tooling and certification paths", "license and supported feature matrix"]

[[rows]]
cells = ["Zenoh", "routed edge/cloud topologies and low overhead", "router topology and partial QoS semantics"]
```

Do not benchmark “ROS 2.” Benchmark **your graph over your chosen RMW**.

<!-- speaker_note: ROS docs recommend keeping a distributed system on the same ROS version and RMW because cross-vendor DDS interoperability is not guaranteed for every feature. -->

<!-- end_slide -->

Names still matter
==================

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "fully qualified"
title = "/robot_7/camera/image"
body = "A stable system-level contract."
tone = "positive"

[[items]]
eyebrow = "relative"
title = "camera/image"
body = "Resolved beneath the node namespace."
tone = "accent"

[[items]]
eyebrow = "private"
title = "No ROS 1 ~ shorthand"
body = "Use explicit relative names and node namespaces."
tone = "warning"

[[items]]
eyebrow = "remap"
title = "--ros-args -r"
body = "ROS arguments are separated from application arguments."
tone = "neutral"

[[items]]
eyebrow = "hidden"
title = "_prefix"
body = "Names beginning with an underscore are hidden from default CLI listings."
tone = "neutral"

[[items]]
eyebrow = "design rule"
title = "Name by meaning"
body = "Do not encode hostnames or transport topology into topic names."
tone = "positive"
```

<!-- speaker_note: Create and review an interface naming convention before bulk porting. Names become long-lived integration contracts and Zenoh key mappings later. -->

<!-- end_slide -->

Pick the communication primitive
================================

```faqe:table
title = "Topics, services, and actions express different contracts"
variant = "comparison"
columns = ["Primitive", "Use when", "Avoid when"]

[[rows]]
cells = ["Topic", "state or events flow independently of a caller", "the publisher must know completion"]
tones = ["accent", "positive", "warning"]

[[rows]]
cells = ["Service", "a short request needs one response", "work may take long or needs progress"]
tones = ["accent", "positive", "warning"]

[[rows]]
cells = ["Action", "a long operation needs feedback and cancellation", "high-rate streaming is the real requirement"]
tones = ["accent", "positive", "warning"]
```

ROS 2 actions are first-class interfaces in `.action` files rather than a convention layered on messages.

<!-- speaker_note: Migration is a chance to correct misuse. A service should not represent a five-minute operation. A topic should not fake a transactional request. -->

<!-- end_slide -->

QoS changes how you debug the graph
===================================

In ROS 1, choosing a topic often implied the transport behavior.

In ROS 2, endpoints negotiate a **delivery contract**.

```faqe:graph
title = "Offered and requested QoS must be compatible"
columns = 5
rows = 1

[[nodes]]
id = "pub"
title = "publisher"
subtitle = "offers QoS"
column = 1
row = 1
tone = "positive"

[[nodes]]
id = "offer"
title = "reliable · volatile"
subtitle = "depth 10"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "compat"
title = "compatibility"
subtitle = "not equality"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "request"
title = "best effort · volatile"
subtitle = "depth 5"
column = 4
row = 1
tone = "accent"

[[nodes]]
id = "sub"
title = "subscriber"
subtitle = "requests QoS"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "pub"
to = "offer"

[[edges]]
from = "offer"
to = "compat"
label = "offer"

[[edges]]
from = "request"
to = "compat"
label = "request"

[[edges]]
from = "compat"
to = "sub"
label = "match"
tone = "positive"
```

<!-- speaker_note: A QoS mismatch often looks like a dead topic, with no helpful exception. Make endpoint inspection the first debugging move, before changing application code. -->

<!-- end_slide -->

The QoS vocabulary
==================

```faqe:grid
columns = 4
variant = "cards"

[[items]]
eyebrow = "loss"
title = "Reliability"
body = "Reliable retries delivery; best effort may drop samples."
tone = "positive"

[[items]]
eyebrow = "late joiners"
title = "Durability"
body = "Transient local can replay retained samples; volatile starts now."
tone = "accent"

[[items]]
eyebrow = "buffer"
title = "History + depth"
body = "Keep last N or keep all, subject to resource limits."
tone = "warning"

[[items]]
eyebrow = "freshness"
title = "Deadline"
body = "Expected maximum interval between samples."
tone = "negative"

[[items]]
eyebrow = "expiry"
title = "Lifespan"
body = "How long a sample remains valid for delivery."
tone = "warning"

[[items]]
eyebrow = "presence"
title = "Liveliness"
body = "How an endpoint proves it is still alive."
tone = "positive"

[[items]]
eyebrow = "resources"
title = "Resource limits"
body = "Bound memory and outstanding samples at middleware level."
tone = "neutral"

[[items]]
eyebrow = "policy"
title = "Profile"
body = "A tested combination for a particular data class."
tone = "accent"
```

<!-- speaker_note: Most teams actively configure reliability, durability, history, and depth first. Deadline, lifespan, and liveliness become important for supervision and real-time contracts. -->

<!-- end_slide -->

Compatibility is directional
============================

```faqe:table
title = "Simplified requested/offered examples"
variant = "matrix"
columns = ["Publisher offers", "Subscriber requests", "Result"]

[[rows]]
cells = ["reliable", "reliable", "connect"]
tones = ["positive", "positive", "positive"]

[[rows]]
cells = ["reliable", "best effort", "connect"]
tones = ["positive", "warning", "positive"]

[[rows]]
cells = ["best effort", "reliable", "incompatible"]
tones = ["warning", "positive", "negative"]

[[rows]]
cells = ["transient local", "volatile", "connect; no historic request"]
tones = ["positive", "neutral", "positive"]

[[rows]]
cells = ["volatile", "transient local", "incompatible"]
tones = ["neutral", "positive", "negative"]
```

Use `ros2 topic info -v /topic` to inspect every endpoint and its actual QoS.

<!-- speaker_note: This table is intentionally simplified to the policies that control compatibility. The requested/offered model means a stronger offer can satisfy a weaker request, not vice versa. -->

<!-- end_slide -->

QoS profiles by data class
==========================

```faqe:table
title = "Starting points, not universal answers"
variant = "comparison"
columns = ["Data", "Reliability", "Durability", "Queue"]

[[rows]]
cells = ["camera / lidar", "best effort", "volatile", "small; prefer fresh data"]

[[rows]]
cells = ["commands", "reliable", "volatile", "small; add timeout in consumer"]

[[rows]]
cells = ["configuration state", "reliable", "transient local", "last known value"]

[[rows]]
cells = ["events / audit", "reliable", "volatile or stored externally", "bounded by throughput"]

[[rows]]
cells = ["maps / static transforms", "reliable", "transient local", "retain for late joiners"]
```

Define profiles centrally and test them under packet loss, restart, and slow subscribers.

<!-- speaker_note: Reliable delivery can hurt a high-rate sensor stream when fresh data queues behind retransmissions. Start with what the consumer needs, not with the reassuring name of the policy. -->

<!-- end_slide -->

Latching became durability
===========================

```faqe:graph
title = "ROS 1 latched publisher → ROS 2 transient-local durability"
columns = 5
rows = 2

[[nodes]]
id = "pub"
title = "publisher"
subtitle = "transient local"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "sample"
title = "last sample"
subtitle = "retained by writer"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "late"
title = "late subscriber"
subtitle = "requests transient local"
column = 5
row = 1
tone = "warning"

[[nodes]]
id = "gone"
title = "publisher exits"
subtitle = "retention depends on implementation/lifecycle"
column = 3
row = 2
tone = "negative"

[[edges]]
from = "pub"
to = "sample"
label = "retain"
tone = "positive"

[[edges]]
from = "sample"
to = "late"
label = "replay"
tone = "accent"

[[edges]]
from = "gone"
to = "sample"
label = "boundary"
dashed = true
tone = "negative"
```

This mapping matters for `/tf_static`, maps, configuration snapshots, and the ROS 1 bridge.

<!-- speaker_note: Do not translate latch=true on autopilot. Decide how much history matters and how long the publisher lives. The bridge often needs explicit transient-local QoS for tf_static. -->

<!-- end_slide -->

Discovery becomes observable work
=================================

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "default DDS"
title = "Multicast discovery"
body = "Convenient on a lab LAN; often blocked or noisy in production networks."
tone = "warning"

[[items]]
eyebrow = "controlled DDS"
title = "Discovery server"
body = "Explicit endpoints reduce multicast dependence and discovery fan-out."
tone = "positive"

[[items]]
eyebrow = "Zenoh"
title = "Router + gossip"
body = "Nodes connect through a router topology with configurable scouting."
tone = "accent"

[[items]]
eyebrow = "local isolation"
title = "Discovery range"
body = "Restrict discovery to localhost, subnet, or configured peers."
tone = "neutral"

[[items]]
eyebrow = "debug"
title = "Daemon state"
body = "A CLI daemon started under another RMW can report a misleading graph."
tone = "negative"

[[items]]
eyebrow = "test"
title = "Cold-start graph"
body = "Measure discovery after simultaneous boot and after network reconnection."
tone = "positive"
```

<!-- speaker_note: Discovery is part of startup performance. A talker/listener demo proves very little, so test the expected robot and endpoint counts. -->

<!-- end_slide -->

Domains isolate; they do not secure
===================================

```faqe:graph
title = "One physical network, separate logical ROS graphs"
columns = 5
rows = 2

[[nodes]]
id = "lab"
title = "domain 17"
subtitle = "integration lab"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "robot"
title = "domain 42"
subtitle = "robot cell"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "sim"
title = "domain 61"
subtitle = "simulation"
column = 5
row = 1
tone = "warning"

[[nodes]]
id = "bridge"
title = "explicit bridge"
subtitle = "selected interfaces only"
column = 3
row = 2
tone = "negative"

[[edges]]
from = "lab"
to = "bridge"
label = "allowlist"
dashed = true

[[edges]]
from = "robot"
to = "bridge"
label = "allowlist"
dashed = true

[[edges]]
from = "sim"
to = "bridge"
label = "allowlist"
dashed = true
```

`ROS_DOMAIN_ID` prevents ordinary discovery across domains. It is not authentication or access control.

<!-- speaker_note: Use domains to avoid accidental graph collisions. Use security policy, firewalls, and explicit bridges to control authority. -->

<!-- end_slide -->

Interfaces changed shape
========================

```faqe:table
title = "ROS interface migration"
variant = "comparison"
columns = ["Concern", "ROS 1", "ROS 2"]

[[rows]]
cells = ["Type name", "pkg/Message", "pkg/msg/Message"]

[[rows]]
cells = ["Service name", "pkg/Service", "pkg/srv/Service"]

[[rows]]
cells = ["Actions", "actionlib-generated topics", "pkg/action/Action first-class interface"]

[[rows]]
cells = ["IDL", ".msg and .srv generation", "ROS IDL pipeline plus .msg/.srv/.action"]

[[rows]]
cells = ["Header fields", "legacy Header sequence conventions", "builtin_interfaces/Time and explicit semantics"]
```

Freeze interface meaning before porting implementations. A compatible name is not enough if units or timing semantics drift.

<!-- speaker_note: Build interface-only packages early. They unlock the bridge, parallel ports, generated code in every language, and contract testing. -->

<!-- end_slide -->

Parameters moved into nodes
===========================

```faqe:graph
title = "Global parameter tree → owned node configuration"
columns = 5
rows = 2

[[nodes]]
id = "global"
title = "ROS 1 parameter server"
subtitle = "shared mutable namespace"
column = 1
row = 1
tone = "warning"

[[nodes]]
id = "config"
title = "configuration source"
subtitle = "versioned YAML · launch"
column = 3
row = 1
tone = "accent"

[[nodes]]
id = "a"
title = "node A parameters"
subtitle = "declared · typed · validated"
column = 5
row = 1
tone = "positive"

[[nodes]]
id = "events"
title = "parameter events"
subtitle = "changes are observable"
column = 5
row = 2
tone = "neutral"

[[edges]]
from = "global"
to = "config"
label = "migrate ownership"
tone = "accent"

[[edges]]
from = "config"
to = "a"
label = "load"
tone = "positive"

[[edges]]
from = "a"
to = "events"
label = "publish change"
```

No global parameter concept exists in ROS 2. YAML must address nodes and use `ros__parameters`.

<!-- speaker_note: Treat configuration ownership as architecture. Declare expected parameters, validate ranges, and reject unsafe atomic updates instead of reading arbitrary global keys. -->

<!-- end_slide -->

Dynamic reconfigure becomes policy
==================================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "declare"
title = "Descriptor"
body = "Document type, range, constraints, and read-only intent."
tone = "accent"

[[items]]
eyebrow = "validate"
title = "Set callback"
body = "Accept or reject a change before it becomes active."
tone = "positive"

[[items]]
eyebrow = "observe"
title = "Parameter events"
body = "Watch configuration changes across the graph."
tone = "neutral"

[[items]]
eyebrow = "atomicity"
title = "Atomic set"
body = "Apply a related group together or reject the whole update."
tone = "positive"

[[items]]
eyebrow = "lifecycle"
title = "Change window"
body = "Require deactivation before applying configuration that reallocates resources."
tone = "warning"

[[items]]
eyebrow = "safety"
title = "Not every knob"
body = "Do not expose internal tuning merely because runtime parameters exist."
tone = "negative"
```

<!-- speaker_note: Port the dynamic_reconfigure behavior, not its UI. Define which parameters are safe live, which require an inactive lifecycle state, and which should be immutable. -->

<!-- end_slide -->

Launch is an event system
=========================

```faqe:table
title = "roslaunch → ROS 2 launch"
variant = "comparison"
columns = ["ROS 1 habit", "ROS 2 equivalent", "Better migration choice"]

[[rows]]
cells = ["XML only", "XML, YAML, or Python", "Prefer declarative XML/YAML until logic is required"]

[[rows]]
cells = ["type=", "exec=", "mechanical rename"]

[[rows]]
cells = ["ns=", "namespace=", "mechanical rename"]

[[rows]]
cells = ["required=true", "on_exit=shutdown", "model the failure reaction"]

[[rows]]
cells = ["machine tag", "not available", "use external orchestration"]

[[rows]]
cells = ["startup order by sleeps", "event handlers and lifecycle", "wait for readiness, not time"]
```

<!-- speaker_note: Python launch can quietly grow into an untested application. Use it for real event logic or composition, not because every launch file needs to be clever. -->

<!-- end_slide -->

Build: catkin → ament + colcon
==============================

```faqe:graph
title = "The build responsibilities separate"
columns = 5
rows = 1

[[nodes]]
id = "manifest"
title = "package.xml"
subtitle = "metadata · dependencies"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "build"
title = "ament"
subtitle = "package build conventions"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "tool"
title = "CMake / Python / Cargo"
subtitle = "language build"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "workspace"
title = "colcon"
subtitle = "workspace orchestration"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "index"
title = "ament index"
subtitle = "installed resources"
column = 5
row = 1
tone = "negative"

[[edges]]
from = "manifest"
to = "build"

[[edges]]
from = "build"
to = "tool"

[[edges]]
from = "tool"
to = "workspace"

[[edges]]
from = "workspace"
to = "index"
```

ROS 2 requires `package.xml` format 2 or newer. Metapackages become ordinary packages containing runtime dependencies.

<!-- speaker_note: Ament is the package convention and resource index. Colcon discovers and orders packages across build types. Do not describe colcon as simply “the new catkin.” -->

<!-- end_slide -->

Colcon habits
=============

```text
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --event-handlers console_cohesion+
source install/setup.bash

colcon test --packages-up-to company_robot
colcon test-result --verbose

colcon list
colcon graph
```

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "speed"
title = "packages-select"
body = "Build the package you are changing."
tone = "positive"

[[items]]
eyebrow = "confidence"
title = "packages-up-to"
body = "Include its dependency closure."
tone = "accent"

[[items]]
eyebrow = "reproducibility"
title = "clean release build"
body = "Prove the workspace without stale overlays."
tone = "warning"
```

<!-- speaker_note: Symlink install accelerates iteration but can hide installation mistakes. CI must include a clean install-space build and run from the install space. -->

<!-- end_slide -->

Overlays are powerful and dangerous
===================================

```faqe:timeline
[[items]]
title = "Source the base distribution"
meta = "/opt/ros/<distro>/setup.bash"
body = "This is the underlay and establishes the selected ROS environment."
tone = "neutral"

[[items]]
title = "Build the workspace"
meta = "colcon build"
body = "Dependencies are resolved against the underlay plus declared overlays."
tone = "accent"

[[items]]
title = "Source the overlay"
meta = "install/local_setup.bash"
body = "Workspace packages shadow same-named packages below them."
tone = "positive"

[[items]]
title = "Record the environment"
meta = "CI and deployment"
body = "A different source order can load a different ABI without changing code."
tone = "warning"

[[items]]
title = "Test a clean shell"
meta = "release gate"
body = "No developer shell history should be required to run the robot."
tone = "negative"
```

<!-- speaker_note: Many migration bugs are environment bugs. Log ROS_DISTRO, RMW_IMPLEMENTATION, prefixes, package versions, and configuration file paths in every test artifact. -->

<!-- end_slide -->

The C++ port looks familiar
===========================

```cpp
class CameraNode : public rclcpp::Node {
public:
  CameraNode() : Node("camera") {
    exposure_ = declare_parameter("exposure_ms", 8.0);
    pub_ = create_publisher<Image>("image", rclcpp::SensorDataQoS());
    timer_ = create_wall_timer(20ms, [this] { capture(); });
  }
};

int main(int argc, char **argv) {
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<CameraNode>());
  rclcpp::shutdown();
}
```

The visible API changes are easy. Ownership, execution, QoS, and shutdown behavior deserve the review.

<!-- speaker_note: Keep algorithm code outside the Node class when possible. The node should adapt ROS messages, configuration, and lifecycle to a testable domain component. -->

<!-- end_slide -->

Executors own callback scheduling
=================================

```faqe:graph
title = "Ready work enters an execution policy"
columns = 5
rows = 2

[[nodes]]
id = "sub"
title = "subscription"
subtitle = "message ready"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "timer"
title = "timer"
subtitle = "deadline ready"
column = 1
row = 2
tone = "warning"

[[nodes]]
id = "exec"
title = "executor"
subtitle = "selects callbacks"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "groups"
title = "callback groups"
subtitle = "mutual exclusion · reentrancy"
column = 3
row = 2
tone = "neutral"

[[nodes]]
id = "threads"
title = "OS threads"
subtitle = "one or many"
column = 5
row = 1
tone = "negative"

[[edges]]
from = "sub"
to = "exec"

[[edges]]
from = "timer"
to = "exec"

[[edges]]
from = "groups"
to = "exec"
label = "constrain"

[[edges]]
from = "exec"
to = "threads"
label = "dispatch"
tone = "positive"
```

`spin()` is now a convenient executor choice, not the complete execution model.

<!-- speaker_note: Executors invoke subscriptions, timers, services, actions, and guard conditions. The selected executor and callback-group layout are part of behavior and latency. -->

<!-- end_slide -->

Callback groups encode concurrency
==================================

```faqe:table
title = "Parallelism requires both threads and permission"
variant = "comparison"
columns = ["Group", "Guarantee", "Use"]

[[rows]]
cells = ["Mutually exclusive", "callbacks in the group never overlap", "shared non-thread-safe state or ordered control"]
tones = ["accent", "positive", "neutral"]

[[rows]]
cells = ["Reentrant", "callbacks may overlap, including themselves", "independent or synchronized work"]
tones = ["warning", "warning", "neutral"]

[[rows]]
cells = ["Different groups", "may execute in parallel", "isolate control, I/O, and background work"]
tones = ["positive", "warning", "positive"]
```

A multithreaded executor cannot create useful concurrency when every callback is left in one mutually-exclusive default group.

<!-- speaker_note: Store callback-group handles for the life of the node. Review every blocking call and every shared object. Concurrency should be designed, not discovered in production. -->

<!-- end_slide -->

The classic ROS 2 deadlock
==========================

```faqe:graph
title = "Synchronous wait inside a mutually-exclusive callback"
columns = 5
rows = 1

[[nodes]]
id = "cb"
title = "callback A"
subtitle = "holds group"
column = 1
row = 1
tone = "warning"

[[nodes]]
id = "call"
title = "service request"
subtitle = "waits synchronously"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "reply"
title = "response ready"
subtitle = "needs callback B"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "blocked"
title = "callback B"
subtitle = "same group blocked"
column = 4
row = 1
tone = "negative"

[[nodes]]
id = "deadlock"
title = "deadlock"
subtitle = "A waits for B"
column = 5
row = 1
tone = "negative"

[[edges]]
from = "cb"
to = "call"

[[edges]]
from = "call"
to = "reply"

[[edges]]
from = "reply"
to = "blocked"

[[edges]]
from = "blocked"
to = "deadlock"
tone = "negative"
```

Prefer async clients, explicit state machines, or callback groups that permit the response to run.

<!-- speaker_note: This is one of the most important porting reviews. Code that was harmless under ROS 1 spinner assumptions can deadlock under a ROS 2 executor and callback-group layout. -->

<!-- end_slide -->

Composition replaces nodelets
=============================

```faqe:table
title = "Processes and components become a deployment decision"
variant = "comparison"
columns = ["Separate processes", "Composable nodes"]

[[rows]]
cells = ["fault isolation", "lower serialization and process overhead"]

[[rows]]
cells = ["independent security identity", "shared address space and context"]

[[rows]]
cells = ["simple resource accounting", "dynamic load and unload"]

[[rows]]
cells = ["more discovery participants", "fewer participants and shared executor"]

[[rows]]
cells = ["copy across process boundary", "intra-process ownership transfer may avoid copies"]
```

Write components that can also run standalone. Decide composition in launch and deployment.

<!-- speaker_note: Composition is the ROS 2 replacement for the nodelet use case, but it is integrated into the node model. The trade remains performance versus fault and security isolation. -->

<!-- end_slide -->

Lifecycle makes readiness explicit
==================================

```faqe:graph
title = "Managed node state machine"
columns = 5
rows = 2

[[nodes]]
id = "unconfigured"
title = "unconfigured"
subtitle = "object exists"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "inactive"
title = "inactive"
subtitle = "resources ready · no work"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "active"
title = "active"
subtitle = "processing enabled"
column = 5
row = 1
tone = "positive"

[[nodes]]
id = "final"
title = "finalized"
subtitle = "shutdown complete"
column = 5
row = 2
tone = "negative"

[[nodes]]
id = "error"
title = "error handling"
subtitle = "recover or finalize"
column = 3
row = 2
tone = "accent"

[[edges]]
from = "unconfigured"
to = "inactive"
label = "configure"

[[edges]]
from = "inactive"
to = "active"
label = "activate"
tone = "positive"

[[edges]]
from = "active"
to = "inactive"
label = "deactivate"
tone = "warning"

[[edges]]
from = "active"
to = "error"
label = "fault"
tone = "negative"

[[edges]]
from = "error"
to = "final"
label = "fail"
tone = "negative"
```

Supervisors can configure dependencies, activate in order, and recover intentionally.

<!-- speaker_note: Lifecycle is most valuable for drivers, hardware, and coordinated stacks. It converts “node process exists” into explicit readiness and operational state. -->

<!-- end_slide -->

Zero-copy is a path, not a checkbox
===================================

```faqe:timeline
[[items]]
title = "Same process"
meta = "intra-process communication"
body = "Ownership-aware APIs may pass messages without serialization."
tone = "positive"

[[items]]
title = "Same host"
meta = "loaned messages or shared memory"
body = "Support depends on message type, client library, and RMW."
tone = "accent"

[[items]]
title = "Different hosts"
meta = "network serialization"
body = "Copies, fragmentation, and transport configuration return."
tone = "warning"

[[items]]
title = "Accelerator memory"
meta = "negotiated buffer backends"
body = "New Lyrical paths can carry backend descriptors, but transport scope remains backend-specific."
tone = "negative"

[[items]]
title = "Proof"
meta = "trace the actual path"
body = "Measure copies and fallback behavior instead of inferring from API names."
tone = "positive"
```

<!-- speaker_note: A loaned-message API does not guarantee end-to-end zero copy. The RMW, allocator, message type, topology, and subscriber behavior all matter. -->

<!-- end_slide -->

Time is a dependency
====================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "wall"
title = "System time"
body = "Calendar time; can jump under synchronization."
tone = "warning"

[[items]]
eyebrow = "duration"
title = "Steady time"
body = "Monotonic measurement for intervals and timeouts."
tone = "positive"

[[items]]
eyebrow = "simulation"
title = "ROS time"
body = "May follow `/clock`, pause, jump, or run faster than real time."
tone = "accent"

[[items]]
eyebrow = "messages"
title = "Source timestamp"
body = "When the sensor or producer observed the data."
tone = "neutral"

[[items]]
eyebrow = "middleware"
title = "Receive timestamp"
body = "When the consumer or transport observed arrival."
tone = "neutral"

[[items]]
eyebrow = "migration gate"
title = "Jump tests"
body = "Pause, rewind, restart, and change rate in simulation."
tone = "negative"
```

<!-- speaker_note: Audit every use of now(), sleep, timeout, and age. A control timeout should usually use monotonic time; a simulation algorithm may intentionally use ROS time. -->

<!-- end_slide -->

rosbag2 is a data pipeline
==========================

```faqe:table
title = "Beyond rosbag1"
variant = "comparison"
columns = ["Capability", "Migration impact"]

[[rows]]
cells = ["Storage plugins", "SQLite and MCAP are selectable backends"]

[[rows]]
cells = ["MCAP", "indexed multi-channel container with compression options"]

[[rows]]
cells = ["QoS adaptation", "recording and playback may require endpoint overrides"]

[[rows]]
cells = ["Record options", "include/exclude, regex, discovery, splitting, snapshots"]

[[rows]]
cells = ["Conversion", "storage and serialization formats can be transformed offline"]

[[rows]]
cells = ["Programmatic API", "record and play from C++ or Python workflows"]
```

Treat bags as versioned test fixtures: record schema, QoS, commit, calibration, and clock provenance.

<!-- speaker_note: MCAP preset choices trade write throughput, CRCs, indices, and compression. A fastwrite capture should be post-processed before long-term archival. -->

<!-- end_slide -->

Turn observability on
=====================

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "graph"
title = "ros2 CLI"
body = "Inspect nodes, endpoints, interfaces, QoS, lifecycle, and components."
tone = "accent"

[[items]]
eyebrow = "health"
title = "Diagnostics"
body = "Publish structured device and subsystem status."
tone = "positive"

[[items]]
eyebrow = "traffic"
title = "Topic statistics"
body = "Measure message age and period at subscriptions."
tone = "warning"

[[items]]
eyebrow = "scheduling"
title = "ros2_tracing"
body = "Trace callbacks, executor behavior, and latency using LTTng."
tone = "negative"

[[items]]
eyebrow = "logs"
title = "/rosout"
body = "Structured logger names and severity remain graph-visible."
tone = "neutral"

[[items]]
eyebrow = "release"
title = "Artifact metadata"
body = "Attach distro, RMW, QoS profiles, topology, and build identity."
tone = "positive"
```

<!-- speaker_note: Logging alone cannot diagnose scheduler latency or dropped data. Establish a layered observability plan and capture it during migration parity tests. -->

<!-- end_slide -->

Security still needs deployment work
====================================

```faqe:graph
title = "SROS 2 builds on middleware security"
columns = 5
rows = 2

[[nodes]]
id = "identity"
title = "authentication"
subtitle = "PKI identity"
column = 1
row = 1
tone = "positive"

[[nodes]]
id = "access"
title = "access control"
subtitle = "topic and service permissions"
column = 2
row = 1
tone = "accent"

[[nodes]]
id = "crypto"
title = "cryptography"
subtitle = "encryption and signing"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "enclave"
title = "enclave"
subtitle = "security context"
column = 4
row = 1
tone = "negative"

[[nodes]]
id = "ops"
title = "operations"
subtitle = "provision · rotate · revoke"
column = 5
row = 1
tone = "positive"

[[nodes]]
id = "compose"
title = "composed process"
subtitle = "nodes share address space"
column = 4
row = 2
tone = "neutral"

[[edges]]
from = "identity"
to = "access"

[[edges]]
from = "access"
to = "crypto"

[[edges]]
from = "crypto"
to = "enclave"

[[edges]]
from = "enclave"
to = "ops"

[[edges]]
from = "compose"
to = "enclave"
label = "shared context"
dashed = true
```

Enabling encryption without a key lifecycle and least-privilege policy is not a security architecture.

<!-- speaker_note: ROS 2 integrates authentication, access control, and cryptography from DDS-Security. Composition affects isolation because nodes in one process share memory and may share the security context. -->

<!-- end_slide -->

Testing becomes a migration instrument
======================================

```faqe:progress
max = 1.0

[[items]]
label = "unit"
value = 0.25
display = "logic"
text = "ROS-neutral algorithms and state machines"
tone = "neutral"

[[items]]
label = "component"
value = 0.45
display = "node"
text = "node behavior with controlled interfaces"
tone = "accent"

[[items]]
label = "launch"
value = 0.65
display = "graph"
text = "multi-process integration using launch_testing"
tone = "positive"

[[items]]
label = "network"
value = 0.82
display = "QoS"
text = "loss, delay, restart, and discovery at scale"
tone = "warning"

[[items]]
label = "hardware"
value = 1.0
display = "system"
text = "small, explicit acceptance suite on real devices"
tone = "negative"
```

Every migrated slice needs a behavioral oracle from ROS 1: bags, golden outputs, protocol traces, or hardware measurements.

<!-- speaker_note: The bars represent scope, not coverage percentage. Keep hardware tests narrow and automate everything that can run against simulation, replay, mocks, or loopback devices. -->

<!-- end_slide -->

The bridge is scaffolding
=========================

```faqe:graph
title = "Migrate one connected island at a time"
columns = 5
rows = 2

[[nodes]]
id = "r1a"
title = "ROS 1 drivers"
subtitle = "unchanged"
column = 1
row = 1
tone = "warning"

[[nodes]]
id = "r1b"
title = "ROS 1 application"
subtitle = "shrinking island"
column = 1
row = 2
tone = "warning"

[[nodes]]
id = "bridge"
title = "ros1_bridge"
subtitle = "explicit interface boundary"
column = 3
row = 1
tone = "accent"

[[nodes]]
id = "r2a"
title = "ROS 2 application"
subtitle = "migrated slice"
column = 5
row = 1
tone = "positive"

[[nodes]]
id = "r2b"
title = "ROS 2 tooling"
subtitle = "bags · tests · observability"
column = 5
row = 2
tone = "positive"

[[edges]]
from = "r1a"
to = "bridge"
label = "selected topics"

[[edges]]
from = "r1b"
to = "bridge"
label = "selected services"

[[edges]]
from = "bridge"
to = "r2a"
label = "translated"
tone = "positive"

[[edges]]
from = "bridge"
to = "r2b"
label = "observe"
tone = "accent"
```

The bridge is valuable only when the migration plan makes its surface shrink.

<!-- speaker_note: Create a bridge allowlist and an owner for every bridged interface. A dynamic bridge of the entire graph is useful for exploration and poor as a permanent architecture. -->

<!-- end_slide -->

Bridge support has a hard platform problem
==========================================

```faqe:table
title = "Official ros1_bridge compatibility reality"
variant = "comparison"
columns = ["Host", "ROS 1", "ROS 2", "Bridge status"]

[[rows]]
cells = ["Ubuntu 20.04", "Noetic", "Foxy / Galactic / Humble", "full documented support"]
tones = ["neutral", "warning", "warning", "positive"]

[[rows]]
cells = ["Ubuntu 22.04", "unofficial / partial", "Humble / Iron", "build from source; unsupported mix"]
tones = ["neutral", "warning", "warning", "warning"]

[[rows]]
cells = ["Ubuntu 24.04", "not officially available", "Jazzy / Kilted", "not supported on one native host"]
tones = ["neutral", "negative", "positive", "negative"]

[[rows]]
cells = ["Split hosts", "Noetic host", "current ROS 2 host", "use an explicit cross-network gateway design"]
tones = ["accent", "warning", "positive", "accent"]
```

Do not discover this after choosing the final ROS 2 operating system.

<!-- speaker_note: The current ros1_bridge README is explicit: Ubuntu 24.04 does not support ROS 1. Plan bridge hosting as a temporary compatibility appliance, potentially using source builds or separate hosts. -->

<!-- end_slide -->

Custom interfaces determine bridge success
==========================================

```faqe:timeline
[[items]]
title = "Build ROS 1 interfaces"
meta = "source ROS 1 workspace"
body = "The bridge must see the generated ROS 1 types."
tone = "warning"

[[items]]
title = "Build ROS 2 interfaces"
meta = "source ROS 2 workspace"
body = "Names and fields must map, or explicit mapping rules must exist."
tone = "positive"

[[items]]
title = "Build ros1_bridge"
meta = "compile-time type support"
body = "Custom message and service pairs are discovered during the bridge build."
tone = "accent"

[[items]]
title = "Verify mappings"
meta = "print pairs and integration test"
body = "Do not assume a successfully built bridge carries every custom type."
tone = "negative"

[[items]]
title = "Configure QoS"
meta = "especially tf_static and latched data"
body = "Translation of type does not automatically encode application delivery semantics."
tone = "warning"
```

<!-- speaker_note: Interface-first migration pays off here. Keep ROS 1 and ROS 2 interface packages small, buildable, and versioned before porting implementation packages. -->

<!-- end_slide -->

Inventory before implementation
===============================

```faqe:table
title = "Minimum migration inventory"
variant = "striped"
columns = ["Asset", "Record"]

[[rows]]
cells = ["Packages", "owner, language, criticality, build type, release status"]

[[rows]]
cells = ["Interfaces", "publisher, consumer, rate, size, units, QoS intent"]

[[rows]]
cells = ["Dependencies", "ROS 2 availability, replacement, fork, or removal"]

[[rows]]
cells = ["Launch", "processes, machines, namespaces, parameters, startup assumptions"]

[[rows]]
cells = ["Hardware", "drivers, kernels, vendor SDK, timing, recovery behavior"]

[[rows]]
cells = ["Evidence", "bags, tests, logs, metrics, known failures, acceptance owner"]
```

The inventory becomes the migration backlog and the proof that nothing disappeared unnoticed.

<!-- speaker_note: Generate what you can from rospack, rosnode, rostopic, rosservice, rosparam, launch files, and manifests. Then sit down with system owners; runtime-only dependencies often escape a source scan. -->

<!-- end_slide -->

Port in dependency order
========================

```faqe:graph
title = "Interfaces and leaves first; orchestration last"
columns = 5
rows = 2

[[nodes]]
id = "msg"
title = "interfaces"
subtitle = "msg · srv · action"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "lib"
title = "pure libraries"
subtitle = "algorithms · protocols"
column = 2
row = 1
tone = "positive"

[[nodes]]
id = "leaf"
title = "leaf nodes"
subtitle = "few dependencies"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "core"
title = "core services"
subtitle = "shared dependencies"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "system"
title = "system launch"
subtitle = "full graph"
column = 5
row = 1
tone = "negative"

[[nodes]]
id = "test"
title = "contract tests"
subtitle = "travel with every layer"
column = 3
row = 2
tone = "accent"

[[edges]]
from = "msg"
to = "lib"

[[edges]]
from = "lib"
to = "leaf"

[[edges]]
from = "leaf"
to = "core"

[[edges]]
from = "core"
to = "system"

[[edges]]
from = "test"
to = "leaf"
label = "verify"
dashed = true

[[edges]]
from = "test"
to = "core"
label = "verify"
dashed = true
```

<!-- speaker_note: The dependency graph, not package size, determines port order. Porting an orchestration package first creates a shell around missing dependencies. -->

<!-- end_slide -->

Use migration waves
===================

```faqe:timeline
[[items]]
title = "Wave 0 · freeze contracts"
meta = "interfaces · metrics · target distro"
body = "Capture current behavior and agree what may change."
tone = "neutral"

[[items]]
title = "Wave 1 · build the bridge island"
meta = "toolchain · CI · custom types"
body = "Prove one bidirectional interface and one release artifact."
tone = "accent"

[[items]]
title = "Wave 2 · migrate low-risk leaves"
meta = "sensors · adapters · utilities"
body = "Exercise packaging, QoS, logging, and deployment repeatedly."
tone = "positive"

[[items]]
title = "Wave 3 · migrate stateful core"
meta = "lifecycle · services · actions"
body = "Move components with recovery and ordering requirements."
tone = "warning"

[[items]]
title = "Wave 4 · retire ROS 1"
meta = "remove bridge · archive evidence"
body = "Delete the compatibility path only after fleet-level proof."
tone = "negative"
```

<!-- speaker_note: Each wave should produce a deployable system, not a branch that stays broken until the end. The bridge surface and ROS 1 package count must trend downward. -->

<!-- end_slide -->

Strangle, do not split forever
==============================

```faqe:graph
title = "One capability moves behind the same external contract"
columns = 5
rows = 2

[[nodes]]
id = "consumer"
title = "consumer"
subtitle = "stable contract"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "select"
title = "deployment selection"
subtitle = "ROS 1 or ROS 2 implementation"
column = 3
row = 1
tone = "accent"

[[nodes]]
id = "old"
title = "ROS 1 implementation"
subtitle = "reference behavior"
column = 5
row = 1
tone = "warning"

[[nodes]]
id = "new"
title = "ROS 2 implementation"
subtitle = "candidate behavior"
column = 5
row = 2
tone = "positive"

[[nodes]]
id = "compare"
title = "parity recorder"
subtitle = "same input · compare output"
column = 3
row = 2
tone = "negative"

[[edges]]
from = "consumer"
to = "select"

[[edges]]
from = "select"
to = "old"
label = "fallback"

[[edges]]
from = "select"
to = "new"
label = "promote"
tone = "positive"

[[edges]]
from = "old"
to = "compare"
label = "baseline"
dashed = true

[[edges]]
from = "new"
to = "compare"
label = "candidate"
dashed = true
```

Set a removal date for every dual implementation and every bridge route.

<!-- speaker_note: A strangler pattern is safe only if the old path actually shrinks. Feature flags and route selection need owners, metrics, and a deletion condition. -->

<!-- end_slide -->

Parity is multidimensional
==========================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "functional"
title = "Same decisions"
body = "Equivalent inputs produce acceptable outputs and side effects."
tone = "positive"

[[items]]
eyebrow = "temporal"
title = "Same deadlines"
body = "Latency, jitter, timeout, and startup remain within budget."
tone = "accent"

[[items]]
eyebrow = "failure"
title = "Same recovery"
body = "Disconnect, restart, stale data, and hardware faults resolve safely."
tone = "warning"

[[items]]
eyebrow = "resource"
title = "Known footprint"
body = "CPU, memory, bandwidth, disk, and discovery scale are measured."
tone = "neutral"

[[items]]
eyebrow = "operations"
title = "Observable"
body = "The support team can diagnose graph, QoS, lifecycle, and logs."
tone = "positive"

[[items]]
eyebrow = "security"
title = "No new authority"
body = "Migration does not accidentally widen who can publish commands."
tone = "negative"
```

<!-- speaker_note: “The node publishes the same message” is only functional parity. Production parity includes timing, resource usage, fault behavior, operations, and security. -->

<!-- end_slide -->

Production readiness gate
=========================

```faqe:progress
max = 1.0

[[items]]
label = "build"
value = 1.0
display = "required"
text = "clean, pinned, reproducible release artifact"
tone = "positive"

[[items]]
label = "contract"
value = 1.0
display = "required"
text = "interfaces, QoS, parameters, and units documented"
tone = "positive"

[[items]]
label = "failure"
value = 1.0
display = "required"
text = "restart, disconnect, timeout, and degraded mode tested"
tone = "warning"

[[items]]
label = "operate"
value = 1.0
display = "required"
text = "logs, metrics, bags, traces, and runbook available"
tone = "accent"

[[items]]
label = "rollback"
value = 1.0
display = "required"
text = "deployment can safely return to the last release"
tone = "negative"
```

There is no weighted average: one missing safety gate blocks promotion.

<!-- speaker_note: These are binary release gates, not progress percentages. Adapt them to the company's quality system and safety classification. -->

<!-- end_slide -->

ros2_control is a redesign
==========================

```faqe:table
title = "ros_control → ros2_control"
variant = "comparison"
columns = ["ROS 1", "ROS 2"]

[[rows]]
cells = ["RobotHW monolith", "Actuator, Sensor, and System hardware components"]

[[rows]]
cells = ["fixed common joint interface model", "named command and state interfaces"]

[[rows]]
cells = ["controllers access RobotHW interfaces", "ResourceManager owns and loans interfaces"]

[[rows]]
cells = ["implicit hardware startup", "hardware and controller lifecycle"]

[[rows]]
cells = ["single expected update pattern", "different rates and asynchronous components supported"]

[[rows]]
cells = ["configuration around transmissions", "hardware description integrated through URDF ros2_control tags"]
```

Port the hardware boundary first; do not wrap `RobotHW` indefinitely.

<!-- speaker_note: ros2_control deliberately removes some rigid ROS 1 assumptions. Split hardware components where their lifecycle, update rate, or failure domain differs. -->

<!-- end_slide -->

<!-- jump_to_middle -->
<!-- alignment: center -->

Rust belongs in the graph
=========================

**Rust catches ownership mistakes before the robot gets a chance to. It still speaks normal ROS 2.**

Trim Bresilla · active ROS 2 Rust contributor · `rclrs` crate owner

<!-- speaker_note: I am not proposing a Rust rewrite. I want to show where Rust pays for itself and how rclrs joins the same graph through the normal ROS interfaces and RMW. -->

<!-- end_slide -->

Where Rust pays for itself
==========================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "memory"
title = "No GC, fewer memory hazards"
body = "Ownership prevents broad classes of use-after-free and data races at compile time."
tone = "positive"

[[items]]
eyebrow = "concurrency"
title = "Thread safety is typed"
body = "Send and Sync make cross-thread assumptions explicit in APIs."
tone = "accent"

[[items]]
eyebrow = "deployment"
title = "Builds are easier to pin"
body = "Cargo and lockfiles give us a clear dependency record, while cross-compilation support helps with deployment."
tone = "warning"

[[items]]
eyebrow = "integration"
title = "C interoperability"
body = "Wrap vendor SDKs while containing unsafe code at reviewed boundaries."
tone = "neutral"

[[items]]
eyebrow = "performance"
title = "Zero-cost abstractions"
body = "High-level iterators and state models compile without a managed runtime."
tone = "positive"

[[items]]
eyebrow = "honesty"
title = "Not automatic real-time"
body = "Allocation, scheduling, locks, middleware, and kernel behavior still require design."
tone = "negative"
```

<!-- speaker_note: Rust removes specific bug classes. It does not design the system for us, and real-time behavior still depends on the whole stack. -->

<!-- end_slide -->

ROS 2 Rust architecture
=======================

```faqe:graph
title = "rclrs uses the standard ROS 2 stack"
columns = 5
rows = 1

[[nodes]]
id = "app"
title = "Rust node"
subtitle = "domain logic"
column = 1
row = 1
tone = "warning"

[[nodes]]
id = "rclrs"
title = "rclrs"
subtitle = "safe client API"
column = 2
row = 1
tone = "negative"

[[nodes]]
id = "rcl"
title = "rcl"
subtitle = "ROS C client support"
column = 3
row = 1
tone = "accent"

[[nodes]]
id = "rmw"
title = "RMW"
subtitle = "same middleware abstraction"
column = 4
row = 1
tone = "positive"

[[nodes]]
id = "graph"
title = "ROS graph"
subtitle = "C++ · Python · Rust nodes"
column = 5
row = 1
tone = "neutral"

[[edges]]
from = "app"
to = "rclrs"

[[edges]]
from = "rclrs"
to = "rcl"

[[edges]]
from = "rcl"
to = "rmw"

[[edges]]
from = "rmw"
to = "graph"
label = "interoperate"
tone = "positive"
```

Rust is a client-language choice, not a separate robotics island.

<!-- speaker_note: rclrs binds to rcl and participates in the same graph. Existing interface definitions, QoS profiles, bags, and RMW choices remain shared across languages. -->

<!-- end_slide -->

rclrs today
============

```faqe:grid
columns = 4
variant = "strip"

[[items]]
eyebrow = "communication"
title = "Pub/sub"
body = "Sync and async publishers/subscriptions with tunable QoS and loaned messages."
tone = "positive"

[[items]]
eyebrow = "request"
title = "Services + actions"
body = "Clients and servers, including asynchronous actions."
tone = "accent"

[[items]]
eyebrow = "configuration"
title = "Parameters"
body = "Mandatory, optional, read-only, and parameter services."
tone = "warning"

[[items]]
eyebrow = "runtime"
title = "Executors"
body = "Wait sets, guard conditions, timers, and worker patterns."
tone = "negative"

[[items]]
eyebrow = "types"
title = "Message generation"
body = "All ROS message types plus dynamic message handling."
tone = "positive"

[[items]]
eyebrow = "time"
title = "Clocks"
body = "Clock, time-source, and ROS-time APIs."
tone = "accent"

[[items]]
eyebrow = "graph"
title = "Introspection"
body = "Node and topic discovery with endpoint information."
tone = "neutral"

[[items]]
eyebrow = "status"
title = "Evolving"
body = "The API is active and does not yet promise complete stability."
tone = "warning"
```

The current release is `rclrs 0.7.0`. Pin it together with the generated-message crates and ROS distribution.

<!-- speaker_note: rclrs now covers a broad slice of ROS 2. Production trouble is more likely to come from mixing versions of rclrs, rosidl_runtime_rs, rosidl_rust, generated messages, and the ROS distribution. -->

<!-- end_slide -->

A small Rust node
=================

```rust
use rclrs::{Context, Executor, QoSProfile, SpinOptions};
use std_msgs::msg::String as StringMsg;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let context = Context::default_from_env()?;
    let mut executor = Executor::new(&context)?;
    let node = executor.create_node("rust_status")?;
    let publisher = node.create_publisher::<StringMsg>(
        "status",
        QoSProfile::default(),
    )?;

    node.create_wall_timer(std::time::Duration::from_secs(1), move || {
        let msg = StringMsg { data: "alive".into() };
        let _ = publisher.publish(&msg);
    })?;
    executor.spin(SpinOptions::default()).first_error()?;
    Ok(())
}
```

The API still moves. The node remains an ordinary member of the ROS 2 graph.

<!-- speaker_note: Use examples from the pinned rclrs version during the live workshop. The API may move; the useful lesson here is the ownership model. -->

<!-- end_slide -->

Rust makes ownership visible
============================

```faqe:table
title = "Design pressure becomes compile-time feedback"
variant = "comparison"
columns = ["Question", "Rust pressure", "ROS benefit"]

[[rows]]
cells = ["Who owns the publisher?", "captured handle has an explicit lifetime", "no callback after accidental destruction"]

[[rows]]
cells = ["Can callbacks share state?", "Arc, Mutex, channels, or worker pattern", "concurrency policy is reviewable"]

[[rows]]
cells = ["Can this cross threads?", "Send and Sync constraints", "unsafe sharing fails to compile"]

[[rows]]
cells = ["Where is vendor C unsafe?", "small unsafe adapter module", "audit boundary is narrow"]

[[rows]]
cells = ["What can fail?", "Result and typed errors", "failure propagation is explicit"]
```

<!-- speaker_note: One giant mutex around the node is not a concurrency design. Let a worker own mutable domain state and send it typed commands from callbacks. -->

<!-- end_slide -->

Cargo and colcon cooperate
=========================

```faqe:graph
title = "Ament-aware Cargo packages remain workspace citizens"
columns = 5
rows = 2

[[nodes]]
id = "xml"
title = "package.xml"
subtitle = "ROS dependencies"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "toml"
title = "Cargo.toml"
subtitle = "Rust dependencies"
column = 1
row = 2
tone = "warning"

[[nodes]]
id = "ament"
title = "ament_cargo"
subtitle = "package build type"
column = 3
row = 1
tone = "negative"

[[nodes]]
id = "colcon"
title = "colcon-cargo"
subtitle = "workspace integration"
column = 3
row = 2
tone = "positive"

[[nodes]]
id = "install"
title = "install space"
subtitle = "normal ros2 run package"
column = 5
row = 1
tone = "accent"

[[edges]]
from = "xml"
to = "ament"

[[edges]]
from = "toml"
to = "ament"

[[edges]]
from = "ament"
to = "colcon"

[[edges]]
from = "colcon"
to = "install"
label = "install"
tone = "positive"
```

Pin both worlds: Cargo.lock is not a substitute for a pinned ROS distribution and workspace manifest.

<!-- speaker_note: A production release should record Cargo dependencies, vcs-imported repositories, apt packages, ROS distribution snapshots, and the compiler toolchain. -->

<!-- end_slide -->

Where Rust fits first
=====================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "high value"
title = "Protocol gateways"
body = "Own bytes, parsing, retries, and concurrency around hardware or network SDKs."
tone = "positive"

[[items]]
eyebrow = "high value"
title = "Safety supervisors"
body = "Make state, timeouts, and failure transitions explicit."
tone = "accent"

[[items]]
eyebrow = "high value"
title = "Data infrastructure"
body = "Recorders, routers, validators, and schema-aware pipelines."
tone = "warning"

[[items]]
eyebrow = "good fit"
title = "Compute nodes"
body = "Predictable native performance without a managed runtime."
tone = "positive"

[[items]]
eyebrow = "evaluate"
title = "Vendor-heavy drivers"
body = "FFI quality and upstream support may dominate language choice."
tone = "neutral"

[[items]]
eyebrow = "do not force"
title = "Mature C++ stack"
body = "A stable, tested component gains little from a language rewrite during migration."
tone = "negative"
```

<!-- speaker_note: Introduce Rust at new boundaries or high-defect components. Do not combine the ROS 2 migration with a wholesale language rewrite. -->

<!-- end_slide -->

<!-- jump_to_middle -->
<!-- alignment: center -->

Zenoh gives us another topology
==============================

**Streams, queries, and stored data share one hierarchical key space.**

Calling it “faster DDS” misses the point. Zenoh connects live, stored, and queryable data through the same key space.

<!-- speaker_note: Zenoh earns attention when robots cross unreliable links, sites, fleets, or cloud boundaries. A single robot on one good LAN may not need it. -->

<!-- end_slide -->

Three core Zenoh abstractions
=============================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "data in motion"
title = "Publish / subscribe"
body = "Writers put values under keys; subscribers observe matching key expressions."
badge = "stream"
tone = "accent"

[[items]]
eyebrow = "data in use"
title = "Query / reply"
body = "A get reaches matching queryables, which compute or retrieve replies."
badge = "ask"
tone = "warning"

[[items]]
eyebrow = "data at rest"
title = "Storage"
body = "A storage subscribes to updates and answers later queries from a backend."
badge = "remember"
tone = "positive"
```

One key expression can address live producers, computed answers, and stored state without the caller knowing their location.

<!-- speaker_note: Zenoh defines a distributed key/value space. Publishers, subscribers, queryables, and storage share key expressions, enabling location-transparent access patterns. -->

<!-- end_slide -->

Key expressions are architecture
================================

```text
company/site-a/robots/r17/state/pose
company/site-a/robots/r17/sensors/lidar/front
company/site-a/robots/r17/commands/base_velocity
company/site-a/robots/*/state/battery
company/**/events/fault
```

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "identity"
title = "Stable hierarchy"
body = "Organization, site, asset, capability, and datum each get a deliberate level."
tone = "positive"

[[items]]
eyebrow = "selection"
title = "Wildcards"
body = "`*` matches one chunk; `**` spans arbitrary hierarchy."
tone = "accent"

[[items]]
eyebrow = "governance"
title = "Policy boundary"
body = "Routing, storage, downsampling, and ACLs can target key expressions."
tone = "warning"
```

<!-- speaker_note: Do not dump ROS topic strings into a global Zenoh namespace without a design. Key hierarchy becomes the unit of routing, storage, tenancy, and permissions. -->

<!-- end_slide -->

Zenoh deployment modes
======================

```faqe:table
title = "Topology can follow the physical system"
variant = "comparison"
columns = ["Mode", "Connection model", "Use"]

[[rows]]
cells = ["Peer", "peers discover and connect directly", "small trusted local region"]
tones = ["accent", "neutral", "positive"]

[[rows]]
cells = ["Client", "maintains connection to router/broker", "constrained robot or controlled edge"]
tones = ["warning", "neutral", "positive"]

[[rows]]
cells = ["Router", "routes for other applications", "site backbone, WAN, or mesh"]
tones = ["positive", "neutral", "positive"]

[[rows]]
cells = ["Gateway / region", "hides internal topology across hierarchy", "fleet, factory, and multi-site scaling"]
tones = ["negative", "neutral", "accent"]
```

Topology is configuration, but it is still part of the tested product architecture.

<!-- speaker_note: Client mode reduces per-application connection state. Routers make explicit routed regions. Gateways can hide unnecessary internal details and reduce global discovery load. -->

<!-- end_slide -->

Two ways to connect ROS 2 with Zenoh
===================================

```faqe:table
title = "Native RMW or DDS bridge"
variant = "comparison"
columns = ["Path", "Inside robot", "Across Zenoh", "Choose when"]

[[rows]]
cells = ["rmw_zenoh_cpp", "ROS APIs map directly onto Zenoh", "native Zenoh RMW", "the full ROS 2 graph can standardize on Zenoh"]
tones = ["accent", "positive", "positive", "positive"]

[[rows]]
cells = ["zenoh-bridge-ros2dds", "nodes keep a DDS RMW", "bridge routes selected ROS graph", "existing DDS graph must remain untouched"]
tones = ["warning", "neutral", "positive", "positive"]
```

Do not run both a direct DDS path and a bridged Zenoh path between the same islands: duplicate or looping traffic can result.

<!-- speaker_note: rmw_zenoh is a middleware implementation selected by RMW_IMPLEMENTATION. zenoh-bridge-ros2dds discovers a local DDS graph and maps it into Zenoh. They solve different deployment constraints. -->

<!-- end_slide -->

rmw_zenoh operational model
===========================

```faqe:graph
title = "Default sessions use a local router for discovery"
columns = 5
rows = 2

[[nodes]]
id = "a"
title = "ROS 2 node A"
subtitle = "RMW=rmw_zenoh_cpp"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "b"
title = "ROS 2 node B"
subtitle = "RMW=rmw_zenoh_cpp"
column = 1
row = 2
tone = "accent"

[[nodes]]
id = "router"
title = "rmw_zenohd"
subtitle = "discovery · routing"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "remote"
title = "remote router"
subtitle = "site or fleet backbone"
column = 5
row = 1
tone = "warning"

[[nodes]]
id = "native"
title = "native Zenoh app"
subtitle = "query · storage · analytics"
column = 5
row = 2
tone = "negative"

[[edges]]
from = "a"
to = "router"
label = "session"

[[edges]]
from = "b"
to = "router"
label = "session"

[[edges]]
from = "router"
to = "remote"
label = "configured route"
tone = "positive"

[[edges]]
from = "router"
to = "native"
label = "key space"
tone = "accent"
```

`rmw_zenoh_cpp` has full ROS 2 support and ships in binary releases beginning with Kilted.

<!-- speaker_note: Current defaults restrict discovery and communication within a host with a Zenoh router. Router and session JSON5 configurations define how the host connects outward. -->

<!-- end_slide -->

Zenoh QoS is not DDS QoS
========================

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "good news"
title = "Fewer incompatible endpoints"
body = "Zenoh QoS profiles are essentially never incompatible in the DDS requested/offered sense."
tone = "positive"

[[items]]
eyebrow = "important"
title = "Not every policy is implemented"
body = "The ROS RMW guide notes that rmw_zenoh does not implement deadline and lifespan policies."
tone = "warning"

[[items]]
eyebrow = "configuration"
title = "Network QoS overrides"
body = "Routers can alter priority, congestion control, express mode, and reliability for matching key expressions."
tone = "accent"

[[items]]
eyebrow = "migration rule"
title = "Revalidate semantics"
body = "Switching RMW preserves APIs, not every operational guarantee or failure behavior."
tone = "negative"
```

<!-- speaker_note: RMW abstraction makes selection possible. It does not guarantee identical implementation of every policy. Maintain an RMW capability matrix for features the product relies on. -->

<!-- end_slide -->

A world model is not one topic
==============================

A useful world model answers:

```faqe:grid
columns = 3
variant = "strip"

[[items]]
eyebrow = "identity"
title = "What exists?"
body = "Robots, people, assets, zones, maps, tasks, and capabilities."
tone = "positive"

[[items]]
eyebrow = "state"
title = "What is true now?"
body = "Pose, mode, ownership, occupancy, health, and active intent."
tone = "accent"

[[items]]
eyebrow = "history"
title = "What changed?"
body = "Events, observations, commands, transitions, and provenance."
tone = "warning"

[[items]]
eyebrow = "uncertainty"
title = "How sure are we?"
body = "Source, timestamp, frame, confidence, and validity interval."
tone = "negative"

[[items]]
eyebrow = "query"
title = "What matches?"
body = "Assets in a zone, available robots, recent faults, or current task state."
tone = "positive"

[[items]]
eyebrow = "authority"
title = "Who may change it?"
body = "Sensors observe, estimators derive, planners propose, controllers command."
tone = "accent"
```

<!-- speaker_note: A latest-pose stream is not a world model. The model also needs identity, time, uncertainty, provenance, and authority. -->

<!-- end_slide -->

Separate four data classes
==========================

```faqe:table
title = "Different data deserves different retention and authority"
variant = "comparison"
columns = ["Class", "Example", "Delivery", "Retention"]

[[rows]]
cells = ["Observation", "camera detection", "stream; lossy may be acceptable", "short window or selected evidence"]

[[rows]]
cells = ["State", "robot mode or latest pose", "latest valid value", "current snapshot plus change history"]

[[rows]]
cells = ["Event", "entered zone or fault raised", "reliable and ordered per source", "append-only audit timeline"]

[[rows]]
cells = ["Command", "navigate to station", "authorized request with acknowledgement", "intent, acceptance, result"]
```

Do not use one fire-and-forget topic contract for all four.

<!-- speaker_note: The classes differ in truth semantics. An observation is evidence, state is a current assertion, an event records change, and a command requests authority to act. -->

<!-- end_slide -->

Data in motion, at rest, and in use
==================================

```faqe:graph
title = "Zenoh can connect the three without collapsing them"
columns = 5
rows = 2

[[nodes]]
id = "robot"
title = "robot publishes"
subtitle = "live observations and state"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "sub"
title = "live subscribers"
subtitle = "control · UI · alerts"
column = 3
row = 1
tone = "positive"

[[nodes]]
id = "store"
title = "storage"
subtitle = "subscribes and persists"
column = 3
row = 2
tone = "warning"

[[nodes]]
id = "query"
title = "queryable"
subtitle = "compute current answer"
column = 5
row = 1
tone = "negative"

[[nodes]]
id = "get"
title = "get"
subtitle = "location-transparent request"
column = 5
row = 2
tone = "accent"

[[edges]]
from = "robot"
to = "sub"
label = "pub/sub"
tone = "positive"

[[edges]]
from = "robot"
to = "store"
label = "persist"
tone = "warning"

[[edges]]
from = "sub"
to = "query"
label = "derive"
tone = "negative"

[[edges]]
from = "store"
to = "get"
label = "reply"
tone = "accent"

[[edges]]
from = "query"
to = "get"
label = "reply"
tone = "accent"
```

<!-- speaker_note: Zenoh storage is both subscriber and queryable. A get can receive replies from live computation and persisted state using the same key-space addressing. -->

<!-- end_slide -->

The world graph needs provenance
================================

```faqe:graph
title = "A state assertion should keep its evidence"
columns = 5
rows = 2

[[nodes]]
id = "sensor"
title = "camera 4"
subtitle = "source identity"
column = 1
row = 1
tone = "neutral"

[[nodes]]
id = "obs"
title = "observation"
subtitle = "person at x,y · t=42"
column = 2
row = 1
tone = "warning"

[[nodes]]
id = "fusion"
title = "fusion"
subtitle = "frame · confidence · policy"
column = 3
row = 1
tone = "accent"

[[nodes]]
id = "state"
title = "world assertion"
subtitle = "zone occupied"
column = 4
row = 1
tone = "positive"

[[nodes]]
id = "decision"
title = "planner decision"
subtitle = "route around zone"
column = 5
row = 1
tone = "negative"

[[nodes]]
id = "expiry"
title = "validity interval"
subtitle = "expires without evidence"
column = 4
row = 2
tone = "warning"

[[edges]]
from = "sensor"
to = "obs"
label = "witness"

[[edges]]
from = "obs"
to = "fusion"
label = "transform"

[[edges]]
from = "fusion"
to = "state"
label = "assert"
tone = "positive"

[[edges]]
from = "state"
to = "decision"
label = "influence"
tone = "negative"

[[edges]]
from = "expiry"
to = "state"
label = "bound"
tone = "warning"
```

<!-- speaker_note: The world model should support a “why?” query: which sensor, transform, timestamp, algorithm version, and confidence produced the assertion that changed behavior? -->

<!-- end_slide -->

Freshness is part of truth
==========================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "event time"
title = "Observed at"
body = "When the physical event or measurement occurred."
tone = "positive"

[[items]]
eyebrow = "ingest time"
title = "Received at"
body = "When the data infrastructure saw the sample."
tone = "accent"

[[items]]
eyebrow = "validity"
title = "True until"
body = "The interval during which an assertion may guide behavior."
tone = "warning"

[[items]]
eyebrow = "ordering"
title = "Source sequence"
body = "Detect gaps, duplicates, and reordering per producer."
tone = "neutral"

[[items]]
eyebrow = "causality"
title = "Derived from"
body = "Link state back to observations and transforms."
tone = "positive"

[[items]]
eyebrow = "conflict"
title = "Resolution policy"
body = "Do not silently use last arrival as universal truth."
tone = "negative"
```

<!-- speaker_note: Last-write-wins by receive time is often wrong for delayed robot data. State resolution should use source time, authority, confidence, and validity according to the domain. -->

<!-- end_slide -->

State and command must not share authority
==========================================

```faqe:table
title = "A data plane is not automatically a control plane"
variant = "comparison"
columns = ["Surface", "Who writes", "Required controls"]

[[rows]]
cells = ["observations", "identified sensors and perception", "schema, source identity, rate limits"]

[[rows]]
cells = ["derived world state", "authorized fusion/world-model service", "provenance, expiry, conflict policy"]

[[rows]]
cells = ["task intent", "operator or scheduler", "authentication, authorization, idempotency"]

[[rows]]
cells = ["actuator command", "local controller only", "tight allowlist, timeout, safety envelope"]

[[rows]]
cells = ["audit event", "every authority boundary", "append-only storage and clock provenance"]
```

Never let a convenient wildcard turn fleet telemetry into fleet command authority.

<!-- speaker_note: Zenoh ACL rules can target key expressions, but key-space design must make safe policy possible. Separate telemetry, intent, and actuation namespaces. -->

<!-- end_slide -->

A scalable fleet shape
======================

```faqe:graph
title = "Local autonomy, routed summaries, selective remote control"
columns = 5
rows = 3

[[nodes]]
id = "r1"
title = "robot 1 ROS graph"
subtitle = "local control stays local"
column = 1
row = 1
tone = "positive"

[[nodes]]
id = "r2"
title = "robot 2 ROS graph"
subtitle = "local control stays local"
column = 1
row = 3
tone = "positive"

[[nodes]]
id = "edge"
title = "site router"
subtitle = "filter · route · store"
column = 3
row = 2
tone = "accent"

[[nodes]]
id = "world"
title = "site world model"
subtitle = "queryables · current state"
column = 4
row = 1
tone = "warning"

[[nodes]]
id = "cloud"
title = "fleet services"
subtitle = "analytics · audit · planning"
column = 5
row = 2
tone = "negative"

[[nodes]]
id = "ops"
title = "operator"
subtitle = "authorized intent"
column = 4
row = 3
tone = "neutral"

[[edges]]
from = "r1"
to = "edge"
label = "selected data"

[[edges]]
from = "r2"
to = "edge"
label = "selected data"

[[edges]]
from = "edge"
to = "world"
label = "state"
tone = "positive"

[[edges]]
from = "edge"
to = "cloud"
label = "routed"
tone = "accent"

[[edges]]
from = "ops"
to = "edge"
label = "intent"
tone = "warning"

[[edges]]
from = "world"
to = "ops"
label = "query"
dashed = true
```

<!-- speaker_note: High-rate control loops should not depend on WAN availability. Route summaries, events, selected sensor data, and task intent across regions; keep stabilization and safety local. -->

<!-- end_slide -->

Disconnected operation is a feature
===================================

```faqe:timeline
[[items]]
title = "Connected"
meta = "normal routed operation"
body = "Robot publishes selected state and receives authorized task intent."
tone = "positive"

[[items]]
title = "Link degrades"
meta = "loss and latency rise"
body = "Downsampling and congestion policy protect command and health traffic."
tone = "warning"

[[items]]
title = "Disconnected"
meta = "local autonomy"
body = "Robot continues within a defined envelope and records an outbound event log."
tone = "accent"

[[items]]
title = "Reconnected"
meta = "reconcile"
body = "Events synchronize; current state is recomputed rather than blindly replaying commands."
tone = "negative"

[[items]]
title = "Audited"
meta = "explain divergence"
body = "Operators can inspect what happened locally while the fleet view was stale."
tone = "positive"
```

<!-- speaker_note: A world model must distinguish delayed history from current state. Reconnection should never replay expired actuator commands as if they were still valid. -->

<!-- end_slide -->

Useful ROS 2 ecosystem upgrades
===============================

```faqe:grid
columns = 4
variant = "strip"

[[items]]
eyebrow = "control"
title = "ros2_control"
body = "Lifecycle-aware hardware, resource management, and controller composition."
tone = "positive"

[[items]]
eyebrow = "navigation"
title = "Nav2"
body = "Lifecycle-managed navigation with behavior-tree orchestration."
tone = "accent"

[[items]]
eyebrow = "manipulation"
title = "MoveIt 2"
body = "ROS 2-native planning, execution, and modernized APIs."
tone = "warning"

[[items]]
eyebrow = "embedded"
title = "micro-ROS"
body = "ROS 2 concepts on RTOS and microcontroller-class devices."
tone = "negative"

[[items]]
eyebrow = "data"
title = "MCAP"
body = "Indexed, multi-channel recordings usable beyond ROS tooling."
tone = "positive"

[[items]]
eyebrow = "inspection"
title = "Foxglove"
body = "Modern visualization and remote observability workflows."
tone = "accent"

[[items]]
eyebrow = "performance"
title = "ros2_tracing"
body = "Trace executor and callback behavior rather than guessing."
tone = "warning"

[[items]]
eyebrow = "network"
title = "Zenoh"
body = "Routed edge, fleet, query, and storage patterns."
tone = "negative"
```

<!-- speaker_note: Do not adopt everything in one migration. Use the inventory to identify where an ecosystem upgrade removes a real ROS 1 limitation or an existing company-maintained fork. -->

<!-- end_slide -->

Workshop lab 1 · observe the graph
==================================

```text
ros2 node list
ros2 node info /camera
ros2 topic list -t
ros2 topic info -v /camera/image
ros2 interface show sensor_msgs/msg/Image
ros2 doctor --report
```

**Exercise:** draw the discovered graph, then stop discovery, restart one node, and explain every change the CLI reports.

**Deliverable:** a graph inventory that records endpoint QoS alongside the topic names.

<!-- speaker_note: Participants should run this with the workshop fixture or a small company subsystem. The output becomes the first ROS 2 inventory artifact. -->

<!-- end_slide -->

Workshop lab 2 · port one leaf node
===================================

```faqe:timeline
[[items]]
title = "Extract behavior"
meta = "before touching ROS APIs"
body = "Move algorithm and protocol logic behind a unit-tested boundary."
tone = "positive"

[[items]]
title = "Create ROS 2 package"
meta = "manifest · build · interface dependencies"
body = "Build and run from a clean install space."
tone = "accent"

[[items]]
title = "Port endpoints"
meta = "names · QoS · parameters"
body = "Document every changed contract."
tone = "warning"

[[items]]
title = "Replay evidence"
meta = "same recorded input"
body = "Compare outputs against ROS 1 tolerances."
tone = "positive"

[[items]]
title = "Inject failure"
meta = "restart · timeout · bad configuration"
body = "Prove degraded behavior before integration."
tone = "negative"
```

<!-- speaker_note: Choose a leaf with real value but low blast radius: a parser, diagnostic adapter, sensor utility, or stateless transform. -->

<!-- end_slide -->

Workshop lab 3 · break QoS on purpose
=====================================

```text
# Publisher
ros2 topic pub /demo std_msgs/msg/String '{data: hello}' \
  --qos-reliability best_effort

# Subscriber requesting a stronger contract
ros2 topic echo /demo std_msgs/msg/String \
  --qos-reliability reliable

ros2 topic info -v /demo
```

1. Explain why no data arrives.
2. Repair the compatibility.
3. Repeat with durability and a late subscriber.
4. Record the topic with a rosbag2 QoS override.

<!-- speaker_note: After this lab, a QoS mismatch should feel routine rather than mysterious. Inspect the offered and requested profiles before touching the code. -->

<!-- end_slide -->

Workshop lab 4 · lifecycle and failure
======================================

```faqe:grid
columns = 2
variant = "cards"

[[items]]
eyebrow = "implement"
title = "Managed driver"
bullets = ["open device on configure", "publish only while active", "close on cleanup", "report failure"]
tone = "accent"

[[items]]
eyebrow = "supervise"
title = "Bringup policy"
bullets = ["configure dependencies", "activate in order", "detect error state", "choose retry or shutdown"]
tone = "positive"

[[items]]
eyebrow = "inject"
title = "Remove the device"
body = "Observe state transition, diagnostics, command timeout, and recovery path."
tone = "negative"

[[items]]
eyebrow = "prove"
title = "No blind sleep"
body = "Startup succeeds because readiness is observed, not because five seconds elapsed."
tone = "warning"
```

<!-- speaker_note: If company hardware is unavailable, use a fake transport with the same open/read/write/fault contract. -->

<!-- end_slide -->

Workshop lab 5 · Rust in the graph
==================================

```faqe:timeline
[[items]]
title = "Build one rclrs package"
meta = "ament_cargo + colcon"
body = "Use the same interface package as C++ and Python nodes."
tone = "warning"

[[items]]
title = "Subscribe to company data"
meta = "typed generated message"
body = "Validate units and reject malformed values."
tone = "positive"

[[items]]
title = "Own state in a worker"
meta = "channel-based callback boundary"
body = "Keep mutable domain state out of arbitrary callback threads."
tone = "accent"

[[items]]
title = "Publish diagnostics"
meta = "normal ROS 2 endpoint"
body = "Inspect it using the same CLI and bag tooling."
tone = "positive"

[[items]]
title = "Compare artifacts"
meta = "size · startup · CPU · failure"
body = "Choose Rust from measured engineering value."
tone = "negative"
```

<!-- speaker_note: This is an interoperability lab, not a language contest. A small validator or gateway gives Rust a fair first job. -->

<!-- end_slide -->

Workshop lab 6 · Zenoh world state
==================================

```text
site-a/robots/r1/state/pose
site-a/robots/r1/state/battery
site-a/robots/r1/events/fault
site-a/robots/r1/intent/task
```

**Build:**

1. Publish live robot state.
2. Store only `state/**` and `events/**`.
3. Declare a queryable for “robots available in zone A.”
4. Disconnect a robot, reconnect it, and reject expired intent.
5. Show provenance for one derived world-state answer.

<!-- speaker_note: This lab makes the difference between ROS streaming data and a queryable world model concrete. Require event time, source identity, and validity on every answer. -->

<!-- end_slide -->

The first 30 days
=================

```faqe:progress
max = 1.0

[[items]]
label = "inventory"
value = 1.0
display = "complete"
text = "packages, graph, interfaces, dependencies, hardware"
tone = "positive"

[[items]]
label = "platform"
value = 1.0
display = "chosen"
text = "ROS distro, OS, RMW, compiler, deployment"
tone = "accent"

[[items]]
label = "evidence"
value = 1.0
display = "captured"
text = "bags, golden outputs, timing, failure cases"
tone = "warning"

[[items]]
label = "bridge"
value = 1.0
display = "proven"
text = "one custom type and explicit bridge topology"
tone = "negative"

[[items]]
label = "pilot"
value = 1.0
display = "selected"
text = "one low-risk leaf with an owner and gate"
tone = "positive"
```

<!-- speaker_note: Judge the first month by the risks removed, not the number of lines ported. We should finish it with a working bridge and one chosen pilot. -->

<!-- end_slide -->

Days 31–60
==========

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "deliver"
title = "Pilot in ROS 2"
body = "Package, test, deploy, observe, and roll back one real component."
tone = "positive"

[[items]]
eyebrow = "standardize"
title = "Company templates"
body = "Package layout, QoS profiles, parameters, launch, diagnostics, and CI."
tone = "accent"

[[items]]
eyebrow = "measure"
title = "Network fixture"
body = "Loss, delay, restart, cold discovery, and high endpoint counts."
tone = "warning"

[[items]]
eyebrow = "operate"
title = "Runbook"
body = "Graph, daemon, RMW, QoS, lifecycle, bag, and trace diagnostics."
tone = "positive"

[[items]]
eyebrow = "train"
title = "Code reviews"
body = "Pair on the first ports and review execution semantics explicitly."
tone = "neutral"

[[items]]
eyebrow = "control"
title = "Track bridge debt"
body = "Every route has an owner, replacement, and deletion milestone."
tone = "negative"
```

<!-- speaker_note: Let the pilot teach us what belongs in the templates. A paved road built before one production-shaped port is mostly guesswork. -->

<!-- end_slide -->

Days 61–90
==========

```faqe:timeline
[[items]]
title = "Expand the migration wave"
meta = "several related leaf packages"
body = "Use the proven package, QoS, launch, and CI conventions."
tone = "positive"

[[items]]
title = "Exercise fleet scale"
meta = "real topology and endpoint count"
body = "Measure discovery, traffic, CPU, memory, and restart convergence."
tone = "accent"

[[items]]
title = "Port one stateful core"
meta = "lifecycle and recovery"
body = "Prove startup ordering and a hardware or network failure path."
tone = "warning"

[[items]]
title = "Decide Rust and Zenoh pilots"
meta = "evidence-based adoption"
body = "Select boundaries where each solves a measured problem."
tone = "negative"

[[items]]
title = "Publish retirement forecast"
meta = "bridge surface trends to zero"
body = "Forecast the last ROS 1 release, bridge removal, and archive plan."
tone = "positive"
```

<!-- speaker_note: After 90 days, we need a repeatable way to migrate and a believable retirement date. A fully migrated robot is not required yet. -->

<!-- end_slide -->

Decision checklist
==================

```faqe:table
title = "Questions every migrated component answers"
variant = "striped"
columns = ["Decision", "Recorded answer"]

[[rows]]
cells = ["Target", "ROS distro, OS, RMW, architecture, support horizon"]

[[rows]]
cells = ["Contract", "names, types, units, QoS, rate, size, timing"]

[[rows]]
cells = ["Execution", "executor, callback groups, blocking behavior, priorities"]

[[rows]]
cells = ["Lifecycle", "configuration, readiness, shutdown, recovery"]

[[rows]]
cells = ["Evidence", "ROS 1 baseline, tests, bags, traces, acceptance limits"]

[[rows]]
cells = ["Operations", "deploy, observe, secure, update, and roll back"]
```

If an answer lives only in one engineer’s memory, it is not yet a migration artifact.

<!-- speaker_note: Use this as the component migration template and pull-request checklist. It forces architecture decisions to travel with code. -->

<!-- end_slide -->

Migration anti-patterns
=======================

```faqe:grid
columns = 3
variant = "cards"

[[items]]
eyebrow = "anti-pattern"
title = "Big-bang branch"
body = "Nothing integrates until every package ports."
tone = "negative"

[[items]]
eyebrow = "anti-pattern"
title = "Reliable everywhere"
body = "Retransmission turns stale sensor data into latency."
tone = "negative"

[[items]]
eyebrow = "anti-pattern"
title = "One default callback group"
body = "Expected parallelism never happens. Blocking work may also deadlock."
tone = "negative"

[[items]]
eyebrow = "anti-pattern"
title = "Dynamic bridge forever"
body = "Compatibility becomes invisible permanent architecture."
tone = "negative"

[[items]]
eyebrow = "anti-pattern"
title = "Python launch application"
body = "Business logic and sleeps accumulate in deployment code."
tone = "negative"

[[items]]
eyebrow = "anti-pattern"
title = "RMW swap without proof"
body = "APIs compile while operational semantics change."
tone = "negative"
```

<!-- speaker_note: Add company-specific anti-patterns during the workshop. These six are common ways a technically successful port becomes an operational regression. -->

<!-- end_slide -->

The test is a boring deployment
===============================

```faqe:graph
title = "The migration should leave a stronger engineering system"
columns = 5
rows = 1

[[nodes]]
id = "known"
title = "known contracts"
subtitle = "interfaces · QoS · time"
column = 1
row = 1
tone = "accent"

[[nodes]]
id = "tested"
title = "tested behavior"
subtitle = "normal and failure paths"
column = 2
row = 1
tone = "positive"

[[nodes]]
id = "observable"
title = "observable runtime"
subtitle = "graph · metrics · traces"
column = 3
row = 1
tone = "warning"

[[nodes]]
id = "replace"
title = "replaceable boundaries"
subtitle = "language · RMW · deployment"
column = 4
row = 1
tone = "negative"

[[nodes]]
id = "retired"
title = "ROS 1 retired"
subtitle = "bridge surface = 0"
column = 5
row = 1
tone = "positive"

[[edges]]
from = "known"
to = "tested"

[[edges]]
from = "tested"
to = "observable"

[[edges]]
from = "observable"
to = "replace"

[[edges]]
from = "replace"
to = "retired"
tone = "positive"
```

Success is not a ROS 1 imitation running on ROS 2. It is the same trusted product behavior with contracts the team can inspect and test.

<!-- speaker_note: End the main story here. A newer dependency stack is not enough; the migration has to leave behind a safer system and a process the next team can repeat. -->

<!-- end_slide -->

ROS 2 primary sources
=====================

* [ROS 1 Noetic end-of-life](https://www.ros.org/blog/noetic-eol/)
* [ROS 2 distributions and support dates](https://docs.ros.org/en/rolling/Releases.html)
* [ROS 1 → ROS 2 migration guides](https://docs.ros.org/en/lyrical/How-To-Guides/Migrating-from-ROS1.html)
* [Quality of Service settings](https://docs.ros.org/en/lyrical/Concepts/Intermediate/About-Quality-of-Service-Settings.html)
* [Executors and callback groups](https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Executors.html)
* [Composition](https://docs.ros.org/en/rolling/Tutorials/Intermediate/Composition.html)
* [ROS 2 security design](https://design.ros2.org/articles/ros2_dds_security.html)
* [ros2_tracing tutorial](https://docs.ros.org/en/lyrical/Tutorials/Advanced/ROS2-Tracing-Trace-and-Analyze.html)

<!-- speaker_note: These links support the ROS 2 architecture, migration, QoS, execution, composition, security, and observability sections. -->

<!-- end_slide -->

Migration and ecosystem sources
===============================

* [ros1_bridge current README and compatibility matrix](https://github.com/ros2/ros1_bridge/blob/master/README.md)
* [Migrating launch files](https://docs.ros.org/en/lyrical/How-To-Guides/Migrating-from-ROS1/Migrating-Launch-Files.html)
* [Migrating parameters](https://docs.ros.org/en/lyrical/How-To-Guides/Migrating-from-ROS1/Migrating-Parameters.html)
* [Migrating packages](https://docs.ros.org/en/kilted/How-To-Guides/Migrating-from-ROS1/Migrating-Packages.html)
* [rosbag2 MCAP storage](https://docs.ros.org/en/ros2_packages/kilted/api/rosbag2_storage_mcap/)
* [ros2_control differences from ROS 1](https://control.ros.org/jazzy/doc/migration/differences_to_ros1.html)
* [ROS 2 testing](https://docs.ros.org/en/rolling/Tutorials/Intermediate/Testing/Testing-Main.html)

<!-- speaker_note: The bridge platform matrix is especially time-sensitive. Recheck it when the company freezes its migration image. -->

<!-- end_slide -->

Rust and Zenoh primary sources
==============================

* [ros2-rust / rclrs](https://github.com/ros2-rust/ros2_rust)
* [rclrs 0.7.0 API documentation](https://docs.rs/rclrs/latest/rclrs/)
* [ROS 2 middleware vendors](https://docs.ros.org/en/lyrical/Concepts/Intermediate/About-Different-Middleware-Vendors.html)
* [Working with rmw_zenoh](https://docs.ros.org/en/lyrical/Installation/RMW-Implementations/Non-DDS-Implementations/Working-with-Zenoh.html)
* [rmw_zenoh implementation and configuration](https://github.com/ros2/rmw_zenoh)
* [Zenoh abstractions](https://zenoh.io/docs/manual/abstractions/)
* [Zenoh deployment topologies](https://zenoh.io/docs/getting-started/deployment/)
* [Zenoh storage manager](https://zenoh.io/docs/manual/plugin-storage-manager/)
* [zenoh-bridge-ros2dds](https://github.com/eclipse-zenoh/zenoh-plugin-ros2dds)

<!-- speaker_note: These are the current upstream sources behind the Rust, Zenoh RMW, bridge, query, storage, and topology sections. -->

<!-- end_slide -->

<!-- jump_to_middle -->
<!-- alignment: center -->
<!-- no_footer -->

Move the contracts, then the code
=================================

**ROS BLUE // RUST ORANGE // ONE ROBOT GRAPH**
