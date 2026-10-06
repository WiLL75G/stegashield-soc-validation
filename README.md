# StegaShield SOC Validation Pilot

This repository documents my hands on evaluation of StegaShield from a SOC analyst and detection engineering perspective.

The goal is not simply to see whether the tool produces an alert.

I want to understand what the signal means, what evidence supports it, where the detection has limitations, and how useful that signal becomes when correlated with endpoint, network, and application telemetry.

## Research Question

Can StegaShield distinguish known clean images from images I intentionally modify using steganography?

From there, I will look at:

* Probability scores
* Correct and incorrect classifications
* Repeatability
* False positives and false negatives
* Behavior outside the stated training scope
* SOC investigation value when the result is correlated with other telemetry

## Why I Am Testing This

My previous image transfer lab taught me an important lesson.

Seeing an image move across a network does not tell me whether that image contains hidden data.

In that lab, Suricata gave me network visibility and I later created a rule that searched for a marker I already knew existed.

That was useful, but it was not the same as detecting unknown steganography.

This pilot takes the investigation one layer deeper by testing content aware analysis.

## Lab Architecture

The pilot uses my existing M2 Mac home lab.

```text
Windows Endpoint
      |
      | HTTPS
      v
Ubuntu Server
      |
      | Content Analysis
      v
StegaShield

Endpoint Telemetry: Sysmon
Network Telemetry: Zeek
SOC Platform: Splunk
```

### Windows

Windows acts as the simulated corporate endpoint.

It will hold the clean and controlled modified images and generate the endpoint activity used during the investigation.

### Ubuntu

Ubuntu acts as the controlled server environment.

It will host the server side components required for the pilot, including StegaShield and later network monitoring.

### Splunk

Splunk is my central investigation platform.

The objective is to correlate evidence rather than treat one detection source as the complete answer.

## Validation Approach

I am separating ground truth from detection.

Before an image is submitted to StegaShield, I should already know whether that image is clean or intentionally modified.

StegaShield does not define my ground truth.

This allows me to compare what I know about the sample against what the model reports.

The main validation begins with LSB steganography because that is the stated training scope of the model.

Testing outside that scope will be treated separately as generalization testing rather than expected detection behavior.

## Investigation Method

I am using the same investigation process I have been building across my SOC labs:

```text
Baseline
Controlled Activity
Expected Telemetry
Actual Telemetry
Hypothesis
Triage
Investigation
Correlation
Interpretation
Evidence Gaps
Disposition
Detection Review
Variation
Repeat
```

Throughout the pilot I will separate:

**Observed**

What the evidence directly shows.

**Correlated**

What multiple evidence sources show when viewed together.

**Interpretation**

What I believe the evidence means.

**Unknown**

What the available evidence cannot prove.

This distinction matters because a high probability score is a content signal. It is not automatically proof of malicious activity or data exfiltration.

# Day 1: Environment Baseline

I started the pilot by documenting the environment before installing StegaShield or Zeek and before creating any steganographic samples.

I wanted to know what already existed in the lab before introducing new components.

That gives me something to compare against later.

## Windows Endpoint

The Windows VM was documented before controlled testing.

The baseline included:

* Operating system and build
* CPU and memory
* Storage
* Network configuration
* Sysmon status and configuration
* Splunk Universal Forwarder status
* Existing telemetry

![Windows system baseline](evidence/day-01/01-windows-system-baseline.png)

Sysmon was already active and Splunk telemetry from the endpoint was searchable.

This is important because later activity can be investigated using telemetry that existed before StegaShield was introduced.

![Windows telemetry in Splunk](evidence/day-01/02-splunk-windows-telemetry.png)

## Ubuntu Server

Ubuntu was also documented before installing the pilot components.

The VM is using `192.168.64.12` on the lab network.

I recorded its resources, storage, network interfaces, existing services, firewall configuration, Docker state, installed components, and existing background activity.

![Ubuntu network baseline](evidence/day-01/03-ubuntu-network-baseline.png)

One useful lesson from this stage was that the server was not a completely clean system.

It already contained services and inherited configuration from previous lab work.

Instead of removing those immediately, I documented them first.

That matters because existing activity can later appear in network telemetry and could otherwise be mistaken for pilot activity.

## Connectivity Validation

The Windows endpoint and Ubuntu server must be able to communicate before I build the HTTPS transfer path.

Ubuntu successfully reached the Windows endpoint at `192.168.64.17` with four ICMP replies and zero packet loss.

![Ubuntu to Windows connectivity](evidence/day-01/04-windows-ubuntu-connectivity.png)

This does not prove that the future HTTPS workflow works.

It proves something more basic:

The two systems currently have IP connectivity.

That distinction is important because each layer should be validated separately.

## Day 1 Findings

The baseline established that:

* Windows and Ubuntu can communicate at the network layer.
* Sysmon is already producing endpoint telemetry.
* Windows telemetry is reaching Splunk.
* Docker already exists on Ubuntu.
* Ubuntu contains inherited services and configuration that must be considered during testing.
* StegaShield has not yet been introduced into the controlled test workflow.
* Zeek has not yet been introduced into the controlled test workflow.

No steganography detection conclusions can be made from Day 1.

That is intentional.

Day 1 is about knowing the environment before changing it.

## Current Architecture

```text
Windows VM
192.168.64.17
      |
      | Network connectivity confirmed
      |
Ubuntu VM
192.168.64.12

Windows
   |
Sysmon
   |
Splunk Universal Forwarder
   |
Splunk
```

The next stages will introduce ground truth, controlled image samples, StegaShield analysis, and eventually the HTTPS and network telemetry layers.

## Pilot Progress

| Day | Focus | Status |
|---|---|---|
| Day 1 | Environment baseline | Complete |
| Day 2 | Dataset and ground truth | Pending |
| Day 3 | Clean image baseline | Pending |
| Day 4 | LSB validation | Pending |
| Day 5 | Error analysis and generalization | Pending |
| Day 6 | HTTPS and SOC investigation | Pending |
| Day 7 | Reproduction and findings | Pending |

## Core Principle

The main lesson I am carrying through this pilot is simple:

**Telemetry is not detection, and detection is not automatically proof of malicious activity.**

My job as the analyst is to understand what each source can prove, correlate the evidence, identify what remains unknown, and make a defensible conclusion.
