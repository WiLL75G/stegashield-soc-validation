# Day 1: Environment Baseline

## Objective

Before installing StegaShield or Zeek, creating steganographic samples, or configuring the HTTPS workflow, I documented the existing state of my lab.

The objective was to establish what already existed before introducing new components.

This gives me a known starting point for later comparison and helps prevent existing services, traffic, or configuration from being incorrectly attributed to the StegaShield pilot.

## Scope

The Day 1 baseline covers three systems:

* Mac M2 host running Splunk Enterprise
* Windows endpoint used as the simulated corporate workstation
* Ubuntu 24.04 server used as the controlled server environment

I am documenting:

* System resources
* Operating system state
* Network configuration
* Existing security telemetry
* Splunk forwarding and ingestion
* Existing services
* Docker state
* Firewall configuration
* Background network activity
* Time configuration
* Connectivity between systems

## Baseline Principle

I am not trying to detect steganography on Day 1.

I am establishing what normal and inherited activity already exists in the environment before controlled testing begins.

This gives me a baseline for answering a simple question later:

**Did this activity exist before the pilot, or did it appear because of something I introduced?**

---

## Mac Host Baseline

The Mac is the host system for my existing lab and also runs Splunk Enterprise for SOC investigation and correlation.

Before continuing with the pilot, I documented the host resources and current system state.

### Observed

* Model: MacBook Air
* Chip: Apple M2
* CPU: 8 cores
* Memory: 8 GB
* macOS: 27.0.1
* Build: 26A434
* Primary interface: en0
* IPv4 address: 192.168.146.66

At the time of the baseline, both the Windows and Ubuntu VMs were already running. I recorded the host resources so I can compare resource usage later as additional pilot components are introduced.

Sensitive hardware identifiers such as the serial number and hardware UUID are intentionally excluded from the public documentation.

### Analyst Interpretation

The Mac is not being treated as a StegaShield detection source.

Its main role in this pilot is to provide the existing lab environment and host Splunk for investigation and correlation.

Documenting its state gives me a reference point if resource or connectivity problems appear after additional pilot components are introduced.

---

## Splunk Baseline and Troubleshooting

Splunk Enterprise is the central investigation platform for this pilot.

Before relying on Windows telemetry during later testing, I needed to verify that the existing forwarding path was actually operational.

### Expected

Windows telemetry should be forwarded from the Splunk Universal Forwarder to Splunk Enterprise on TCP port 9997 and remain searchable from the SOC platform.

### What I Found

During the baseline, Splunk was not running on the Mac.

This meant the Windows endpoint could not establish its expected connection to TCP port 9997.

The investigation followed this sequence:

```text
Expected: Windows telemetry reaches Splunk
        ↓
Observed: Splunk was not running
        ↓
Effect: TCP 9997 connectivity failed
        ↓
Action: Started Splunk
        ↓
Verification: TCP 9997 listening
        ↓
Windows connection established
        ↓
Telemetry searchable in Splunk
```

### Change Made

I started Splunk Enterprise during the baseline.

This was a deliberate state change and is documented because the environment after this action was different from the environment I initially observed.

After Splunk started, TCP port 9997 was listening and the Windows Universal Forwarder established a connection to the Splunk receiver.

### Verification

I verified the forwarding path from both sides.

The Windows endpoint established a TCP connection to the Splunk receiver, and telemetry from `JAMES-VM` was searchable in Splunk.

The available telemetry included:

* Windows Security
* Sysmon
* PowerShell
* System
* Application
* Microsoft Defender

### Evidence 1: Windows Telemetry Reaching Splunk

![Windows telemetry reaching Splunk](../../evidence/day-01/01-splunk-windows-telemetry.png)

This screenshot provides evidence that Windows telemetry from `JAMES-VM` was searchable in Splunk after the forwarding path was restored.

### Analyst Interpretation

This confirms the existing Windows telemetry pipeline is operational:

```text
Windows
   ↓
Splunk Universal Forwarder
   ↓
TCP 9997
   ↓
Splunk Enterprise
```

It does not prove anything about StegaShield, HTTPS image transfers, or Zeek.

Those components have not been introduced into the controlled workflow yet.

---

## Windows Endpoint Baseline

The Windows VM represents the simulated corporate endpoint in this pilot.

Before creating or transferring controlled image samples, I documented the endpoint's operating system, resources, network configuration, and existing security telemetry.

### Observed

The baseline identified:

* Hostname: JAMES-VM
* Windows 10 Home
* Version: 25H2
* OS build: 26200.9550
* Logical processors: 4
* Memory: approximately 4 GB
* Primary IPv4 address: 192.168.64.17
* Default gateway: 192.168.64.1

The system also had existing security monitoring components from previous lab work.

### Evidence 2: Windows System Baseline

![Windows endpoint system baseline](../../evidence/day-01/02-windows-system-baseline.png)

This screenshot establishes the Windows system state before controlled StegaShield testing.

### Sysmon Baseline

Sysmon was already installed and running before the StegaShield pilot.

Existing telemetry included process creation and file creation events.

The Sysmon configuration also had network connection monitoring enabled with filtering rather than unrestricted collection.

This is important because I should not assume that every network connection generated by the endpoint will automatically appear in Sysmon.

### Evidence 3: Sysmon Baseline

![Windows Sysmon baseline](../../evidence/day-01/03-windows-sysmon-baseline.png)

This screenshot documents the existing Sysmon state before controlled image testing begins.

### Splunk Universal Forwarder

The Splunk Universal Forwarder was already installed and running.

Its existing configuration sends telemetry to:

```text
192.168.64.1:9997
```

After resolving the Splunk receiver issue documented earlier, the endpoint established a connection to the receiver and Windows telemetry became searchable in Splunk.

### Analyst Interpretation

The Windows endpoint already had an established telemetry pipeline before controlled StegaShield testing began.

This gives me endpoint evidence that can later be correlated with image creation, process execution, file activity, and transfer activity.

However, the existence of Sysmon and Splunk telemetry does not mean every future action will automatically be visible.

The actual telemetry generated during controlled testing must still be verified rather than assumed.

---

## Ubuntu Server Baseline

The Ubuntu VM is the controlled server environment for the StegaShield pilot.

Before installing StegaShield or Zeek, I documented its existing system, network, service, firewall, and container state.

### Observed

The baseline identified:

* Ubuntu 24.04.5 LTS
* Kernel: 6.8.0-146-generic
* Architecture: ARM64
* CPU: 4 virtual CPUs
* Memory: approximately 4.3 GiB
* Primary interface: enp0s1
* IPv4 address: 192.168.64.12
* Docker already installed
* No existing Docker containers
* UFW enabled
* Apache already listening on TCP port 80
* SSH available on TCP port 22
* Existing Samba services
* Existing Wazuh agent

StegaShield and Zeek had not yet been introduced into the controlled pilot environment.

### Evidence 4: Ubuntu Network and Server Baseline

![Ubuntu server baseline](../../evidence/day-01/04-ubuntu-server-baseline.png)

This screenshot establishes the Ubuntu server and network state before the pilot components are introduced.

### Existing Firewall State

UFW was already active with a default policy of denying incoming connections and allowing outgoing connections.

This became visible during connectivity testing.

Windows could reach Ubuntu at the IP layer, but a connection attempt to TCP port 80 failed even though Apache was listening.

The failure was consistent with the existing UFW policy rather than evidence that the two VMs could not communicate.

I did not open TCP port 80 simply to make the baseline test succeed.

Changing the firewall at this stage would have altered the environment unnecessarily.

### Inherited Wazuh Activity

The Ubuntu VM also contained a Wazuh agent from previous lab work.

The agent was configured to contact:

```text
192.168.64.1:1514
```

The baseline showed repeated failed connection attempts to the old Wazuh manager configuration, including enrollment attempts on TCP port 1515.

No corresponding Wazuh manager listener was available on the Mac during the baseline.

### Analyst Interpretation

The Ubuntu VM is not a completely clean server.

It contains inherited services, firewall rules, and background Wazuh activity that existed before the StegaShield pilot.

This matters because those services can later appear in network telemetry.

Rather than immediately removing them, I documented them first so I can distinguish inherited activity from traffic introduced by the pilot.

The repeated Wazuh connection attempts are therefore treated as known background noise unless later evidence shows otherwise.

---

## Network Connectivity Validation

Before building the HTTPS transfer workflow, I needed to establish basic connectivity between the Windows endpoint and Ubuntu server.

The objective was not to prove that the future application workflow works.

The objective was to answer a simpler question:

**Can the two systems currently reach each other at the network layer?**

### Ubuntu to Windows

From Ubuntu, I tested connectivity to the Windows endpoint at:

```text
192.168.64.17
```

The test returned four replies with zero packet loss.

### Windows to Ubuntu

Connectivity testing from Windows to Ubuntu also confirmed that the Ubuntu server at:

```text
192.168.64.12
```

was reachable at the IP layer.

A separate TCP connection attempt to Ubuntu port 80 failed.

That result did not contradict the successful IP connectivity test because UFW was already configured to block unsolicited inbound access to that port.

### Evidence 5: Ubuntu to Windows Connectivity

![Ubuntu to Windows connectivity](../../evidence/day-01/05-ubuntu-to-windows-connectivity.png)

The screenshot shows four ICMP replies from the Windows endpoint with zero packet loss.

### Observed

* Ubuntu could reach Windows at 192.168.64.17.
* Windows could reach Ubuntu at 192.168.64.12 at the IP layer.
* Ubuntu returned four ICMP replies with zero packet loss.
* TCP port 80 was not reachable from Windows under the existing firewall policy.

### Interpretation

The evidence establishes bidirectional IP connectivity between the two VMs.

It does not establish that HTTPS works.

It does not establish that StegaShield works.

It does not establish that image transfers work.

Those are separate layers that will be introduced and validated later.

This distinction prevents me from treating basic network reachability as proof that the complete application workflow is operational.

---

## Time Synchronization Baseline

Reliable timestamps are important for correlation.

Later in the pilot, I will need to compare activity recorded by Windows, Ubuntu, Splunk, Zeek, and StegaShield.

Before relying on those timestamps, I checked the existing time configuration.

### Windows

Windows was configured for Pacific time with daylight saving time active.

During the baseline, the Windows Time service initially reported an unhealthy synchronization state.

The system later successfully contacted `time.windows.com`, and Windows recorded successful synchronization activity.

However, the status output continued to report indicators that did not fully match the successful synchronization evidence.

### Ubuntu

Ubuntu was configured for:

```text
Africa/Accra
UTC +00:00
```

The system reported that its clock was synchronized and network time synchronization was active.

### Observed

* Windows successfully contacted its configured time source.
* Windows recorded successful time synchronization activity.
* Some Windows time status indicators remained inconsistent with that evidence.
* Ubuntu reported synchronized system time.
* The systems use different local time zones.

### Interpretation

I should not compare displayed local timestamps across these systems without accounting for their different time zones.

For the pilot, investigation timestamps will be normalized to UTC when correlating events across multiple systems.

The remaining inconsistency in the Windows time status is documented as an evidence gap rather than being treated as fully resolved.

### Evidence Gap

Windows produced evidence of successful synchronization while some status fields continued to indicate an abnormal synchronization state.

I will therefore avoid claiming that Windows time synchronization was completely healthy based only on the successful synchronization event.

---

## Changes Made During the Baseline

Although the purpose of Day 1 was primarily observation, several changes were made while validating and troubleshooting the existing environment.

I am documenting those changes separately so I can distinguish the original state from the state after troubleshooting.

### Splunk Enterprise Started

Splunk Enterprise was initially not running on the Mac.

This prevented the Windows Universal Forwarder from establishing its expected connection to TCP port 9997.

I started Splunk Enterprise and then verified that:

* TCP port 9997 was listening.
* The Windows endpoint could reach the receiver.
* The Universal Forwarder established a connection.
* Windows telemetry was searchable in Splunk.

This was a corrective change required to restore the existing telemetry pipeline.

### Mac Terminal Configuration

The `top` command on the Mac was being replaced by an existing shell alias that launched `btop`.

I identified the alias in the shell configuration and commented it out.

A new shell confirmed that `top` resolved to the native macOS command.

This was a local shell configuration change and did not affect the pilot detection architecture.

### Ubuntu Hostname

The Ubuntu VM still used the hostname:

```text
wazuh-manager
```

That name represented an older role from previous lab work and no longer accurately described the system's purpose.

I changed the hostname to:

```text
ubuntu
```

This was an administrative naming change.

It did not remove or reconfigure the existing Wazuh agent.

### Ubuntu Shell Prompt

The Ubuntu shell also contained a customized prompt from previous SOC lab work.

I changed it to a standard prompt to reduce inherited cosmetic configuration.

This change did not affect network traffic, security telemetry, or StegaShield testing.

### Change Control Interpretation

These changes are part of the Day 1 record.

They should not be confused with the original baseline state.

The important distinction is:

```text
Original state
      ↓
Problem or inherited configuration identified
      ↓
Controlled change
      ↓
Verification
      ↓
New known state
```

Recording this sequence gives me a defensible starting point for Day 2 rather than pretending the environment was unchanged throughout the baseline.

---

## Day 1 Analysis

### Observed

The Day 1 evidence directly shows that:

* The Windows endpoint is using 192.168.64.17.
* The Ubuntu server is using 192.168.64.12.
* Windows and Ubuntu can communicate at the IP layer.
* Sysmon was already running on the Windows endpoint.
* The Splunk Universal Forwarder was already installed and running.
* Splunk Enterprise was initially not running.
* After Splunk was started, TCP port 9997 became available and the Windows forwarding connection was restored.
* Windows telemetry became searchable in Splunk.
* Docker was already installed on Ubuntu with no existing containers.
* UFW was already active on Ubuntu.
* Apache, SSH, Samba, and the Wazuh agent were inherited from previous lab activity.
* The Wazuh agent was repeatedly attempting to contact an unavailable previous manager.
* StegaShield had not yet been introduced into the controlled workflow.
* Zeek had not yet been introduced into the controlled workflow.

### Correlated

When the evidence is viewed together, I can establish an existing telemetry path:

```text
Windows Endpoint
      |
      | Existing endpoint telemetry
      v
Splunk Universal Forwarder
      |
      | TCP 9997
      v
Splunk Enterprise
```

I can also establish basic network reachability between the systems that will later participate in the controlled transfer workflow:

```text
Windows
192.168.64.17
      |
      | IP connectivity
      |
Ubuntu
192.168.64.12
```

### Interpretation

The environment is ready to move into dataset and ground truth preparation, but it is not a clean environment.

Several inherited services and configurations already generate activity.

That background state must be considered when later reviewing network and endpoint telemetry.

The baseline also demonstrates why I should validate each layer independently.

Basic connectivity does not prove HTTPS works.

Windows telemetry reaching Splunk does not prove image activity will be visible.

An image transfer does not prove steganography.

A StegaShield signal will not automatically prove exfiltration.

Each claim requires its own supporting evidence.

### Unknown

Day 1 does not establish:

* Whether StegaShield can distinguish clean and modified images.
* What probability scores StegaShield will produce.
* Whether clean images will generate elevated scores.
* Whether controlled LSB samples will be detected.
* Whether StegaShield results are repeatable.
* Whether Zeek will provide useful network context.
* Whether the planned HTTPS transfer workflow operates correctly.
* Whether all relevant endpoint activity will be captured by the existing Sysmon configuration.
* Whether StegaShield signals will improve analyst confidence when correlated with other telemetry.

These questions belong to later stages of the pilot.

### Evidence Gaps

The Windows time service produced evidence of successful synchronization while some status indicators remained inconsistent with that result.

The existing Sysmon configuration also applies filtering to network connection telemetry, so future network activity cannot be assumed to appear in Sysmon without verification.

These gaps are documented rather than treated as resolved.

---

## Day 1 Disposition

**Disposition: Proceed to Day 2**

The environment baseline is established well enough to begin controlled dataset and ground truth preparation.

No steganography detection conclusion is being made from Day 1.

The next stage will establish known clean and intentionally modified sample identities independently of StegaShield.

## Day 1 Lesson

The most important lesson from Day 1 was that a baseline is more than recording IP addresses and system specifications.

The environment already contained inherited services, firewall behaviour, background Wazuh traffic, existing telemetry, and a broken Splunk receiver state.

Documenting those conditions before testing gives me a reference point for separating existing behaviour from activity introduced by the pilot.
