---
title: "ROS 1 → **ROS 2**"
sub_title: Architecture, migration, installation, and a practical porting workshop
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

ROS 1 → ROS 2
=============

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "presentation"
title = "40 slides"
body = "Architecture, migration, Rust, Zenoh, and world-model data."
tone = "accent"
[[items]]
eyebrow = "workshop"
title = "30 slides"
body = "Install ROS 2 and port a small ROS 1 package."
tone = "warning"
```

Trim Bresilla & Anouk Leunissen

The first section takes about forty minutes. The practical section uses a small package that everyone can finish during the session.

<!-- end_slide -->

Decisions for this migration
============================

1. Product distribution and operating-system baseline
2. Packages that can be ported mechanically
3. Contracts that need redesign
4. ROS 1 and ROS 2 coexistence period
5. Acceptance evidence for each migration batch

We will write the answers against the company system during the workshop instead of leaving them as general recommendations.

<!-- end_slide -->

ROS 1 support status
====================

* Official ROS Noetic support ended on 31 May 2025.
* Existing installations and binaries continue to run.
* Later security fixes and platform maintenance are downstream work.
* Vendor drivers and operating-system support now constrain the remaining lifetime.

An inventory of pinned packages and local patches gives us the real maintenance cost of staying on ROS 1.

<!-- end_slide -->

Target distribution
===================

```faqe:table
variant = "comparison"
columns = ["Distribution", "Platform", "EOL", "Use"]
[[rows]]
cells = ["Jazzy", "Ubuntu 24.04", "May 2029", "Mature LTS"]
[[rows]]
cells = ["Kilted", "Ubuntu 24.04", "Dec 2026", "Existing deployments"]
[[rows]]
cells = ["Lyrical", "Ubuntu 26.04", "May 2031", "New LTS"]
[[rows]]
cells = ["Rolling", "moving", "continuous", "Upstream work"]
```

The workshop uses Jazzy on Ubuntu 24.04.

Before selecting Lyrical, confirm that each camera, fieldbus, GPU, and robot-controller dependency supports Ubuntu 26.04.

<!-- end_slide -->

Migration scope
===============

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "preserve"
title = "Proven behavior"
bullets = ["algorithms", "calibration", "hardware protocols", "test vectors"]
tone = "positive"
[[items]]
eyebrow = "review"
title = "System contracts"
bullets = ["delivery", "discovery", "startup", "configuration", "recovery"]
tone = "accent"
```

Track porting and redesign as separate work.

A port should reproduce known behavior first. Architectural changes get their own tests and review.

<!-- end_slide -->

ROS 1 graph
===========

```faqe:graph
title = "Central registration; peer data path"
columns = 3
rows = 2
[[nodes]]
id = "pub"
title = "publisher"
column = 1
row = 2
[[nodes]]
id = "master"
title = "ROS master"
subtitle = "names · lookup"
column = 2
row = 1
tone = "warning"
[[nodes]]
id = "sub"
title = "subscriber"
column = 3
row = 2
[[edges]]
from = "pub"
to = "master"
[[edges]]
from = "sub"
to = "master"
[[edges]]
from = "pub"
to = "sub"
label = "TCPROS / UDPROS"
tone = "accent"
```

The master handles registration and lookup. Publishers and subscribers exchange application data directly after negotiation.

<!-- end_slide -->

ROS 2 graph
===========

```faqe:graph
title = "Distributed discovery and middleware transport"
columns = 3
rows = 2
[[nodes]]
id = "pub"
title = "publisher + QoS"
column = 1
row = 2
[[nodes]]
id = "discovery"
title = "discovery"
subtitle = "RMW-specific"
column = 2
row = 1
tone = "accent"
[[nodes]]
id = "sub"
title = "subscriber + QoS"
column = 3
row = 2
[[edges]]
from = "pub"
to = "discovery"
[[edges]]
from = "sub"
to = "discovery"
[[edges]]
from = "pub"
to = "sub"
label = "selected RMW"
tone = "warning"
```

Each endpoint announces its type and QoS. Matching happens through the selected RMW implementation.

<!-- end_slide -->

Discovery without a master
==========================

* `ROS_DOMAIN_ID` separates logical systems.
* DDS discovery usually uses multicast on a local network.
* Configured peers or discovery servers suit routed networks.
* VPNs, firewalls, containers, and Wi-Fi affect discovery.
* `ros2 daemon` is a CLI graph cache, not a master.

Use `ros2 multicast send` and `ros2 multicast receive` to check basic multicast reachability before debugging a node.

<!-- end_slide -->

ROS 2 software layers
=====================

```faqe:timeline
[[items]]
title = "application"
body = "product nodes"
[[items]]
title = "client library"
body = "rclcpp · rclpy · rclrs"
[[items]]
title = "rcl"
body = "common C API"
[[items]]
title = "RMW"
body = "middleware contract"
tone = "accent"
[[items]]
title = "transport"
body = "DDS · Zenoh"
tone = "warning"
```

The client-library API is shared across RMW implementations. Transport behavior and configuration still vary.

<!-- end_slide -->

Middleware selection
====================

Measure candidate RMWs with the same graph and network:

* discovery and restart time;
* latency, loss, throughput, and fragmentation;
* configuration and diagnostic quality;
* security and vendor support;
* application dependence on RMW-specific behavior.

Keep the raw measurements and configuration files so another engineer can repeat the comparison.

<!-- end_slide -->

Names and namespaces
====================

```bash
ros2 run camera_driver camera_node --ros-args \
  -r __ns:=/robot_7 \
  -r image_raw:=camera/front/image_raw \
  -p frame_id:=camera_front
```

Namespaces support repeated subsystems. Robot identity belongs in deployment configuration.

The same executable can run as `/robot_7/camera` or `/robot_8/camera` without recompilation.

<!-- end_slide -->

Communication primitives
========================

```faqe:table
variant = "comparison"
columns = ["Primitive", "Use", "Avoid"]
[[rows]]
cells = ["Topic", "streams and observed state", "request/reply"]
[[rows]]
cells = ["Service", "short bounded request", "long physical work"]
[[rows]]
cells = ["Action", "cancellable work with feedback", "high-rate data"]
```

During migration, verify that each ROS 1 primitive still fits the interface.

For example, a calibration request can remain a service while a two-minute docking command usually fits an action.

<!-- end_slide -->

QoS is an interface contract
============================

* Reliability: `reliable` or `best_effort`
* Durability: `volatile` or `transient_local`
* History: `keep_last` or `keep_all`
* Queue depth
* Deadline, lifespan, and liveliness

Document QoS with the type and topic name.

Review it whenever payload size, publication rate, network type, or subscriber behavior changes.

<!-- end_slide -->

QoS compatibility
=================

```faqe:table
variant = "comparison"
columns = ["Publisher", "Subscriber", "Result"]
[[rows]]
cells = ["reliable", "reliable", "match"]
[[rows]]
cells = ["reliable", "best effort", "match"]
[[rows]]
cells = ["best effort", "reliable", "no match"]
tones = ["warning", "accent", "negative"]
```

Data flows when the topic type and endpoint QoS policies are compatible.

`ros2 topic info --verbose` shows the offered and requested policies for every endpoint.

<!-- end_slide -->

QoS by data class
=================

```faqe:table
variant = "comparison"
columns = ["Data", "Starting profile", "Reason"]
[[rows]]
cells = ["camera / lidar", "best effort, shallow", "fresh samples matter"]
[[rows]]
cells = ["commands", "reliable, bounded", "loss needs handling"]
[[rows]]
cells = ["configuration", "reliable, transient local", "late joiners need state"]
[[rows]]
cells = ["events", "reliable, measured depth", "bursts need capacity"]
```

Validate profiles under expected load.

Queue depth should follow measured processing delay and burst size instead of a workspace-wide default.

<!-- end_slide -->

Discovery at product scale
==========================

Test simultaneous boot, process restart, Wi-Fi roaming, network partitions, multiple robots per broadcast domain, multi-interface containers, and VPN access.

Record:

* endpoint discovery time;
* application readiness time;
* traffic generated by discovery;
* recovery after topology changes.

Run the test once on Ethernet and once on the wireless or routed network used by the product.

<!-- end_slide -->

Domain isolation and security
=============================

`ROS_DOMAIN_ID` reduces accidental discovery; it provides no authentication.

DDS Security can provide identity, encryption, signed governance, and permissions. Deployment still needs certificate issuance, renewal, revocation, auditing, and recovery.

Domain IDs are useful for test isolation even when the production deployment enables DDS Security.

<!-- end_slide -->

Interface definitions
=====================

```faqe:table
variant = "comparison"
columns = ["ROS 1", "ROS 2"]
[[rows]]
cells = ["`Header header`", "`std_msgs/Header header`"]
[[rows]]
cells = ["mixed code and messages", "dedicated interface packages"]
[[rows]]
cells = ["implicit dependencies", "explicit build and runtime dependencies"]
[[rows]]
cells = ["message generation", "`rosidl` generators and type support"]
```

Stabilize shared interfaces before porting consumers.

Changing a field after several packages have moved creates rebuild work on both sides of the bridge.

<!-- end_slide -->

Node-scoped parameters
======================

ROS 2 has no global parameter server.

```bash
ros2 param list /camera
ros2 param get /camera exposure
ros2 param set /camera exposure 1200
ros2 param dump /camera
```

Separate startup configuration, mutable runtime state, and persistent product data.

Parameter YAML suits deployment configuration. Calibration history usually needs a separate durable store.

<!-- end_slide -->

ROS 2 launch
============

```python
camera = Node(
    package="camera_driver",
    executable="camera_node",
    namespace="robot_7",
    parameters=["config/camera.yaml"],
    remappings=[("image_raw", "camera/front/image_raw")],
)
```

Launch coordinates deployment. Business logic stays in nodes.

ROS 2 launch can react to process exit and lifecycle events. That makes startup and recovery sequences explicit.

<!-- end_slide -->

Catkin to ament and colcon
==========================

```faqe:table
variant = "comparison"
columns = ["ROS 1", "ROS 2"]
[[rows]]
cells = ["catkin package", "ament package"]
[[rows]]
cells = ["catkin tools", "colcon"]
[[rows]]
cells = ["devel space", "install space"]
[[rows]]
cells = ["devel/setup.bash", "install/setup.bash"]
```

Ament packages install executables, libraries, headers, launch files, and index resources into an install prefix.

<!-- end_slide -->

Colcon workflow
===============

```bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
colcon test
colcon test-result --verbose
```

CI should build from a clean workspace.

Use `--packages-up-to` for a focused build and `--event-handlers console_direct+` when a CI failure needs complete output.

<!-- end_slide -->

Overlay resolution
==================

```bash
source /opt/ros/jazzy/setup.bash
source ~/vendor_ws/install/setup.bash
source ~/product_ws/install/setup.bash
```

Order changes resolution. Inspect `AMENT_PREFIX_PATH` when an unexpected package version appears. Build and run with the same underlay chain.

If a package exists in two prefixes, the last sourced overlay normally wins.

<!-- end_slide -->

Executors
=========

Executors schedule subscriptions, timers, services, and actions.

* Single-threaded: serialized callbacks
* Multi-threaded: eligible callbacks may overlap
* Callback groups: node-level concurrency constraints
* Long callbacks: delay other work on the same thread

Existing concurrency and real-time constraints remain after the port.

List every callback that blocks, locks shared state, or has a deadline before choosing an executor.

<!-- end_slide -->

Callback groups and deadlocks
=============================

```faqe:table
variant = "comparison"
columns = ["Group", "Behavior", "Use"]
[[rows]]
cells = ["Mutually exclusive", "one callback at a time", "shared device state"]
[[rows]]
cells = ["Reentrant", "callbacks may overlap", "independent work"]
```

Waiting for work assigned to the same mutually exclusive group can deadlock.

A common case is a callback making a synchronous service call whose completion needs that same group.

<!-- end_slide -->

Component containers
====================

```faqe:table
variant = "comparison"
columns = ["Separate process", "Component container"]
[[rows]]
cells = ["strong fault boundary", "lower serialization overhead"]
[[rows]]
cells = ["simple ownership", "shared executor and failure domain"]
[[rows]]
cells = ["OS supervision", "runtime component loading"]
```

Choose after profiling; keep composition out of business logic.

Composition changes the failure boundary, so measure restart behavior as well as message-copy savings.

<!-- end_slide -->

Managed lifecycle
=================

```faqe:timeline
[[items]]
title = "unconfigured"
body = "process exists"
[[items]]
title = "inactive"
body = "resources ready"
tone = "warning"
[[items]]
title = "active"
body = "work enabled"
tone = "positive"
[[items]]
title = "finalized"
body = "shutdown"
tone = "negative"
```

A supervisor must own transitions and retry policy.

Define what `configure`, `activate`, `deactivate`, and `cleanup` do for each managed resource.

<!-- end_slide -->

Time and clocks
===============

* Use steady time for durations and watchdogs.
* Use ROS time for simulation and playback.
* Set `use_sim_time` consistently.
* Handle forward and backward time jumps.
* Audit direct wall-clock use.

Clock choice affects estimators, caches, timeouts, and bag playback.

Record the clock type beside every timeout and timestamp during the porting review.

<!-- end_slide -->

rosbag2
========

```bash
ros2 bag record /tf /odom /camera/image_raw
ros2 bag info my_bag
ros2 bag play my_bag --clock
```

Test storage format, compression, QoS overrides, disk throughput, and power-loss recovery over a production-length recording.

Keep one representative bag as a repeatable input for regression tests.

<!-- end_slide -->

Security deployment
===================

```faqe:timeline
[[items]]
title = "identity"
body = "participant certificates"
[[items]]
title = "governance"
body = "protected domains"
[[items]]
title = "permissions"
body = "node-level access"
[[items]]
title = "operations"
body = "rotate · revoke · audit"
tone = "warning"
```

Test denied communication as well as allowed communication.

Store keystores outside the application image and rehearse certificate replacement before deployment.

<!-- end_slide -->

Verification strategy
=====================

```faqe:table
variant = "comparison"
columns = ["Layer", "Evidence"]
[[rows]]
cells = ["logic", "unit tests and recorded vectors"]
[[rows]]
cells = ["node", "launch and parameter tests"]
[[rows]]
cells = ["graph", "interfaces, QoS, discovery"]
[[rows]]
cells = ["robot", "hardware-in-loop and faults"]
[[rows]]
cells = ["fleet", "load, partitions, recovery"]
```

Tie every migrated package to evidence at the lowest layer that can expose its failure modes.

<!-- end_slide -->

ros1_bridge
===========

```faqe:graph
columns = 3
rows = 1
[[nodes]]
id = "r1"
title = "ROS 1"
column = 1
row = 1
[[nodes]]
id = "bridge"
title = "ros1_bridge"
subtitle = "temporary"
column = 2
row = 1
tone = "warning"
[[nodes]]
id = "r2"
title = "ROS 2"
column = 3
row = 1
[[edges]]
from = "r1"
to = "bridge"
[[edges]]
from = "bridge"
to = "r2"
```

Treat each bridge route as a recorded dependency with an owner and removal date.

<!-- end_slide -->

Bridge constraints
==================

* Supported platform combinations are limited.
* Custom interfaces must be present when building the bridge.
* Fields and types must map.
* ROS 2 QoS needs an explicit policy.
* Services and actions need separate tests.
* Serialization cost matters for large messages.

Build the bridge environment early.

For custom messages, build the ROS 1 and ROS 2 interface packages before compiling the bridge.

<!-- end_slide -->

Migration order
===============

```faqe:timeline
[[items]]
title = "inventory"
body = "packages · interfaces · hardware"
[[items]]
title = "stabilize"
body = "types and test vectors"
[[items]]
title = "port leaves"
body = "tools and endpoints"
[[items]]
title = "port control"
body = "hardware and safety"
tone = "warning"
[[items]]
title = "retire"
body = "bridge and ROS 1"
tone = "positive"
```

Porting leaf packages first limits the number of active bridge routes and gives the team an early integration check.

<!-- end_slide -->

Migration batches
=================

```faqe:grid
columns = 3
variant = "cards"
[[items]]
eyebrow = "input"
title = "Known boundary"
body = "Interfaces and hardware are documented."
[[items]]
eyebrow = "proof"
title = "Measured parity"
body = "Behavior, timing, resources, and recovery."
tone = "accent"
[[items]]
eyebrow = "exit"
title = "ROS 1 removed"
body = "Old routes and deployment files are deleted."
tone = "positive"
```

Keep each batch small enough to compare against ROS 1 on the same hardware and recorded inputs.

<!-- end_slide -->

Rust in ROS 2
=============

`rclrs` connects Rust nodes to ROS 2 through the common C layer.

* Ownership clarifies device and callback state.
* `Result` makes failure paths explicit.
* Cargo handles Rust dependencies.
* `ament_cargo` and `colcon-cargo` provide workspace integration.
* Required ROS features need an API-coverage check.

Trim actively contributes to and maintains ROS Rust crates.

Start a Rust pilot by checking message, service, action, parameter, executor, and logging support against the node's requirements.

<!-- end_slide -->

rclrs node
==========

```rust
let context = rclrs::Context::new(std::env::args())?;
let node = rclrs::Node::new(&context, "rust_talker")?;
let publisher =
    node.create_publisher::<std_msgs::msg::String>("chatter")?;
publisher.publish(std_msgs::msg::String {
    data: "hello".into(),
})?;
```

A tool, adapter, or leaf node with a stable interface is a suitable first pilot.

Generated message types follow Rust ownership rules while the ROS graph remains interoperable with C++ and Python nodes.

<!-- end_slide -->

Zenoh options
=============

```faqe:table
variant = "comparison"
columns = ["Option", "Code change", "Boundary"]
[[rows]]
cells = ["`rmw_zenoh`", "select RMW", "ROS 2 domain"]
[[rows]]
cells = ["DDS bridge", "none for DDS nodes", "sites and links"]
[[rows]]
cells = ["native Zenoh", "explicit API", "non-ROS data"]
```

Measure discovery, routing, bandwidth, and recovery.

`rmw_zenoh` keeps ROS 2 graph APIs; native Zenoh exposes its own key expressions, queries, and storage model.

<!-- end_slide -->

World-model data with Zenoh
===========================

```faqe:graph
columns = 4
rows = 1
[[nodes]]
id = "robot"
title = "ROS 2 robot"
column = 1
row = 1
[[nodes]]
id = "projection"
title = "projection"
subtitle = "validate · timestamp"
column = 2
row = 1
tone = "accent"
[[nodes]]
id = "zenoh"
title = "Zenoh"
subtitle = "pub/sub · query · storage"
column = 3
row = 1
tone = "warning"
[[nodes]]
id = "apps"
title = "consumers"
subtitle = "fleet · UI · analytics"
column = 4
row = 1
[[edges]]
from = "robot"
to = "projection"
[[edges]]
from = "projection"
to = "zenoh"
[[edges]]
from = "zenoh"
to = "apps"
```

Durable claims need provenance, source time, validity, and authority.

A consumer should be able to distinguish a current observation from an older value returned by storage.

<!-- end_slide -->

Presentation summary
====================

* ROS 2 changes discovery, delivery, scheduling, configuration, and deployment.
* Existing algorithms can stay behind a new integration boundary.
* QoS, executor layout, lifecycle, and security require decisions.
* Bridge routes should shrink after every migration batch.
* Rust, Zenoh, and durable world data need separate acceptance tests.

Write down unresolved distribution, middleware, QoS, and bridge decisions before the practical section.

Questions, then a short break. The workshop starts next.

<!-- end_slide -->

Workshop: installation and first port
=====================================

```faqe:grid
columns = 3
variant = "cards"
[[items]]
eyebrow = "checkpoint A"
title = "Install"
body = "Working Jazzy demo graph."
[[items]]
eyebrow = "checkpoint B"
title = "Build"
body = "Clean colcon workspace."
tone = "accent"
[[items]]
eyebrow = "checkpoint C"
title = "Port"
body = "Translated ROS 1 package."
tone = "warning"
```

We will stop at each checkpoint long enough to compare commands and fix environment differences.

<!-- end_slide -->

Installation paths
==================

```faqe:table
variant = "comparison"
columns = ["Path", "Workshop role", "Host"]
[[rows]]
cells = ["Ubuntu deb", "primary", "Ubuntu 24.04"]
[[rows]]
cells = ["rosflake", "alternative", "Linux with Nix"]
[[rows]]
cells = ["container", "fallback", "Docker/Podman"]
```

Use one path per shell; each shell should contain either the apt environment or the Nix environment.

The commands on the next slides assume a clean Ubuntu 24.04 host unless the slide says otherwise.

<!-- end_slide -->

Ubuntu prerequisites
====================

```bash
sudo apt update
sudo apt install locales software-properties-common curl -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
sudo add-apt-repository universe
```

Checkpoint: `locale` reports UTF-8.

Open a new shell if the locale still shows `C` or a non-UTF-8 value.

<!-- end_slide -->

Configure the ROS apt source
============================

```bash
sudo apt update
export ROS_APT_SOURCE_VERSION="$(
  curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest |
  grep -F 'tag_name' | awk -F'"' '{print $4}')"
curl -L -o /tmp/ros2-apt-source.deb \
  "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```

After installation, `apt-cache policy ros-jazzy-desktop` should show a candidate from the ROS repository.

<!-- end_slide -->

Install ROS 2 Jazzy
===================

```bash
sudo apt update
sudo apt upgrade
sudo apt install ros-jazzy-desktop ros-dev-tools
```

Use `desktop` for the workshop. A product image can later use `ros-base` or an explicit package set.

Record the installed package versions before comparing results between workshop machines.

<!-- end_slide -->

Set up the shell
================

```bash
source /opt/ros/jazzy/setup.bash
echo "$ROS_DISTRO"
printenv | grep -E '^(ROS|AMENT|COLCON)'
```

Optional for Bash:

```bash
echo 'source /opt/ros/jazzy/setup.bash' >> ~/.bashrc
```

Every new terminal needs the selected ROS environment sourced before `ros2` commands can find packages.

<!-- end_slide -->

Verify talker and listener
==========================

Terminal A:

```bash
ros2 run demo_nodes_cpp talker
```

Terminal B:

```bash
ros2 run demo_nodes_py listener
```

If no data arrives, compare domain IDs, RMW selection, firewalls, and network interfaces.

Running a C++ publisher with a Python subscriber also checks cross-language message interoperability.

<!-- end_slide -->

Use rosdep
==========

```bash
sudo rosdep init       # once per machine
rosdep update
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

`package.xml` declares dependencies. `rosdep` maps the keys to system packages.

Run it again after changing package manifests so missing system dependencies appear before the build.

<!-- end_slide -->

Create a workspace
==================

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake \
  --license Apache-2.0 \
  --dependencies rclcpp std_msgs migration_demo
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```

Commit the empty package as the workshop baseline.

That commit gives the group a shared point for comparing each translation step.

<!-- end_slide -->

ROS on Nix
==========

[`nix-ros-overlay`](https://github.com/lopsided98/nix-ros-overlay) packages ROS inside Nix.

```bash
nix develop \
  github:lopsided98/nix-ros-overlay/master#example-ros2-desktop-jazzy
ros2 launch demo_nodes_cpp talker_listener_launch.xml
```

The environment can pin ROS, compilers, and non-ROS dependencies. Check package and cache coverage.

Nix is an alternative setup path here; the ported package and ROS interfaces remain the same.

<!-- end_slide -->

Trim's rosflake
===============

[`bresilla/rosflake`](https://github.com/bresilla/rosflake) is Trim's current ROS 2 development environment.

```bash
git clone https://github.com/bresilla/rosflake.git
cd rosflake
nix develop --impure
ros2 run demo_nodes_cpp talker
```

It selects Jazzy, Cyclone DDS, MoveIt, Ackermann messages, colcon, CMake, Clang, Mold, and stable Rust.

Use the repository as a working reference, then trim its package set to the needs of the company workspace.

<!-- end_slide -->

What rosflake configures
========================

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "ROS workspace"
title = "Ament and CMake paths"
body = ".rosws.nix deduplicates flags and creates a synthetic ament prefix."
tone = "accent"
[[items]]
eyebrow = "graphics"
title = "Host NVIDIA matching"
body = ".envrc and .nixgl.nix pin the host driver."
tone = "warning"
```

The shell selects Cyclone DDS. Graphics detection is why the command uses `--impure`.

If the host GPU driver changes, rebuild the environment before investigating ROS visualization failures.

<!-- end_slide -->

The package to port
===================

```text
migration_demo/
├── CMakeLists.txt
├── package.xml
├── msg/Status.msg
├── launch/demo.launch
└── src/
    ├── talker.cpp
    └── listener.cpp
```

It contains one message, parameter, publisher, subscriber, and launch file.

The package is small enough to finish today while still touching the files found in a normal port.

<!-- end_slide -->

Inventory before editing
========================

```bash
rostopic info /status
rostopic hz /status
rosmsg show migration_demo/Status
rosparam get /talker/rate
rosnode info /talker
```

Record names, types, rates, queue sizes, latching, parameters, remaps, and shutdown behavior.

Save the command output beside the package so the ROS 2 result can be compared against the same baseline.

<!-- end_slide -->

Translate package.xml
=====================

```xml
<package format="3">
  <name>migration_demo</name>
  <version>0.1.0</version>
  <maintainer email="team@example.com">Robot Team</maintainer>
  <license>Apache-2.0</license>
  <buildtool_depend>ament_cmake</buildtool_depend>
  <depend>rclcpp</depend>
  <depend>std_msgs</depend>
  <depend>rosidl_default_generators</depend>
  <exec_depend>rosidl_default_runtime</exec_depend>
  <member_of_group>rosidl_interface_packages</member_of_group>
</package>
```

Format 3 keeps build, execution, and interface-generation dependencies explicit for the ROS 2 package.

<!-- end_slide -->

Translate CMakeLists.txt
========================

```cmake
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)
rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Status.msg" DEPENDENCIES std_msgs)
add_executable(talker src/talker.cpp)
ament_target_dependencies(talker rclcpp std_msgs)
install(TARGETS talker DESTINATION lib/${PROJECT_NAME})
ament_package()
```

Generate interfaces before compiling targets that include their headers, and install every executable that launch will resolve.

<!-- end_slide -->

Translate a message
===================

ROS 1:

```text
Header header
string state
float32 temperature
```

ROS 2:

```text
std_msgs/Header header
string state
float32 temperature
```

Inspect it with `ros2 interface show migration_demo/msg/Status`.

The qualified `std_msgs/Header` name removes the package lookup that ROS 1 performed implicitly.

<!-- end_slide -->

Translate node initialization
=============================

ROS 1:

```cpp
ros::init(argc, argv, "talker");
ros::NodeHandle node;
```

ROS 2:

```cpp
rclcpp::init(argc, argv);
auto node = rclcpp::Node::make_shared("talker");
rclcpp::spin(node);
rclcpp::shutdown();
```

Larger nodes normally derive from `rclcpp::Node`.

Initialize one context per process, keep nodes alive while spinning, and shut the context down on exit.

<!-- end_slide -->

Translate a publisher
=====================

ROS 1:

```cpp
auto pub = node.advertise<std_msgs::String>("chatter", 10);
pub.publish(message);
```

ROS 2:

```cpp
auto pub = node->create_publisher<std_msgs::msg::String>(
  "chatter", rclcpp::QoS(10).reliable());
pub->publish(message);
```

Translate the queue depth and review every QoS policy.

The ROS 2 QoS object replaces the ROS 1 queue integer as the complete delivery configuration.

<!-- end_slide -->

Translate a subscription
========================

```cpp
auto sub = node->create_subscription<std_msgs::msg::String>(
  "chatter", rclcpp::QoS(10).reliable(),
  [](std_msgs::msg::String::ConstSharedPtr msg) {
    RCLCPP_INFO(rclcpp::get_logger("listener"),
                "%s", msg->data.c_str());
  });
```

Store the handle; the subscription is destroyed when it leaves scope.

In a node class, keep the subscription as a member so it lives for as long as the node.

<!-- end_slide -->

Replace the ROS 1 loop
======================

```cpp
timer_ = create_wall_timer(
  std::chrono::milliseconds(100),
  [this]() { publish_status(); });
```

ROS 2 timers are executor work. Check the old loop for blocking calls, wall-time use, and callback-dispatch assumptions.

A timer callback should return quickly or hand slow work to a controlled worker.

<!-- end_slide -->

Translate parameters
====================

```cpp
rate_hz_ = declare_parameter<double>("rate_hz", 10.0);
callback_handle_ = add_on_set_parameters_callback(
  [this](const auto & params) {
    return validate_and_apply(params);
  });
```

```bash
ros2 param set /talker rate_hz 20.0
```

Reject invalid updates without partially changing state.

Declare parameters before reading them, then keep validation and application in one transaction.

<!-- end_slide -->

Select QoS
==========

```faqe:table
variant = "comparison"
columns = ["ROS 1 behavior", "ROS 2 candidate"]
[[rows]]
cells = ["TCPROS, queue 10", "reliable, keep last 10"]
[[rows]]
cells = ["UDPROS sensor stream", "sensor data profile"]
[[rows]]
cells = ["latched state", "transient local, depth 1"]
```

Verify endpoints with `ros2 topic info /status --verbose`.

The latched ROS 1 case needs compatible transient-local durability on both sides to serve late subscribers.

<!-- end_slide -->

Translate a service
===================

```cpp
service_ = create_service<example_interfaces::srv::SetBool>(
  "enable",
  [this](const std::shared_ptr<Request> request,
         std::shared_ptr<Response> response) {
    response->success = set_enabled(request->data);
  });
```

Test with `ros2 service call`. Use an action for cancellable or long-running work.

Service callbacks should finish within a bounded time and report application errors in the response.

<!-- end_slide -->

Translate long-running work
===========================

An action defines a goal, feedback, result, and cancellation path.

* Goal acceptance and rejection
* Feedback content and frequency
* Cancellation latency
* Restart behavior
* Concurrent-goal policy

Define these before replacing a long ROS 1 service.

The client should observe goal state changes and handle cancellation that arrives during execution.

<!-- end_slide -->

Translate the launch file
=========================

```python
def generate_launch_description():
    return LaunchDescription([
        Node(
            package="migration_demo",
            executable="talker",
            parameters=[{"rate_hz": 10.0}],
            remappings=[("status", "/robot/status")],
        ),
    ])
```

Install the launch directory, rebuild, and run it with `ros2 launch`.

Keep machine-specific paths and robot identities in launch arguments or parameter files.

<!-- end_slide -->

Translate a Python node
=======================

```python
class Listener(Node):
    def __init__(self):
        super().__init__("listener")
        self.subscription = self.create_subscription(
            String, "chatter", self.on_message, 10)

    def on_message(self, msg):
        self.get_logger().info(msg.data)
```

Initialize `rclpy`, spin the node, destroy it, then call `rclpy.shutdown()`.

Keep the subscription on `self`; a local variable can be collected after initialization.

<!-- end_slide -->

Build, test, and compare
========================

```bash
colcon build --symlink-install --packages-up-to migration_demo
source install/setup.bash
colcon test --packages-select migration_demo
colcon test-result --verbose
ros2 launch migration_demo demo.launch.py
ros2 topic hz /robot/status
```

Compare content, rate, startup, shutdown, CPU use, and recovery with ROS 1.

Use the same hardware, input bag, run length, and load for both measurements.

<!-- end_slide -->

Mixed-system integration
========================

1. Build a supported `ros1_bridge` environment.
2. Source ROS 1, ROS 2, and custom interfaces in the documented order.
3. Start the routes listed for this batch.
4. Repeat functional and load tests across the bridge.
5. Give every route a removal condition.

Record the remaining ROS 1 runtime dependencies.

When a route passes its final acceptance test, remove it from the bridge configuration in the same change.

<!-- end_slide -->

Workshop acceptance
===================

```faqe:grid
columns = 2
variant = "cards"
[[items]]
eyebrow = "workshop output"
title = "Reproducible example"
bullets = ["setup notes", "workspace commit", "tests", "known failures"]
tone = "positive"
[[items]]
eyebrow = "company follow-up"
title = "First migration batch"
bullets = ["owner", "boundary", "bridge routes", "evidence", "retirement date"]
tone = "accent"
```

Choose the pilot package, target platform, owner, and review date.

End the session with the first package named, its baseline captured, and the next review placed on the calendar.
