
---

### 1. **Default Bridge Network**
   - **Overview**: 
     - This is Docker’s default network. When you start a container without specifying a network, it’s attached to this bridge network. Containers on this network can communicate with each other by IP address but not by container name.
   
   - **How to Use**:
     - Create a container on the default bridge:
       ```bash
       sudo docker run -dit --name container1 busybox
       ```

     - Check the container's IP:
       ```bash
       sudo docker inspect container1 | jq -r '.[0].NetworkSettings.Networks.bridge.IPAddress'
       ```

   - **Advantages**:
     - Simple and quick setup.
     - Useful for single-host networking.

   - **Disadvantages**:
     - Lack of DNS-based container discovery (must communicate by IP).
     - Not suited for complex, multi-host setups.

---

### 2. **User-defined Bridge Network**
   - **Overview**: 
     - A bridge network created by users, which allows better control and provides automatic DNS resolution, meaning containers can communicate by name instead of just IP.

   - **How to Use**:
     - Create a user-defined bridge:
       ```bash
       sudo docker network create my_bridge
       ```

     - Run containers on the network:
       ```bash
       sudo docker run -dit --name container1 --network my_bridge busybox
       sudo docker run -dit --name container2 --network my_bridge busybox
       ```

     - Test communication:
       ```bash
       sudo docker exec container1 ping container2
       ```

   - **Advantages**:
     - DNS-based name resolution between containers.
     - Useful for service discovery in isolated environments.

   - **Disadvantages**:
     - Limited to single-host networking.

---

### 3. **MACVLAN Network**
   - **Overview**: 
     - MACVLAN allows containers to have their own MAC addresses and appear as physical devices on the network. This is useful when containers need to be treated as individual devices on the network.

   - **How to Use**:
     - Create a MACVLAN network:
       ```bash
       sudo docker network create -d macvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       -o parent=eth0 macvlan_net
       ```

     - Run a container on this network:
       ```bash
       sudo docker run -dit --name container1 --network macvlan_net busybox
       ```

   - **Advantages**:
     - Containers get direct network access.
     - Reduces network isolation and increases performance.
     - Suitable for older apps that expect a physical network interface.

   - **Disadvantages**:
     - Complex to set up.
     - No internal communication between containers on the same MACVLAN network.

---

### 4. **MACVLAN Trunked (802.1q VLAN)**
   - **Overview**:
     - A more advanced MACVLAN mode where you can use VLAN tags (802.1q) to segment traffic. This is useful when working with enterprise network environments with VLANs.
   
   - **How to Use**:
     - Create a MACVLAN with VLAN tagging:
       ```bash
       sudo docker network create -dit macvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       -o parent=eth0.100 macvlan_vlan100
       ```

     - Run a container on this VLAN network:
       ```bash
       sudo docker run -d --name container1 --network macvlan_vlan100 busybox
       ```

   - **Advantages**:
     - Network traffic can be segmented with VLANs for isolation and security.
     - Efficient for large, complex enterprise environments.

   - **Disadvantages**:
     - Requires knowledge of VLANs and switch configuration.
     - Increased setup complexity.

---

### 5. **IPVLAN (L2 Mode)**
   - **Overview**:
     - Like MACVLAN, but instead of assigning a unique MAC address to each container, all containers share the parent interface's MAC address. They can communicate directly at Layer 2.

   - **How to Use**:
     - Create an IPVLAN L2 network:
       ```bash
       sudo docker network create -dit ipvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       -o parent=eth0 ipvlan_l2
       ```

     - Run a container on this network:
       ```bash
       sudo docker run -d --name container1 --network ipvlan_l2 busybox
       ```

   - **Advantages**:
     - Efficient network traffic as containers are not limited by Docker's virtual switch.
     - Useful when working with virtualized or cloud environments.

   - **Disadvantages**:
     - No direct container-to-container communication within the same network.

---

### 6. **IPVLAN (L3 Mode)**
   - **Overview**:
     - L3 mode moves routing to Layer 3 (IP-level), giving containers their own IP addresses but sharing the parent’s MAC. It’s especially useful in larger, routed networks.

   - **How to Use**:
     - Create an IPVLAN L3 network:
       ```bash
       sudo docker network create -dit ipvlan \
       --subnet=192.168.1.0/24 \
       --gateway=192.168.1.1 \
       -o parent=eth0 -o ipvlan_mode=l3 ipvlan_l3
       ```

     - Run a container on this network:
       ```bash
       sudo docker run -d --name container1 --network ipvlan_l3 busybox
       ```

   - **Advantages**:
     - Containers can route traffic through different subnets.
     - More scalable for large container deployments with multiple subnets.

   - **Disadvantages**:
     - More complex routing setup.

---

### 7. **Overlay Network**
   - **Overview**:
     - Overlay networks enable communication between Docker services running on different hosts. It’s commonly used in Docker Swarm and Kubernetes for multi-host networking.

   - **How to Use**:
     - Create an overlay network:
       ```bash
       sudo docker network create -d overlay my_overlay
       ```

     - Run a container on the overlay network (typically used in Swarm):
       ```bash
       sudo docker service create --name my_service --network my_overlay busybox
       ```

   - **Advantages**:
     - Enables multi-host container networking.
     - Simplifies service discovery and scaling across hosts.

   - **Disadvantages**:
     - Requires Docker Swarm or Kubernetes for proper orchestration.
     - Slower compared to single-host networking.

---

### **Summary of Advantages and Disadvantages**

| Network Type          | Advantages                                                                                   | Disadvantages                                                                                 |
|-----------------------|-----------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| **Default Bridge**     | Simple setup, useful for small projects                                                       | No container name resolution, limited to single-host                                          |
| **User-defined Bridge**| DNS-based container name resolution, flexible for small setups                                | Still limited to single-host                                                                  |
| **MACVLAN**            | Direct network access, treated as physical devices                                            | No internal communication between containers on the same network, more complex setup           |
| **MACVLAN (802.1q)**   | Segments traffic with VLANs, good for enterprise environments                                 | Requires VLAN knowledge and switch configuration                                               |
| **IPVLAN (L2)**        | Efficient at L2 level, suitable for cloud environments                                        | No internal container-to-container communication                                              |
| **IPVLAN (L3)**        | IP-level routing, scalable for multi-subnet environments                                      | Complex routing setup                                                                         |
| **Overlay**            | Enables multi-host networking, essential for Swarm/Kubernetes setups                          | Slower performance, requires orchestration tools like Swarm or Kubernetes                      |

---
