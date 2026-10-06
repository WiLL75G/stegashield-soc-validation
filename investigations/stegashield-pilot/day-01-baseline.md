
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

The host had sufficient resources to continue the existing lab, but resource usage matters because the Windows and Ubuntu VMs were already running during the baseline.

Sensitive hardware identifiers such as the serial number and hardware UUID are intentionally excluded from the public documentation.

### Analyst Interpretation

The Mac is not being treated as a detection source for StegaShield.

Its main role in this pilot is to provide the existing lab environment and host Splunk for investigation and correlation.

Documenting its state gives me a reference point if resource or connectivity problems appear after additional pilot components are introduced.
