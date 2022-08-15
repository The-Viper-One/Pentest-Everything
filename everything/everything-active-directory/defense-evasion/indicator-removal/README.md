---
description: https://attack.mitre.org/techniques/T1070/
---

# Indicator Removal

**ATT\&CK ID:** [T1070](https://attack.mitre.org/techniques/T1070/)

**Description**

Adversaries may delete or modify artifacts generated on a host system to remove evidence of their presence or hinder defenses. Various artifacts may be created by an adversary or something that can be attributed to an adversary’s actions. Typically these artifacts are used as defensive indicators related to monitored events, such as strings from downloaded files, logs that are generated from user actions, and other data analyzed by defenders. Location, format, and type of artifact (such as command or login history) are often specific to each platform.

Removal of these indicators may interfere with event collection, reporting, or other processes used to detect intrusion activity. This may compromise the integrity of security solutions by causing notable events to go unreported. This activity may also impede forensic analysis and incident response, due to lack of sufficient data to determine what occurred.

\[[Source](https://attack.mitre.org/techniques/T1070/)]

## Sub-Techniques

### T1070.001: Clear Windows Event Logs

\<WIP>

### T1070.003: Clear Command History

\<WIP>

### T1070.004: File Deletion

\<WIP>

### T1070.005: Network Share Connection Removal

\<WIP

### T1070.006: Timestomp

\<WIP>
