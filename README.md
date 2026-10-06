# StegaShield Detection Validation Pilot

## Overview

I built this pilot to answer one main question:

**Can StegaShield distinguish known clean images from images I intentionally modify using steganography?**

The goal is not simply to make the product generate an alert.

I want to understand what the model detects, where it performs well, where it does not, and how useful its signal becomes when investigated alongside endpoint, network, and application evidence.

This repository documents that process from environment baseline to final findings.

## Why I Built This

A network connection can show that an image was transferred.

That does not prove the image contains hidden data.

A detection alert can show that something looks suspicious.

That does not automatically prove data exfiltration occurred.

That distinction is what I want to investigate.

Instead of treating one alert as the answer, I am establishing known ground truth first and then comparing StegaShield's results with the surrounding telemetry.

## Research Question

Can StegaShield distinguish known clean images from images intentionally modified using steganography under controlled conditions?

I will also investigate:

* How probability scores differ between clean and modified images
* Correct and incorrect classifications
* False positives and false negatives
* Repeatability across controlled tests
* Behaviour outside the model's stated training scope
* Whether additional telemetry improves analyst confidence

## Lab Architecture

The pilot uses my existing M2 Mac home lab.

### Windows VM

Simulated corporate endpoint.

Used to store and transfer the controlled image samples.

Telemetry includes Sysmon and Windows event logs.

### Ubuntu 24.04 VM

Controlled server environment.

Planned components include:

* HTTPS receiver
* StegaShield
* Zeek

### Mac M2

SOC analysis platform.

Splunk Enterprise is used to investigate and correlate telemetry from the environment.

## Investigation Flow

```text
Windows Activity
      |
Endpoint Telemetry
      |
HTTPS Transfer
      |
Network Telemetry
      |
Received Image
      |
StegaShield Analysis
      |
Splunk Correlation
      |
Analyst Interpretation
```

Each layer answers a different question.

No individual signal will be treated as proof of exfiltration by itself.

## Ground Truth

Ground truth is established before an image is submitted to StegaShield.

StegaShield does not decide whether my test sample is clean or modified.

I establish that independently.

Each sample will be documented with information such as:

* Test ID
* Parent image
* Filename
* Ground truth
* SHA256 hash
* File format
* Dimensions
* File size
* Image source
* Embedding method
* Payload preparation
* Payload size
* Tool and version
* StegaShield analysis ID
* Probability score
* Classification
* Timestamp
* Analyst notes

This allows every result to be traced back to a known sample.

## Validation Scope

The primary validation focuses on Least Significant Bit steganography because this is the stated training scope for the model being evaluated.

Testing begins with clean controls and controlled LSB samples.

Additional techniques may later be tested separately to explore behaviour outside the stated training distribution.

Those results will be treated as generalization testing rather than equivalent to the primary LSB validation.

## Investigation Method

I am using the same investigation process I use throughout my SOC lab work:

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
Tuning
Variation
Repeat
```

Throughout the pilot I separate four important levels of analysis.

### Observed

What the evidence directly shows.

### Correlated

What becomes visible when multiple evidence sources are connected.

### Interpretation

What I believe the evidence means.

### Unknown

What the available evidence cannot establish.

This distinction is important because a detection signal is evidence for investigation, not automatically proof of malicious activity.

## Pilot Plan

### Day 1: Environment Baseline

Document the existing Windows, Ubuntu, Mac, Splunk, Sysmon, Docker, firewall, network, and background service state before introducing the pilot components.

### Day 2: Dataset and Ground Truth

Prepare the controlled dataset and establish independent ground truth before submitting samples to StegaShield.

### Day 3: StegaShield and Clean Baseline

Deploy StegaShield and establish how known clean images are scored before introducing controlled steganographic samples.

### Day 4: LSB Validation

Test controlled LSB samples against their known clean counterparts and record the resulting probability scores and classifications.

### Day 5: Error Analysis and Generalization

Investigate incorrect classifications, false positives, false negatives, repeatability, and behaviour outside the stated training scope.

### Day 6: HTTPS and SOC Investigation

Introduce the controlled HTTPS transfer path and correlate endpoint, network, application, and StegaShield evidence from an analyst perspective.

### Day 7: Reproduction and Findings

Reproduce important results, document limitations and evidence gaps, review configuration, and produce the final findings.

## Repository Structure

```text
stegashield-soc-validation/
│
├── README.md
│
├── investigations/
│   └── stegashield-pilot/
│       ├── day-01-baseline.md
│       ├── day-02-dataset-ground-truth.md
│       ├── day-03-clean-baseline.md
│       ├── day-04-lsb-validation.md
│       ├── day-05-error-analysis.md
│       ├── day-06-soc-investigation.md
│       └── day-07-findings.md
│
├── evidence/
│   ├── day-01/
│   ├── day-02/
│   ├── day-03/
│   ├── day-04/
│   ├── day-05/
│   ├── day-06/
│   └── day-07/
│
└── dataset/
    └── ground-truth.csv
```

## What I Am Not Claiming

A high StegaShield probability does not automatically prove malicious activity.

An image upload does not automatically prove steganography.

Steganography does not automatically prove data exfiltration.

A missed technique outside the model's stated training distribution does not automatically mean the product failed.

The final conclusion will be based on the evidence collected during the pilot.

## Status

**Pilot in progress**

| Day | Investigation | Status |
| --- | --- | --- |
| Day 1 | Environment baseline | Complete |
| Day 2 | Dataset and ground truth | Pending |
| Day 3 | StegaShield and clean baseline | Pending |
| Day 4 | LSB validation | Pending |
| Day 5 | Error analysis and generalization | Pending |
| Day 6 | HTTPS and SOC investigation | Pending |
| Day 7 | Reproduction and findings | Pending |

## Core Principle

**Telemetry is not detection, and detection is not automatically proof of malicious activity.**

My role as the analyst is to understand what each source can prove, correlate the evidence, identify what remains unknown, and make a defensible conclusion.
