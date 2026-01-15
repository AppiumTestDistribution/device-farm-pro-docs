---
title: Overview
hide:
  - navigation
  - toc
---


<div style="text-align: center">
  <img src="assets/images/DeviceFarm-Logo.jpg" class="center" width="200"/>
</div>

## :material-swap-horizontal: Architecture Evolution: Plugin vs Orchestrator/Runner

This project evolved from [appium-device-farm](https://github.com/AppiumTestDistribution/appium-device-farm) (Appium plugin) to a distributed **Hub-Node architecture** for enterprise scalability. Here's how they compare:

### Quick Comparison

| Aspect | Plugin (appium-device-farm) | Orchestrator/Runner (This Project) |
|--------|---------------------------|-----------------------------------|
| **Architecture** | Monolithic Appium plugin | Distributed Hub-Node system |
| **Installation** | `appium plugin install device-farm` | Hub + Node services + PostgreSQL |
| **Communication** | HTTP polling | WebSocket (persistent, faster) |
| **Codebase** | Monorepo (cluttered) | Clean separation (3 packages) |
| **Scalability** | Supports multiple nodes | Optimized for 10-500+ devices |
| **High Availability** | ❌ Single point of failure | ✅ Node failures don't affect Hub |
| **Database** | LokiJS + Prisma/SQLite (in-memory/file) | PostgreSQL (multi-node concurrency) |
| **Deployment Updates** | Requires Appium restart | Hub & Nodes update independently |
| **Fault Isolation** | Crash affects all devices | Crash affects only that node's devices |
| **Use Case** | Small teams, CI/CD | Production farms, enterprises |
| **Observability** | Limited (mixed Appium logs) | OpenTelemetry (traces, metrics, logs) |

### Why We Moved to Hub-Node

**The plugin worked great** for small setups, but enterprises hit these walls:

| Limitation | Impact | Solution in Hub-Node |
|------------|--------|---------------------|
| **Single Process** | Appium crash = all devices down | Independent nodes, isolated failures |
| **HTTP Communication** | Slower request/response, poor error recovery | WebSocket (faster, persistent, auto-reconnect) |
| **Cluttered Codebase** | Hard to maintain, plugin+dashboard+devices in one repo | Clean separation: Hub, Node, Frontend packages |
| **Tight Coupling** | Can't update without interrupting tests | Hub and nodes deploy independently |
| **LokiJS + Prisma** | In-memory/file-based, no true multi-node support | PostgreSQL with row-level locking |

### Which Should You Use?

=== "Use Plugin If..."
    - Managing **< 10 devices**
    - Single developer or small team
    - Simple CI/CD setup
    - Don't need high availability
    - Want minimal setup complexity

=== "Use Hub-Node If..."
    - Managing **10+ devices**
    - Enterprise/production environment
    - Need **99.9% uptime**
    - Devices across multiple locations
    - Require scalability and fault tolerance

---

:octicons-heart-fill-24:{ .heart } **Why Appium Device Farm?**

<div class="grid cards" markdown>

-   :status-mgt:{ .device } __Enhanced Session Management__

	---

	Streamline your testing workflow for efficiency with our solution, which simplifies the setup and oversight of driver sessions on iOS simulators, tvOS devices, and Android devices

-   :status-automated:{ .device } __Automated Device Recognition__

	---

	Enjoy a hassle-free testing experience as our system automatically detects connected Android, iOS, and tvOS devices—both simulators and real hardware—prior to session initiation.

-   :status-sync:{ .device } __Proactive Device Pool Management__

	---

	During test execution, our solution updates the device pool with newly available devices, ensuring that your tests run on the latest resources.


-   :status-settings:{ .device } __Parallel Testing Enhancement__

	---

	Facilitate the concurrent running of multiple tests through the allocation of random ports, significantly boosting efficiency and decreasing total testing duration.


-   :status-gps:{ .device } __Remote Testing Facility__

	---

    Cater to the needs of geographically distributed teams or diversified testing settings with capabilities for remote device test executions.

-   :status-livetesting:{ .device } __Manual Device Interaction__

	---

	Grant testers the power for hands-on device control during tests. Whether for exploratory, debugging, or user interaction tests, our platform enhances testing depth and quality by offering complete device commands.

-   :status-remote:{ .device } __Cloud-Based Testing__
    
    ---
 	
	Integrates flawlessly with cloud services, allowing for test executions on cloud-hosted devices, broadening your testing scope.

-   :status-report:{ .device } __Advanced Reporting Insights__

	---

	Equipped with an extensive reporting dashboard, our platform delivers in-depth analyses of your testing outcomes, enabling informed decision-making and strategic planning.
</div>

Use the navigation on the left or proceed to [Setup](setup.md)!

**Big thanks to the following organizations for their support to the project**

<h3>
	<a href= "https://www.lambdatest.com"><img src="assets/images/lt.png" alt="ATD" width="45%" align="top"></a>
</h3>
