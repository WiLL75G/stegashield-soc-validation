
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
