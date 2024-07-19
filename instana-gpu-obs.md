GPU Observability With Instana
 By Yanwei Li posted Fri June 14, 2024 02:45 PM
     
 
GPU Observability With Instana
Overview of Solution
Observing the performance and health of GPUs is crucial in modern computing environments, especially for applications that require heavy computational power, such as machine learning and scientific simulations. Instana, a leading enterprise observability platform, offers comprehensive monitoring capabilities that extend to GPUs, providing insights into their utilization, performance metrics, and potential issues. This blog will guide you through setting up GPU observability with Instana, ensuring your GPU resources are optimized and any problems are promptly identified and resolved.

Reference Architecture
In this section, we will discuss the reference architecture for integrating GPU observability with Instana. The architecture involves setting up the Instana agent on systems with GPUs, configuring the GPU Operator, running the OTel Collector and ensuring the data collected is visualized effectively in the Instana dashboard.

GPU Data Collector Architecture
We are using NVIDIA DCGM Exporter to be the source of truth for GPU resources, and otel collector will scrapt GPU metrics from NVIDIA DCGM Exporter and send to Instana.

We support two patterns in Instana to collect the GPU data:

Agent Mode: For this pattern, the GPU data will be send to Instana Agent first, and the GPU sensor running on the agent will help aggregate the data and send to Instana Backend.
Agentless Mode: For this pattern, the GPU data will be send to Instana Backend directly without going through the agent, the data will go to backend otel acceptor directly.
OTel DCGM Architecture
Pre-requisites
Before setting up GPU observability with Instana, ensure you have the following:

Instana Cluster
GPU-Enabled Systems: Servers or workstations with GPUs. For this tutorial, we are testing with a OCP Cluster which contains some GPU servers.
GPU Monitoring
When you install the NVIDIA GPU Operator and create an OpenTelemetry-based data collector, you can view metrics that are related to GPU in the Instana UI.

Installing NVIDIA GPU Operator
You can install the NVIDIA GPU Operator on your GPU environment that helps manage and collect GPU metrics. Enable components such as NVIDIA drivers (to enable CUDA), Kubernetes device plugin for GPUs, and the NVIDIA Container Toolkit if you need them. For more information, see Nvidia GPU Operator

helm install gpu-operator \
   --repo https://helm.ngc.nvidia.com/nvidia \
   --namespace gpu-operator \
   --create-namespace \
   -set driver.enabled=false \
   -set toolkit.enabled=false \
   -set devicePlugin.enabled=false \
   -set mig.strategy=single \
   gpu-operator 
The NVIDIA GPU Operator installs the NVIDIA Data Center GPU Manager (DCGM) by default. DCGM Exporter is a tool to NVIDIA DCGM that allows users to gather GPU metrics and understand workload behavior or monitor GPUs in clusters.

DCGM Exporter exposes GPU metrics at an HTTP endpoint (/metrics) for monitoring solutions. For more information, see DCGM Exporter

OpenTelemetry-based Data Collectors
You can forward the OpenTelemetry data of GPU to an Instana agent or Instana backend by using the OpenTelemetry Collector. The OpenTelemetry Collector is the core component of the OpenTelemetry ecosystem, offering vendor-independent functions for telemetry data collection, processing, and export.

Forwarding Telemetry data to an Instana agent (Agent Pattern)
To enable OTLP ports for an Instana agent, add the following snippet to the configuration.yaml file of your Instana host agent. Make sure to save the changes, and restart the Instana host agent to apply the modifications.
com.instana.plugin.opentelemetry:
  grpc:
    enabled: true
  http:
    enabled: true
The following snippet shows a typical configuration for the OpenTelemetry Collector to forward telemetry data to a local Instana host agent by using the OTLP/gRPC protocol.
Create a YAML file, such as config.yaml, as follows:

receivers:
  otlp:
    protocols:
      grpc:
  prometheus/nvidia-dcgm:
    config:
      scrape_configs:
        - job_name: 'nvidia-dcgm'
          scrape_interval: 10s
          static_configs:
            - targets: "$(DCGM_EXPORTOR_ENDPOINT)"
processors:
  batch:
  resource:
    attributes:
      - key: server.address
        from_attribute: net.host.name
        action: insert
      - key: server.port
        from_attribute: net.host.port
        action: insert
      - key: service.name
        value: nvidia-dcgm
        action: update
      - key: INSTANA_PLUGIN
        value: dcgm
        action: insert
exporters:
  otlp:
    endpoint: "$(INSTANA_AGENT_HOST):4317"
    tls:
      insecure: true
service:
  pipelines:
    metrics/nvidia-dcgm:
      receivers: [prometheus/nvidia-dcgm]
      processors: [batch, resource]
      exporters: [otlp]
The following example shows a typical configuration of the OpenTelemetry Collector for forwarding Telemetry data to a local Instana host agent with the OTLP/HTTP protocol.

exporters:
  otlphttp:
    endpoint: "$(INSTANA_AGENT_HOST):4318"
    tls:
      insecure: true
Notes:

Set the DCGM_EXPORTOR_ENDPOINT field with the DCGM Exporter endpoint.
Set the INSTANA_AGENT_HOST field with the IP or the host of the Instana agent to connect to.
Instana uses OTLP standard port numbers, such as 4317 for OTLP/gRPC and 4318 for OTLP/HTTP.
After you complete all configuration changes in the config.yaml file, run the following command to use the OpenTelemetry Collector:
docker run -d -p 4317:4317 -v $(pwd)/config.yaml:/etc/otelcol-contrib/config.yaml otel/opentelemetry-collector-contrib:latest
Forwarding Telemetry data to the Instana backend (Agentless Pattern)
To forward OpenTelemetry data to the Instana backend by using the OpenTelemetry Collector, complete the following steps:

Create a YAML file, such as config.yaml described above. Change the endpoint from Instana Agent endpoint to Instana Backend endpoint. The special endpoints of the backend otlp-acceptor component are used when OpenTelemetry data is sent. The Instana backend requires Instana agent key for validation. And the Instana backend also requires the host.id, faas.id, or device.id resource attribute.

exporters:
  otlp:
    endpoint: INSTANA_OTLP_GRPC_BACKEND:4317
    headers:
      x-instana-key: xxxxxxx
      x-instana-host: xxxx
Notes:

Set the INSTANA_OTLP_GRPC_BACKEND field with the correct domain name of the otlp-acceptor component of the Instana backend. For more information about the endpoint of the Instana backend otlp-acceptor, see Endpoints of Self-Hosted Instana backend otlp-acceptor or Endpoints of SaaS Instana backend otlp-acceptor.
Set the x-instana-key field with the Agent Key of the Instana agent for targeting the Instana backend. To find your agent key, you can click More > Agents in the navigation bar of the Instana UI and then click Install Agents > Windows.
Set the x-instana-host field with the host ID if no host.id, faas.id, or device.id resource attribute is defined in your application or system.
Instana uses OTLP standard port numbers, such as 4317 for OTLP/gRPC and 4318 for OTLP/HTTP. Port 443 is also supported for OTLP/HTTP.
After you complete all configuration changes in the config.yaml file, run the following command to use the OpenTelemetry Collector:
docker run -d -p 4317:4317 -v $(pwd)/config.yaml:/etc/otelcol-contrib/config.yaml otel/opentelemetry-collector-contrib:latest
Viewing Metrics
After you install OpenTelemetry (OTel) Data Collector, you can view the metrics in the Instana UI.

Open the Instana UI, and click Infrastructure. Then, click Analyze Infrastructure.
Select OTEL Dcgm from the list of types of the entities.
Click the entity instance of OTEL Dcgm entity type to open the associated dashboard.
You can view the following GPU metrics:

GPU metrics - Temperature and Power Usage
GPU metrics - Clocks and Utilization
GPU Metrics - Framebuffer
GPU Temperature: The GPU temperature refers to the operational heat measured on the graphics processing unit (GPU) of a computer. This temperature is crucial because it directly affects the performance and longevity of the GPU. Higher temperatures can lead to thermal throttling, where the GPU slows down its clock speed to reduce heat production, ultimately impacting performance. In extreme cases, excessive heat can cause permanent damage to the GPU. The ideal temperature for a GPU depends on the specific model and manufacturer, but generally, while idle, it should be between 30°C to 50°C. Under heavy load, such as during gaming or graphic-intensive tasks, it can safely reach between 65°C to 85°C. Some high-performance models might operate safely even at slightly higher temperatures. Monitoring software or built-in tools in the operating system or BIOS can be used to check the GPU temperature.
GPU Power Usage: GPU power usage refers to the amount of electrical power a graphics processing unit (GPU) consumes during its operation. This metric is important for assessing the energy efficiency of the GPU, its impact on electricity costs, and its thermal output, which in turn affects cooling requirements.
GPU SM Clocks: This refers to the clock speed of the cores that handle the actual processing tasks on the GPU. These cores are organized into groups called SMs (Shader Modules or Stream Multiprocessors), depending on the manufacturer's terminology. The SM clock speed determines how quickly these cores can execute tasks related to rendering graphics, computing physics, and other GPU-accelerated operations. Higher SM clock speeds generally indicate a capability for faster processing, which can translate to better performance in tasks such as gaming, 3D rendering, and scientific computations.
GPU Memory Clocks:  This is the clock speed of the GPU's dedicated video memory (VRAM), which plays a crucial role in how quickly data can be transferred to and from the GPU. The memory clock speed affects the bandwidth, which is the rate at which data is read from or written to the VRAM. High memory clock speeds are particularly important in applications that require large amounts of texture and video data to be loaded quickly, such as in high-resolution video editing or complex 3D environments in games.
GPU Utilization:  This measures the percentage of time the GPU's processing cores (such as shader cores or CUDA cores, depending on the architecture) are actively processing data versus being idle. It provides a general indication of how much the GPU is being leveraged by applications. High GPU utilization indicates that the GPU is being heavily used, which is typical during intense graphic or computational tasks such as gaming, 3D rendering, or deep learning processes. Conversely, low GPU utilization could indicate that the GPU is either underused or that there are bottlenecks elsewhere in the system, such as CPU limitations or insufficient memory bandwidth.
GPU Memory Cpy Utilization: This metric measures the amount of time the GPU spends performing memory operations, specifically the transfer of data to and from its memory. This can include operations like copying data from system memory to GPU memory or between different areas of GPU memory. High memory copy utilization suggests that there are significant data transfer activities taking place, which might be critical in scenarios where data needs to be frequently updated or exchanged, such as in video processing or when managing large datasets in machine learning applications.
GPU Framebuffer Memory used: This metric indicates the amount of VRAM currently being utilized to store the framebuffer content. High usage of framebuffer memory can occur when rendering high-resolution images or when multiple framebuffers are used for complex rendering techniques such as those involving multiple textures, shadow maps, or off-screen rendering for post-processing effects. Monitoring the used framebuffer memory helps in assessing whether a GPU has sufficient memory to handle the graphics workloads without needing to swap data back and forth between the VRAM and the system's main RAM, which can significantly slow down performance.
GPU Framebuffer Memory Free:  This indicates the amount of VRAM that is still available for framebuffer usage. Having a large amount of free framebuffer memory is generally good, as it suggests that the GPU can handle more or more complex rendering tasks without running out of space. In scenarios where the free framebuffer memory is low, there's a risk of running into performance issues, especially in applications that dynamically load high-resolution textures or use advanced graphical effects.
Alerts for GPU
A Custom Event enables you to create issues or incidents based on an individual metric of GPU. For example, when the GPU temperature is too high, an alert will be displayed.

GPU Alert
GPU Alert Details
Conclusion
Implementing GPU observability with Instana helps maintain optimal performance and reliability of your GPU resources. By following the steps outlined in this blog, you can ensure that your GPUs are monitored effectively, potential issues are identified early, and overall system performance is optimized.