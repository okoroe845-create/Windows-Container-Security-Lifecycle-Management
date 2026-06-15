# Windows Container Security & Lifecycle Management Lab

A hands-on technical lab demonstrating the deployment, network auditing, privilege assessment, and lifecycle management of Windows-based containers. This project walks through running an interactive session inside a lightweight Microsoft NanoServer isolation environment on Windows Server 2019, performing system administration task sequences, and conducting network connectivity validations.

This lab simulates real-world infrastructure security hardening and auditing tasks for a corporate environment (**Structureality Inc.**).

---

## 💻 Environment Overview

* **Host Operating System:** Windows Server 2019 Standard
* **Container Base Image:** `mcr.microsoft.com/windows/nanoserver:1809` (Size: ~252MB)
* **Container Runtime:** Docker Enterprise Subsystem
* **Target Domain Context:** `ad.structureality.com`
* **Lab Objective:** Isolate application deployments, audit internal configurations, and verify boundary defenses using native administrative utilities.

---

## 🛠️ Implementation Workflow & Command Logs

### 1. Host Infrastructure Auditing
Before initiating container runtimes, the host network interface configuration and locally cached container layers are verified to determine the network baseline.

```powershell
# Check Host IP configuration and network adapters
ipconfig

# Verify OS version details and the physical machine hostname context
winver
hostname

# Enumerate local image repository layers to confirm NanoServer availability
docker images
```

<img width="1019" height="589" alt="Screenshot From 2026-06-15 09-54-18" src="https://github.com/user-attachments/assets/4c7e22af-6382-460f-98bc-9a2af75d15c1" />


**Key Baseline Metrics Captured:**
* **Host IPv4 Address:** `10.1.1.13`
* **Host Subnet Mask:** `255.255.255.0`
* **Host Default Gateway:** `10.1.1.254`
* **Host Name:** `MS10`
* **Cached Base Image:** `mcr.microsoft.com/windows/nanoserver:1809` (Image ID: `bd7fe7d0dddd`)

---

### 2. Container Provisioning & Lifecycle States
Deploying standalone test instances using explicit naming schemas, tracking state changes across the container lifecycle, and spawning interactive command line shells.

```powershell
# Provision a named container instance in a stopped state from the base image
docker create --name MyFirstContainer [mcr.microsoft.com/windows/nanoserver:1809]

# Query status metrics and verify the creation of the inactive container
docker ps -a

# Change the lifecycle state of the container to active (started)
docker start MyFirstContainer

# Re-verify container status to observe the execution uptimes
docker ps -a

# Instantiate and attach to a real-time, interactive command interface wrapper (runtime sandbox)
docker run -it --name TestContainer [mcr.microsoft.com/windows/nanoserver:1809]
```

<img width="1019" height="589" alt="Screenshot From 2026-06-15 09-54-30" src="https://github.com/user-attachments/assets/c0b036ee-7659-4e97-9d27-6bc17f18f99d" />


**Lifecycle Observations:**
* `docker create` yields a 64-character long SHA-256 process token and puts the container in the `Created` status.
* `docker start` changes the status to `Up X seconds`.
* The interactive flag combo `-it` overrides the default entrypoint to drop the console operator directly into `C:\>` inside the isolated subsystem.

---

### 3. Network Isolation & Boundary Auditing
Inspecting the containerized network interface properties, validating port allocations, and running ping operations to verify structural routing rules through the default NAT bridge framework.

```cmd
:: Read internal container adapter assignments from inside the sandbox
ipconfig

:: Map current socket connection bindings and listening process IDs (PIDs)
netstat -aon
```

<img width="1019" height="589" alt="Screenshot From 2026-06-15 09-54-41" src="https://github.com/user-attachments/assets/145c4b07-8367-41de-b978-2abc5be82c45" />


```cmd
:: Validate connectivity out to the container internal NAT gateway switch
ping 192.168.96.1

:: Validate cross-boundary path connectivity out to the physical lab host machine
ping 10.1.1.13
```

<img width="1019" height="589" alt="Screenshot From 2026-06-15 09-54-49" src="https://github.com/user-attachments/assets/59690d43-c637-4b9b-b9b8-79dadef0e34b" />


<img width="1019" height="589" alt="Screenshot From 2026-06-15 09-54-54" src="https://github.com/user-attachments/assets/66a724cf-386f-409b-9cc4-7c1dfda13ed2" />


**Network Telemetry Data:**
* **Container Internal IP Address:** `192.168.105.226`
* **Container Subnet Mask:** `255.255.240.0`
* **Container Gateway Router:** `192.168.96.1`
* **Active Listening Sockets:** Ports `135` (RPC), `49152-49155` (RPC Dynamic Ports), and `5353/5355` (LLMNR/mDNS).
* **Ping Statistics:** 0% packet loss across both the default gateway and physical host boundaries, proving successful NAT bridge encapsulation.

---

### 4. Privilege Verification & Account Hardening
Auditing default user accounts inside the active runtime session, altering target credential states, and establishing granular local group memberships.

```cmd
:: Identify current execution context runtime token parameters
echo %username%

:: List active user profiles embedded natively in the container image
net user
```



```cmd
:: Audit specific administrative account constraints and security policies
net user Administrator
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-02-01" src="https://github.com/user-attachments/assets/0a19a44a-b383-4304-ba00-728e5d6d76b9" />


```cmd
:: Enumerate active local system security groups inside the container
net localgroup
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-02-14" src="https://github.com/user-attachments/assets/31a7ac49-0eac-4128-a213-be6c11e33892" />


```cmd
:: Create a secondary unprivileged user and map it to specific operational roles
net localgroup "Power Users" testuser /add
net user testuser
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-04-40" src="https://github.com/user-attachments/assets/7dc1d6b8-c038-43fc-a566-e70009815dc0" />


```cmd
:: Adjust administrative credentials to meet internal password complexity baselines
net user Administrator pa$$word123 /passwordchg:yes
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-04-46" src="https://github.com/user-attachments/assets/13984faf-86ab-4003-8339-92a6436b5691" />


**Security Evaluation:**
* Default runtime interactive user context resolves to `ContainerUser` (least-privilege model enforced by Microsoft NanoServer default policies).
* Built-in accounts discovered: `Administrator`, `DefaultAccount`, `Guest`, `WDAGUtilityAccount`.
* The custom account `testuser` was successfully generated and verified as a concurrent member of both the standard `*Users` and `*Power Users` groups.
* Administrative control parameters were restricted using `/passwordchg:yes`.

---

### 5. Storage Isolation & Local File Operations
Validating localized storage persistence constraints by provisioning explicit directory pathways and evaluating isolated disk input/output loops.

```cmd
:: Review the root drive layout structure and available space metrics
Dir

:: Generate a specific testing data partition folder
md MyFolder
cd MyFolder

:: Write target configuration/telemetry data streams into a localized file
echo "This is some text for my test file." > test.txt
Dir
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-06-23" src="https://github.com/user-attachments/assets/f029392f-012b-4518-9ce9-7ca4a3b445b9" />


```cmd
:: Read the raw generated file content back to the console screen for validation
type test.txt
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-06-28" src="https://github.com/user-attachments/assets/b98fadae-5af3-4488-bd62-ef9e27136c21" />


**Storage Metrics:**
* Volume Serial Number inside the container sandbox: `140D-2D29`.
* The isolated layer safely compiled `test.txt` containing exactly 40 bytes of plain text without touching or modifying the underlying host disk image space.

---

### 6. Clean Lifecycle Teardown
Exiting the sandbox wrapper cleanly and shutting down the running container processes from the host terminal to conserve compute infrastructure resources.

```cmd
:: Break the active container session wrapper loop and drop back to host control
exit
```

```powershell
# From the host terminal, explicitly terminate the active runtime process state
docker stop TestContainer
```

<img width="804" height="605" alt="Screenshot From 2026-06-15 10-08-06" src="https://github.com/user-attachments/assets/37617b6c-45ec-4329-9ee9-40b15e203a00" />


---

## 🔒 Key Security & Architectural Takeaways

1. **Lightweight Attack Surface Minimalization:** Utilizing minimalist base layers like Microsoft NanoServer (~252MB) drastically restricts the binary tooling matrix available to malicious actors post-exploitation compared to standard Windows Server Core installations.
2. **Strict Network Segregation Mechanics:** The container sandbox boundary transparently isolates processes via a virtual NAT system switch layer. This enables fine-grained rule control over inbound socket access while allowing egress testing to physical host networks.
3. **Credential Management Baseline Hardening:** Modifying default local accounts and configuring explicit least-privilege profiles (`ContainerUser`) inside active container environments ensures proper boundary defenses and satisfies continuous infrastructure compliance requirements.
