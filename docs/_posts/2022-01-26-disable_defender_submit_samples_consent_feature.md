---
title: "Disable Defender Submit Samples Consent Feature"
excerpt: "Disable or Modify Tools, Impair Defenses"
categories:
  - Endpoint
last_modified_at: 2022-01-26
toc: true
toc_label: ""
tags:
  - Disable or Modify Tools
  - Defense Evasion
  - Impair Defenses
  - Defense Evasion
  - Splunk Enterprise
  - Splunk Enterprise Security
  - Splunk Cloud
  - Endpoint
---



[Try in Splunk Security Cloud](https://www.splunk.com/en_us/cyber-security.html){: .btn .btn--success}

#### Description

his analytic is to detect a suspicious modification of registry to disable windows defender feature. This technique is to bypassed or evade detection from Windows Defender AV product specially the submit samples feature for further analysis..

- **Type**: [TTP](https://github.com/splunk/security_content/wiki/Detection-Analytic-Types)
- **Product**: Splunk Enterprise, Splunk Enterprise Security, Splunk Cloud
- **Datamodel**: [Endpoint](https://docs.splunk.com/Documentation/CIM/latest/User/Endpoint)
- **Last Updated**: 2022-01-26
- **Author**: Teoderick Contreras, Splunk
- **ID**: 73922ff8-3022-11ec-bf5e-acde48001122


#### [ATT&CK](https://attack.mitre.org/)

| ID             | Technique        |  Tactic             |
| -------------- | ---------------- |-------------------- |
| [T1562.001](https://attack.mitre.org/techniques/T1562/001/) | Disable or Modify Tools | Defense Evasion |

| [T1562](https://attack.mitre.org/techniques/T1562/) | Impair Defenses | Defense Evasion |

#### Search

```

| tstats `security_content_summariesonly` count from datamodel=Endpoint.Registry where Registry.registry_path = "*\\Microsoft\\Windows Defender\\SpyNet*" Registry.registry_value_name = SubmitSamplesConsent Registry.registry_value_data = 0x00000000 by _time span=1h Registry.dest Registry.user Registry.registry_path Registry.registry_value_name Registry.registry_value_data Registry.process_guid 
| `drop_dm_object_name(Registry)` 
|rename process_guid as proc_guid 
|join proc_guid, _time [
| tstats `security_content_summariesonly` count FROM datamodel=Endpoint.Processes by _time span=1h Processes.process_id Processes.process_name Processes.process Processes.dest Processes.parent_process_name Processes.parent_process Processes.process_guid 
| `drop_dm_object_name(Processes)` 
|rename process_guid as proc_guid 
| fields _time dest user parent_process_name parent_process process_name process_path process proc_guid registry_path registry_value_name registry_value_data] 
| table _time dest user parent_process_name parent_process process_name process_path process proc_guid registry_path registry_value_name registry_value_data 
| `disable_defender_submit_samples_consent_feature_filter`
```

#### Macros
The SPL above uses the following Macros:
* [security_content_summariesonly](https://github.com/splunk/security_content/blob/develop/macros/security_content_summariesonly.yml)

Note that `disable_defender_submit_samples_consent_feature_filter` is a empty macro by default. It allows the user to filter out any results (false positives) without editing the SPL.

#### Required field
* _time
* Registry.dest
* Registry.user
* Registry.registry_value_name
* Registry.registry_key_name
* Registry.registry_path
* Registry.registry_value_data


#### How To Implement
To successfully implement this search, you need to be ingesting logs with the registry value name, registry path, and registry value data from your endpoints. If you are using Sysmon, you must have at least version 6.0.4 of the Sysmon TA.

#### Known False Positives
admin or user may choose to disable windows defender product

#### Kill Chain Phase
* Exploitation



#### RBA

| Risk Score  | Impact      | Confidence   | Message      |
| ----------- | ----------- |--------------|--------------|
| 49.0 | 70 | 70 | modified/added/deleted registry entry $Registry.registry_path$ in $dest$ |




#### Reference

* [https://thedfirreport.com/2021/10/18/icedid-to-xinglocker-ransomware-in-24-hours/](https://thedfirreport.com/2021/10/18/icedid-to-xinglocker-ransomware-in-24-hours/)



#### Test Dataset
Replay any dataset to Splunk Enterprise by using our [`replay.py`](https://github.com/splunk/attack_data#using-replaypy) tool or the [UI](https://github.com/splunk/attack_data#using-ui).
Alternatively you can replay a dataset into a [Splunk Attack Range](https://github.com/splunk/attack_range#replay-dumps-into-attack-range-splunk-server)

* [https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/malware/icedid/disable_av/sysmon.log](https://media.githubusercontent.com/media/splunk/attack_data/master/datasets/malware/icedid/disable_av/sysmon.log)



[*source*](https://github.com/splunk/security_content/tree/develop/detections/endpoint/disable_defender_submit_samples_consent_feature.yml) \| *version*: **2**