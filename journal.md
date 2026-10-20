# Lab Build Journal

This journal records the problems I encountered while building my Wazuh lab, the actions I took and the results I verified.

The screenshots are currently in my original build notes. Reviewed copies will be added to the evidence folder.

## 1. Expanding the siem-host root filesystem

**Goal:** Make the available virtual disk space usable by the operating system and Wazuh.

**Problem:** The root logical volume was approximately 100 GiB, while the volume group still had approximately 148 GiB of free space.

**Action:** I checked the storage layout with `vgs` and `lvs`, then ran:

`sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv`

**Verified result:** The logical volume increased to approximately 248 GiB. The output of `df -h /` showed a root filesystem size of 244G.

**Lesson learned:** Increasing a virtual disk does not automatically allocate all of its space to the root filesystem.

## 2. Investigating the certificate generator warning

**Goal:** Generate the certificates required by the Wazuh Docker deployment.

**Problem:** The certificate generator displayed:

`/wazuh-certs-tool.sh: line 641: find: command not found`

**Action:** I checked the generated files in `config/wazuh_indexer_ssl_certs/`. The expected certificate and key files existed and had non-zero sizes. I then started the stack.

**Verified result:** The manager, indexer and dashboard containers started and showed an Up status.

**Status:** Deployment continued successfully after checking the output. I did not demonstrate a fix for the missing `find` command inside the generator.

**Lesson learned:** I should check the actual output and subsequent service behaviour before deciding whether a warning has blocked deployment.

## 3. Running Docker Compose from the correct directory

**Goal:** Stop the Wazuh stack before taking a snapshot.

**Problem:** Running `sudo docker compose stop` from my home directory returned:

`no configuration file provided: not found`

**Action:** Later, I changed to the deployment directory:

`cd ~/wazuh-docker/single-node`

I then ran:

`sudo docker compose up -d`

**Verified result:** Docker Compose found the configuration and reported all three Wazuh containers as Running.

**Evidence limitation:** The screenshot confirms successful startup from the correct directory. It does not show a successful retry of the earlier stop command.

**Lesson learned:** I need to run Compose commands from the deployment directory or explicitly provide the configuration file.

## 4. Correcting indexer administration authentication

**Goal:** Apply the updated Wazuh indexer security configuration during the administrator password change.

**Problem:** An initial security administration command returned:

`ERR: null is not an admin user`

**Action:** I corrected the command to provide the administrator certificate and its matching private key.

**Verified result:** The output identified the administrator certificate, showed a GREEN cluster state and ended with `Done with success`.

**Lesson learned:** The security administration tool must authenticate with the appropriate administrator certificate and private key.

## 5. Fixing the Wazuh repository key import

**Goal:** Configure the Wazuh package repository on ubuntu-web.

**Problem:** The initial key import returned:

`gpg: no valid OpenPGP data found`

**Action:** I downloaded the key to `/tmp/wazuh-key` separately, checked that the file was non-empty and confirmed that it began with `BEGIN PGP PUBLIC KEY BLOCK`. I then imported it into the Wazuh keyring, adjusted its permissions and configured the repository.

**Verified result:** GPG reported that one key was imported. The subsequent package installation successfully installed Wazuh agent 4.14.9-1.

**Lesson learned:** Separating the download and import steps makes it easier to check what data a command is receiving.

## 6. Correcting the Linux agent service name

**Goal:** Enable and start the Wazuh agent on ubuntu-web.

**Problem:** I mistyped the service name as `wazuh-ahent`. Systemd reported that the unit file did not exist.

**Action:** I corrected the command:

`sudo systemctl enable --now wazuh-agent`

I then checked:

`sudo systemctl is-active wazuh-agent`

**Verified result:** The service status was `active`.

**Lesson learned:** Before reinstalling software, I should check the spelling of the service name and read the error carefully.

## 7. Restoring Windows connectivity to the Wazuh manager

**Goal:** Allow win-client to connect to the Wazuh manager at 192.168.72.10.

**Problem:** Ping succeeded, but TCP connection tests to ports 1514 and 1515 returned `TcpTestSucceeded: False`.

**Action:** On siem-host, I restarted Docker and started the Wazuh stack again:

`sudo systemctl restart docker`

`cd ~/wazuh-docker/single-node`

`sudo docker compose up -d`

I checked the containers and repeated both TCP tests from Windows.

**Verified result:** Both ports returned `TcpTestSucceeded: True`. Later screenshots showed WazuhSvc running and both ubuntu-web and win-client listed as Active in the dashboard.

**Evidence limitation:** The before-and-after screenshots show that connectivity was restored. They do not establish the complete underlying cause of the Docker networking problem.

**Lesson learned:** A successful ping confirms basic host reachability, but it does not prove that the required application ports are reachable.

## Open follow-up: Sysmon dashboard verification

I installed Sysmon, configured the Wazuh agent to read `Microsoft-Windows-Sysmon/Operational`, restarted WazuhSvc and confirmed that the service was running.

A local PowerShell query showed Sysmon Event ID 1, including a Notepad process creation event.

However, the dashboard screenshot in my build notes shows an Application event with Event ID 1001 from Windows Error Reporting. It does not yet demonstrate that Sysmon events reached Wazuh.

**Next action:** Confirm the community Sysmon configuration and capture a dashboard event showing the Sysmon channel, event ID and win-client agent name.
