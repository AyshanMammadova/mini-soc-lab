# Mini SOC Lab

## Overview

I built this lab to practise deploying Wazuh and collecting security events from Linux and Windows endpoints. The scope of this repository is Week 1: building the lab and deploying Wazuh.

**Status:** The initial deployment is working. Documentation and the remaining validation tests are in progress.

## Lab environment

The virtual machines run in VMware Workstation on my Windows laptop.

| Machine | Role | Operating system | Lab IP address |
| --- | --- | --- | --- |
| siem-host | Wazuh server running in Docker | Ubuntu Server LTS | 192.168.72.10 |
| ubuntu-web | Linux endpoint and intended web server | Ubuntu Server LTS | 192.168.72.20 |
| win-client | Windows endpoint | Windows 10 | 192.168.72.30 |

The lab uses the VMware VMnet1 host-only network, 192.168.72.0/24, with static addresses.

The intended final network configuration keeps a NAT adapter on siem-host for downloads. Endpoint NAT adapters are temporary and must be removed after software installation. Final isolation verification is pending.

## Wazuh deployment

- Wazuh version used during setup: **4.14.9**.
- Deployment: single-node Docker Compose stack on siem-host.
- Central components: Wazuh manager, Wazuh indexer and Wazuh dashboard.
- Endpoint components: Wazuh agents on ubuntu-web and win-client.
- Dashboard address from the lab laptop: `https://192.168.72.10`.
- Sysmon is installed on win-client. Community-configuration and dashboard validation remain to be completed.

## Validation still to document

- [ ] Confirm the VM resource allocations and clocks.
- [ ] Capture the running Wazuh containers and both agents showing Active.
- [ ] Confirm Sysmon uses a community configuration and show a Sysmon event in Wazuh.
- [ ] Open the ubuntu-web default web page from the laptop.
- [ ] Find all five required test events in Wazuh and record their rule descriptions, levels and evidence filenames.
- [ ] Explain the path of one event from the endpoint to the dashboard.
- [ ] Verify that agents and alerts persist after a stack stop/start and a siem-host reboot.
- [ ] Verify the indexer kernel setting after reboot.
- [ ] Record the idle memory use of each Wazuh container.
- [ ] Confirm that endpoints have no permanent internet route.
- [ ] Record existing snapshots and take the final snapshots after validation.
- [ ] Complete the startup and shutdown instructions, architecture files, journal and repository secret check.

## Test event evidence

The following table will be completed using observed alerts, rather than assumed rule values.

| Test event | Rule description | Rule level | Evidence filename |
| --- | --- | --- | --- |
| Failed SSH login | Pending | Pending | Pending |
| Command run with sudo | Pending | Pending | Pending |
| New Linux local user | Pending | Pending | Pending |
| Windows program start captured by Sysmon | Pending | Pending | Pending |
| Failed Windows logon | Pending | Pending | Pending |

## Documentation to add

- `architecture/`: a diagram of the actual lab and a table of addresses and VM resources.
- `evidence/`: reviewed screenshots and command outputs, with descriptions of what they prove.
- `journal.md`: at least five entries covering actions, problems, fixes and results.
- `.gitignore`: exclusions for credentials, certificates, private keys and VM files.

Passwords, keys and certificates must not be committed. Screenshots must be checked for visible credentials before upload. VM snapshot files stay on the laptop; only screenshots of their lists belong in the evidence folder.
