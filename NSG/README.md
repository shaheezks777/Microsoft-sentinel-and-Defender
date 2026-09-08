# Azure NSG Detection Use Cases

Before using these detections, enable Azure NSG diagnostic settings and send the following log categories to Microsoft Sentinel / Log Analytics:

- `NetworkSecurityGroupEvent`
- `NetworkSecurityGroupRuleCounter`

## Planned Use Cases

- Any Source to Sensitive Management Port
- Any Source to Database Service
- Any-to-Any Inbound Rule
- Temporary or Test Allow Rule
- Rule Name and Configuration Mismatch
- High-Risk Rule Actively Matching Traffic
- Unrestricted Outbound Access

## Goal

Identify overly permissive NSG rules and improve Azure network security using Microsoft Sentinel.
