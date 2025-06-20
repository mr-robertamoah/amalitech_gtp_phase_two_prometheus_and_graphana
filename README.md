## Questions

### Question 1: Container Orchestration Analysis
Examine the Docker Compose configuration used in this lab. Explain why the Node Exporter container requires mounting host directories (/proc, /sys, /) and what would happen if these mounts were removed or configured incorrectly. How does this design decision reflect the principle of containerized monitoring?

#### Answer
The node export requires mounting host directories in order to have access to the necessary files and directories to collect metrics.

Based on the flag: --collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/) used in the command section of the node-exporter service, mounts outside of those specified will cause the node-exporter container to collect metrics from the container itself which can be misleading.

This design reflects the principle of containerized monitoring in that it allows the node-exporter to collect metrics from specific directories on the host system.


### Question 2: Network Security Implications
The lab creates a custom Docker network named "prometheus-grafana". Analyze the security implications of this setup versus using the default Docker network. What potential vulnerabilities could arise from the current port exposure configuration (9090, 9100, 3000), and how would you modify the setup for a production environment?

#### Answer
The `com.docker.network.bridge.host_binding_ipv4` option of the default network is set to `0.0.0.0` which means that EC2 instances on the same network will also have access to the services running in the default network. The `prometheus-grafana` provides better network isolation since it is not bound to the host IP.

The potential vulnerabilities that could arise from the current port exposure configuration include attacks on the exposed ports since these ports are well known to have such services running on them and in resolvin them, one can change the host ports and apply proxies to these services.

### Question 3: Data Persistence Strategy
Compare the volume mounting strategies used for Prometheus data versus Grafana data in the Docker Compose file. Explain why different approaches might be needed for different components and what would happen to your dashboards and historical metrics if you removed these volume configurations.

#### Answer
Prometheus uses both bind mount and named volume while Grafana uses named volume. The approach used by Prometheus is to allow for the metrics collected to be stored in a persistent volume while allowing us to pass configurations to the prometheus container. Grafana uses named volume to persist the dashboards and other data.

If the volume configurations are removed, the metrics collected by Prometheus will be deleted as well as the dashboard and alerts created.

### Question 4: PromQL Logic Breakdown
The tutorial uses this query for calculating uptime: node_time_seconds - node_boot_time_seconds. Explain step-by-step what each metric represents, why subtraction gives us uptime, and what potential issues could arise with this calculation method. Propose an alternative approach and
justify when you might use it instead.

#### Answer
The `node_time_seconds` represents the current timestamp for the system being monitored whereas the `node_boot_time_seconds` represents the timestamp when the system was last booted.

Subtracting these two metrics will give the time, in seconds, the system has been running.

The system when restarted resets its boot time. This can lead to misleading metrics.

A best option is to use `time()` function in place of the `node_time_seconds` since it is prometheus on function that returns the current time.

### Question 5: Memory Metrics Deep Dive
The lab uses node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes to calculate memory usage. Research and explain why this approach is preferred over using node_memory_MemFree_bytes. What's the fundamental difference between "free" and "available" memory in Linux systems, and how does this impact monitoring accuracy?

#### Answer
`node_memory_MemFree_bytes` measures the amount of memory that is not allocated to a process but is available while `node_memory_MemAvailable_bytes` measures the amount of memory that is available for use by processes. This approach ensures that all forms of memory are taken into account.

The main difference between the two forms of memory is that available memory takes memory which have caches or buffers into account. These forms of memory can as well be used by processes. Free memory does not take those into account.

#### Question 6: Filesystem Query AnalysisAnalyze this filesystem usage query:
1 - (node_filesystem_avail_bytes{mountpoint="/"} /
node_filesystem_size_bytes{mountpoint="/"})
Break down the mathematical logic, explain why the result needs to be subtracted from 1, and discuss what could go wrong if you monitoring multiple mount points with this approach. How would you modify this query to exclude temporary filesystems?

#### Answer


### Question 7: Visualization Type Justification
The tutorial uses three different visualization types: Stat, Time Series, and Gauge. For each visualization created in the lab, justify why that specific type was chosen over alternatives. What criteria should guide visualization selection, and when might your choices be suboptimal?

#### Answer


### Question 8: Threshold Configuration Strategy
Explain the reasoning behind the 80% threshold setting for disk usage in the gauge chart.
Research industry standards and propose a more sophisticated alerting strategy that considers
different types of systems (database servers, web servers, etc.). How would you implement
multi-level thresholds that provide actionable insights?
Question 9: Dashboard Variable Implementation
The tutorial introduces dashboard variables using the $job variable. Explain how this variable
system works internally in Grafana, what happens when you have multiple values for a variable,
and design a scenario where poorly implemented variables could break your dashboard. How
would you test variable robustness?
Section 4: Production and Scalability Considerations
Question 10: Resource Planning and Scaling
Based on your lab experience, calculate the approximate resource requirements (CPU, memory,
storage) for monitoring 100 servers using this setup. Consider metric ingestion rates, retention
periods, and dashboard query load. What bottlenecks would you expect to encounter first, and
how would you address them?
Question 11: High Availability Design
The current lab setup is single-node. Design a high-availability architecture for this monitoring
stack that can handle component failures. Explain your approach to data consistency, load
balancing, and disaster recovery. What trade-offs would you make between complexity and
reliability?
Question 12: Security Hardening Analysis
Identify at least five security vulnerabilities in the lab setup and propose specific remediation
strategies. Consider authentication, authorization, network security, and data protection. How
would you implement secrets management and secure communication between components?Section 5: Troubleshooting and Operations
Question 13: Debugging Methodology
Describe a systematic approach to troubleshooting when Prometheus shows a target as
"DOWN" in the targets page. Walk through the diagnostic steps you would take, including
command-line tools, log analysis, and configuration verification. What are the most common
causes and their solutions?
Question 14: Performance Optimization
After running the lab, analyze the query performance of your dashboards. Identify which queries
might be expensive and explain why. Propose optimization strategies for both PromQL queries
and Grafana dashboard design. How would you monitor the monitoring system itself?
Question 15: Capacity Planning Scenario
You notice that Prometheus is consuming increasing amounts of disk space over time. Analyze
the factors that contribute to storage growth, calculate retention policies based on business
requirements, and design a data lifecycle management strategy. How would you balance
historical data availability with resource constraints?