# **PRD: Distributed Multi-Tenant Gaming Infrastructure (wolf.grffn.ca)**

## **1\. Vision**

Transform a GPU-heavy Kubernetes cluster into a private "Cloud Gaming" provider for 4+ local thin-client TVs and VR headsets. The system is entirely declarative, managed via Flux CD, and utilizes NixOS for the edge devices to provide a console-like experience for the family.

## **2\. Technical Stack**

* **Domain/Ingress:** wolf.grffn.ca via Traefik/Nginx Ingress \+ Cert-Manager (Let's Encrypt).  
* **Backend:** Kubernetes (K3s/Talos/NixOS nodes) with NVIDIA 4090 GPU.  
* **GPU Management:** NVIDIA Device Plugin with **Time-Slicing** (1x 4090 partitioned 4+ ways).  
* **Streaming Core (2D):** [Wolf](https://github.com/Games-on-Whales/wolf) \+ Sunshine.  
* **Streaming Core (VR):** [ALVR](https://github.com/alvr-org/ALVR) (Air Light VR) server-side driver for Meta Quest 3 support.  
* **UIs:**  
  * **Admin/Stats:** [Wolf Den](https://www.google.com/search?q=https://github.com/games-on-whales/wolf-den) (Web-based management at wolf.grffn.ca).  
  * **Client Picker:** [Wolf UI](https://github.com/games-on-whales/wolf-ui) (The controller-friendly "Big Picture" menu inside the stream).  
* **Thin Clients:**  
  * **TVs:** Raspberry Pi 4/5 running a minimal NixOS Kiosk image.  
  * **Headset:** Meta Quest 3 (Sideloaded Moonlight for 2D, ALVR APK for VR).

## **3\. Monorepo Architecture (/)**

* /cluster: Flux manifests (K8s resources).  
  * /apps/wolf: HelmReleases for Wolf, Wolf-Den, and Wolf-UI.  
  * /apps/vr: ALVR server-side deployments, SteamVR runtimes, and OpenVR drivers.  
  * /infrastructure: NVIDIA GPU time-slicing configurations.  
* /images: Custom Dockerfiles for Minecraft (pre-loaded with performance mods like Sodium/Iris) and ALVR/SteamVR base images.  
* /nodes: NixOS configurations for the Raspberry Pi thin clients (Kiosk builds).  
* /monitoring: Prometheus/Grafana dashboards for real-time GPU/VRAM telemetry.

## **4\. Key Features & Requirements**

### **A. The "Wolf Den" Management Portal**

* **Location:** wolf.grffn.ca  
* **Function:** Real-time dashboard showing active sessions, GPU temperature, and frame rates.  
* **Pairing:** Web-based Moonlight PIN entry and ALVR client authorization.  
* **Self-Service:** Kids can select their profile to mount their specific persistent volumes.

### **B. The NixOS Kiosk (The "Thin-Term")**

* **Performance:** Native H.265 (HEVC) hardware decoding for \<10ms latency.  
* **Experience:** Power-on directly to the game menu via cage (Wayland compositor).  
* **Controllers:** Native support for Nintendo Switch Pro, Xbox, and PS5 controllers via Bluetooth.

### **C. Meta Quest 3 & ALVR Integration**

* **2D Mode (Moonlight):** Massive virtual cinema screen for Minecraft Java or StarCraft 2 via Moonlight APK.  
* **VR Mode (ALVR):** \* **Protocol:** ALVR handles 6DOF head-tracking and stereoscopic 3D encoding.  
  * **Container Requirements:** High-privilege pods with access to /dev/uinput and /dev/nvidia-uvm for SteamVR compatibility.  
  * **Client:** Sideloaded ALVR Nightly APK for the latest Quest 3 optimizations.

### **D. Networking for VR**

* **Dedicated Traffic:** ALVR requires specific UDP port-forwarding (typically ports 9943, 9944\) through the cluster LoadBalancer.  
* **QoS:** Low-latency priority for VR packets to prevent nausea-inducing head-tracking jitter.  
* **Connectivity:** Mandatory 5GHz or 6GHz (Wi-Fi 6E) SSID to handle the 100Mbps+ VR video stream.

## **5\. Implementation Roadmap**

1. **Phase 1:** Configure NVIDIA Time-Slicing on the 4090 node to allow 4+ concurrent pods.  
2. **Phase 2:** Deploy Wolf \+ Wolf Den via Flux and map wolf.grffn.ca to the ingress.  
3. **Phase 3:** Build the NixOS SD Image for the first Pi and pair with the cluster.  
4. **Phase 4:** Deploy **ALVR Server Pod** and SteamVR runtime to enable Quest 3 VR streaming.  
5. **Phase 5:** Tuning: Optimize bitrate for AV1/HEVC encoding on the 4090 for Quest 3 clients.

## **6\. Success Metrics**

* **Latency:** \< 15ms total "round-trip" delay for 2D games; \< 30ms for VR head-to-photon.  
* **Simultaneity:** Support for 4 kids playing Minecraft @ 1080p/60fps on a single 4090\.  
* **Declarative State:** 100% of the configuration stored in Git.