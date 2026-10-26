_Computer Systems: A Programmer's Perspective_ (CS:APP) 
# Section 11.1: The Client-Server Programming Model from 

---

### **1. Core Concept & Role**

Every network application is based on the **client-server model**. In this model, an application consists of a **server process** and one or more **client processes**:

- **Server Process**: Manages some set of hardware or software **resources** and provides a service for its clients by manipulating those resources on their behalf.
    - _Web Server_: Manages disk files, retrieving or executing them for clients.
    - _FTP Server_: Manages disk files that it stores and retrieves for clients.
    - _Email Server_: Manages spool files that it reads and updates for clients.
- **Client Process**: Initiates requests for service to obtain or modify a managed resource.

---

### **2. The Client-Server Transaction**

The fundamental unit of operation in the client-server model is the **client-server transaction**. A transaction consists of four sequential steps:

1. **Client sends request**: When a client process needs service, it initiates a transaction by sending a request message across the network to the server (e.g., a Web browser requesting an HTML file).
2. **Server processes request**: The server receives the request message, interprets its contents, and manipulates its internal or disk resources in the appropriate way (e.g., reading a requested file from disk).
3. **Server sends response**: The server sends a response message back to the client and then waits for the next incoming request.
4. **Client processes response**: The client receives the response message and processes it (e.g., a Web browser renders and displays an HTML page on the screen).

---

### **3. Key Nuances: Processes vs. Hosts**

- **Processes, Not Machines**: Clients and servers are **software processes** executing in user space, not hardware machines (hosts).
- **Flexible Host Mapping**: A single physical host can run many different client and server processes concurrently.
- **Location Transparency**: A client and server transaction operates identically whether both processes reside on the **same physical host** (via local communication) or on **different hosts** across a global network.

---

### **4. Figures, Tables, and Asides in Section 11.1**

When documenting or referencing this section, refer to the following items in the text:

1. **Figure 11.1: A client-server transaction**
    - **Location**: Page 955 of CS:APP.
    - **Description**: A flow diagram illustrating the four steps of a transaction between a `Client process` and a `Server process` managing a `Resource`: \[\text{1. Client sends request} \longrightarrow \text{2. Server processes request} \longrightarrow \text{3. Server sends response} \longrightarrow \text{4. Client processes response}\]
2. **Aside: Client-server transactions versus database transactions**
    - **Location**: Page 955 of CS:APP.
    - **Description**: Clarifies a common misconception by highlighting that network client-server transactions are **not database transactions** and do not inherent properties like ACID or atomicity. In network programming, a transaction is simply a 4-step exchange of messages between client and server processes.

---

🌐 **Next Step**: Would you like to proceed to **Section 11.2 (Networks)** to explore how local area networks (LANs), Ethernet bridges/hubs, routers, and wide area networks (WANs) transport these request and response packets?

# Ch 11.2
### **Section 11.2: Networks** (_Computer Systems: A Programmer's Perspective_)

Section 11.2 introduces the physical hardware and software abstractions that enable computer networks, building from local area networks (LANs) up to interconnected wide-area networks (internets).

---

### **1. A Network as an I/O Device**

To an individual host computer, a network is fundamentally treated as **just another I/O device** that acts as a source and sink for data.

- **Network Adapter**: An adapter card plugged into an expansion slot on the system’s I/O bus provides the physical interface to the network wire.
- **Data Transfer**: Received network data is copied from the adapter across the I/O bus and memory bus into main memory (typically via Direct Memory Access / DMA transfers). Similarly, data in main memory is copied across the buses to the network adapter for transmission.

---

### **2. Local Area Networks (LANs)**

Physically, networks are organized hierarchically by geographical proximity. At the lowest level is a **Local Area Network (LAN)**, which typically spans a room, building, or campus.

- **Ethernet Technology**: Developed in the mid-1970s at Xerox PARC, Ethernet is the dominant LAN technology, evolving from 3 Mb/s up to 10 Gb/s and beyond.
- **Ethernet Segments**:
    - Consist of twisted-pair wires and a central connection box called a **hub**.
    - A hub blindly copies every bit it receives on one port to all other ports so that every host on the segment sees every bit.
    - Wires typically have fixed bit bandwidths (e.g., 100 Mb/s or 1 Gb/s).
- **MAC Addresses and Frames**:
    - Every Ethernet adapter has a globally unique **48-bit address** stored in nonvolatile memory.
    - Hosts communicate by sending discrete chunks of bits called **frames**.
    - An Ethernet frame consists of fixed **header bits** (identifying source/destination addresses and frame length) followed by a **payload** of data.
    - While all adapters on the segment physically observe the frame, only the designated destination host actually reads it into its memory.
- **Bridged Ethernets**:
    - Multiple Ethernet segments are joined together using **bridges** to form larger building/campus-wide LANs.
    - Wires connecting bridges to bridges or hubs can operate at different bandwidths (e.g., 1 Gb/s bridge-to-bridge vs. 100 Mb/s hub-to-bridge).
- **Conceptual Abstraction**: Programmers simplify LAN topologies by drawing hubs, bridges, and interconnecting wires as a single horizontal line.

---

### **3. Internets (Interconnected Networks)**

When multiple incompatible LANs and Wide Area Networks (WANs) need to communicate, they are connected using specialized computers called **routers** to form an **internet** (lowercase "i").

- **WANs (Wide Area Networks)**: High-speed point-to-point connections spanning larger geographical areas than LANs.
- **Incompatible Technologies**: Different LANs and WANs use radically different hardware, address formats, bit encodings, and frame packaging.
- **The Internet Protocol Software Layer**: To bridge these differences, a layer of protocol software runs on every host and router, providing two fundamental capabilities:
    1. **Uniform Naming Scheme**: Defines a standard format for host addresses. Every host is assigned at least one unique **internet address**.
    2. **Uniform Delivery Mechanism**: Defines a standard way to bundle data into discrete chunks called **packets**. A packet consists of a **header** (containing size and source/destination internet addresses) and a **payload** (the transmitted data bits).

---

### **4. How Data Travels Across an Internet: Encapsulation**

When Host A on LAN1 sends data to Host B on LAN2 through a router, the transfer follows an **8-step encapsulation process**:

1. **System Call**: Client on Host A invokes a system call copying data from its virtual address space into a kernel buffer.
2. **Encapsulation at Host A**: Host A's protocol software appends an **internet packet header (PH)** (addressed to Host B) and a **LAN1 frame header (FH1)** (addressed to the router) to the data. The internet packet is encapsulated inside the LAN1 frame.
3. **Transmission to LAN1**: LAN1 adapter copies the frame onto the LAN1 wire.
4. **Reception at Router**: Router's LAN1 adapter reads the frame from the wire and passes it to its protocol software.
5. **Routing & Re-encapsulation**:
    - The router extracts the destination internet address from the packet header (PH) and looks it up in a **routing table** to determine the next hop (LAN2).
    - The router strips off the old LAN1 frame header (FH1), prepends a new **LAN2 frame header (FH2)** (addressed to Host B), and passes the frame to its LAN2 adapter.
6. **Transmission to LAN2**: Router's LAN2 adapter copies the new frame onto the LAN2 wire.
7. **Reception at Host B**: Host B's network adapter reads the frame from the wire and passes it to its protocol software.
8. **Decapsulation at Host B**: Protocol software on Host B strips off FH2 and PH, leaving the original data, which is copied into the server process's virtual address space upon a reading system call.

---

### **5. Figures, Tables, and Asides in Section 11.2**

1. **Figure 11.2: Hardware organization of a network host**
    - **Description**: Block diagram showing a computer architecture where a **Network adapter** is plugged into an expansion slot on the I/O bus, connecting to the CPU, main memory, and system bus via an I/O bridge.
2. **Figure 11.3: Ethernet segment**
    - **Description**: Diagram showing three hosts connected via 100 Mb/s lines to a central **Hub** box.
3. **Figure 11.4: Bridged Ethernet segments**
    - **Description**: Shows a campus-wide LAN connecting multiple hubs through **Bridges X and Y**, using 100 Mb/s lines to hosts/hubs and a 1 Gb/s backbone wire between bridges.
4. **Figure 11.5: Conceptual view of a LAN**
    - **Description**: Simplified view representing an entire LAN (hubs, bridges, and wires) as a single horizontal line connected to hosts.
5. **Figure 11.6: A small internet**
    - **Description**: Illustrates an internetwork formed by two LANs and two WANs joined together by three **routers**.
6. **Figure 11.7: How data travel from one host to another on an internet**
    - **Description**: Comprehensive 8-step encapsulation diagram tracing data flow from Host A through protocol layers, LAN1 frame (`Data PH FH1`), Router translation, LAN2 frame (`Data PH FH2`), and into Host B.
7. **Aside: Internet versus internet**
    - **Description**: Clarifies terminology:
        - Lowercase **internet**: The general concept of an interconnected network of LANs and WANs.
        - Uppercase **Internet**: The specific worldwide implementation (the global IP Internet).

---

💡 **Next Step**: Would you like to proceed to **Section 11.3 (The Global IP Internet)** to review 32-bit IP addresses, network byte ordering (`htonl`/`ntohl`), domain name mappings (DNS), and socket address pairs?

---

# Section 11.3: The Global IP Internet 
from _Computer Systems: A Programmer's Perspective_ (CS:APP).

---

### **1. Conceptual Model & Protocol Architecture**

The global IP Internet is the most successful implementation of an internetwork. While its internal router and network topology is complex, the client-server application model has remained remarkably stable.

- **TCP/IP Protocol Family**: Internet hosts run software implementing the TCP/IP protocol suite:
    - **IP (Internet Protocol)**: Provides the basic naming scheme and an **unreliable datagram delivery mechanism** from host to host (does not recover from lost or duplicated packets).
    - **UDP (Unreliable Datagram Protocol)**: Extends IP slightly so datagrams travel **process-to-process** rather than host-to-host.
    - **TCP (Transmission Control Protocol)**: Builds on top of IP to provide **reliable, full-duplex (bidirectional) stream connections** between processes.
- **Three Fundamental Properties of the Internet (Programmer's View)**:
    1. The set of hosts is mapped to a set of **32-bit IP addresses**.
    2. The set of IP addresses is mapped to a set of human-friendly **Internet domain names**.
    3. A process on one Internet host can communicate with a process on any other host over a **connection**.

---

### **2. Detailed Subsection Breakdown**

#### **11.3.1 IP Addresses**

- **Format**: An IP address is an unsigned 32-bit integer.
- **`struct in_addr`**: Stored in a dedicated C structure for historical reasons.
- **Network Byte Order**: Because different host CPUs use different byte orderings (little-endian vs. big-endian), TCP/IP defines a uniform **network byte order (big-endian)** for all integer data in packet headers. Addresses in `struct in_addr` are _always_ stored in big-endian order.
- **Byte Ordering Conversion Functions (`<arpa/inet.h>`)**:
    - `htonl()`: Converts 32-bit integer from host byte order to network byte order.
    - `htons()`: Converts 16-bit integer from host byte order to network byte order.
    - `ntohl()`: Converts 32-bit integer from network byte order to host byte order.
    - `ntohs()`: Converts 16-bit integer from network byte order to host byte order.
- **Dotted-Decimal Notation**: Formatted as 4 decimal integers (0–255) separated by periods (e.g., `128.2.194.242` corresponds to `0x8002c2f2`).
- **Dotted-Decimal / Binary Conversions**:
    - **`inet_pton(AF_INET, src, dst)`**: Converts a dotted-decimal presentation string (`src`) into a binary IP address in network byte order (`dst`).
    - **`inet_ntop(AF_INET, src, dst, size)`**: Converts a binary network-byte-order IP address (`src`) into a dotted-decimal presentation string (`dst`).

#### **11.3.2 Internet Domain Names**

- **Structure**: A domain name is a sequence of words (letters, numbers, dashes) separated by periods (e.g., `whaleshark.ics.cs.cmu.edu`).
- **Hierarchy**: Organized as a tree:
    - **Unnamed Root Node**.
    - **First-Level Domains**: Managed by ICANN (e.g., `com`, `edu`, `gov`, `org`, `net`).
    - **Second-Level Domains**: Assigned on a first-come, first-served basis (e.g., `cmu.edu`, `mit.edu`, `amazon.com`).
    - **Subdomains**: Created internally by domain owners (e.g., `cs.cmu.edu`).
- **Domain Name System (DNS)**: A distributed worldwide database mapping domain names to IP addresses.
- **Mapping Mappings & Properties**:
    - **`localhost`**: Local loopback address `127.0.0.1`.
    - **One-to-One**: A single domain name maps to a single IP address.
    - **Many-to-One**: Multiple domain names map to the same IP address (e.g., `cs.mit.edu` and `eecs.mit.edu` both mapping to `18.62.1.6`).
    - **Many-to-Many**: Multiple domain names map to multiple IP addresses for load balancing (e.g., `twitter.com`).
    - **Unmapped Domains**: Some valid domain names (like top-level `edu`) do not map to any IP address.

#### **11.3.3 Internet Connections**

- **Characteristics**: Connections are **point-to-point** (connects a pair of processes), **full-duplex** (data flows simultaneously in both directions), and **reliable** (bytes arrive in the order sent).
- **Sockets & Socket Addresses**:
    - A **socket** is an endpoint of a connection.
    - A **socket address** consists of an IP address and a 16-bit integer port number, denoted as `address:port`.
    - **Ephemeral Port**: Client port assigned automatically by the kernel during a connection request.
    - **Well-Known Port**: Server port permanently bound to a service (e.g., port 80 for HTTP Web service, port 25 for SMTP email). Mappings are stored in `/etc/services`.
- **Socket Pairs**: A connection is uniquely identified by the socket pair tuple: \[(\text{cliaddr}:\text{cliport}, \text{servaddr}:\text{servport})\]

---

### **3. Figures, Tables, Code Snippets, Asides & Exercises to Reference**

Here is the complete reference list for figures, code fragments, tables, asides, and practice problems in Section 11.3:

#### **Figures**

1. **Figure 11.8: Hardware and software organization of an Internet application**
    - _Description_: Shows user-space client/server code using the Sockets interface system calls to invoke TCP/IP kernel code, which controls network adapters connected to the Global IP Internet.
2. **Figure 11.9: IP address structure**
    - _Description_: Illustrates the `struct in_addr` C definition storing a 32-bit IP address in network byte order.
3. **Figure 11.10: Subset of the Internet domain name hierarchy**
    - _Description_: Tree diagram illustrating the hierarchy from the unnamed root node down through first-level (`edu`, `com`, `gov`), second-level (`cmu`, `mit`, `amazon`), third-level (`cs`, `ece`), and individual host domain names mapped to IP addresses.
4. **Figure 11.11: Anatomy of an Internet connection**
    - _Description_: Diagrams a connection between a client host (`128.2.194.242`) on ephemeral port `51213` and a server host (`208.216.181.15`) on Web port `80`, showing the resulting socket pair `(128.2.194.242:51213, 208.216.181.15:80)`.

#### **Code Fragments & Function Prototypes**

1. **IP Address Structure**:
    
    ```
    struct in_addr {
        uint32_t s_addr; /* Address in network byte order (big-endian) */
    };
    ```
    
2. **Network Byte Order Conversions (`<arpa/inet.h>`)**:
    
    ```
    uint32_t htonl(uint32_t hostlong);
    uint16_t htons(uint16_t hostshort);
    uint32_t ntohl(uint32_t netlong);
    uint16_t ntohs(uint16_t netshort);
    ```
    
3. **Presentation / Network Conversion Functions (`<arpa/inet.h>`)**:
    
    ```
    int inet_pton(AF_INET, const char *src, void *dst);
    const char *inet_ntop(AF_INET, const void *src, char *dst, socklen_t size);
    ```
    

#### **Practice Problems**

1. **Practice Problem 11.1**: Table mapping dotted-decimal IP addresses to 32-bit hexadecimal values (e.g., `107.212.122.205` \(\leftrightarrow\) `0x6BD47ACD`, `0x00000080` \(\leftrightarrow\) `0.0.0.128`).
2. **Practice Problem 11.2**: Write a program `hex2dd.c` that converts a hex argument to a dotted-decimal string.
3. **Practice Problem 11.3**: Write a program `dd2hex.c` that converts a dotted-decimal string to hex.

#### **Sidebars & Asides**

1. **Aside: IPv4 and IPv6**: Highlights that IPv4 (32-bit addresses) remains the dominant Internet protocol despite the introduction of IPv6 (128-bit addresses) in 1996.
2. **Aside: How many Internet hosts are there?**: Notes the exponential growth from ~20,000 hosts in 1987 to over 1,000,000,000 hosts by 2015.
3. **Aside: Origins of the Internet**: Traces the history from Sputnik (1957) \(\rightarrow\) ARPANET (1969) \(\rightarrow\) Cerf & Kahn's TCP/IP (1974) \(\rightarrow\) Official launch on Jan 1, 1983 \(\rightarrow\) DNS (1985) \(\rightarrow\) Commercial backbone NSFNET retirement (1995).

---

🌐 **Next Step**: Would you like to proceed to **Section 11.4: The Sockets Interface** to review socket address structures (`sockaddr_in`), socket system calls (`socket`, `connect`, `bind`, `listen`, `accept`), and helper functions (`getaddrinfo` / `getnameinfo`)?