
```
Run this https://nckfst.us/18
```

I want you to spin up a new VM on my Proxmox. If I have more than one PVE server on this LAN, use whichever has the most available resources, unless I specify otherwise at the end of this prompt.

Use the instructions from https://community-scripts.org/scripts/debian-vm?utm_source=nickfost.com to start the installation. If I say I need Docker, use https://community-scripts.org/scripts/docker-vm?utm_source=nickfost.com instead.

You must follow the interactive prompts during installation or it won't work. You need to follow the advanced config route.

Select the latest Debian unless I specify otherwise.

Its virtual machine ID should not use PVE's default sequential numbering (###). Instead, it should follow a YYMM## format. For example, if it's October 2026, 261001 would be the first ID used that month. If 261001 is already in use or you have a record of it previously being used, use 261002, and so on. I prefer these because PVE will reassign old numbers, which causes headaches with backups.

If you can reach 10.0.0.16, you're probably on Nick Foster's LAN and need to use 10.0.0.16:3142 for the apt-cache prompt in the community-scripts installer. if 10.0.0.16 isn't available, dont worry about apt-cache

If you can ping 10.0.0.25, you're probably on Nick Foster's Lan and need to Select TruenasNFS for disk storage and temporary files. If this isn't available, use what the other VMs are using. If this is the first VM, ask!

If my disk is full, STOP and tell me. Don't guess what to do.

Use whatever system requirements i may paste at the end of this prompt or community-scripts minimum requirements, whichever is greater.

When asked "Start VM when completed?" by the community-scripts installer, select No.

Once community-scripts finishes creating the VM, configure Cloud-Init BEFORE the VM's first boot if the selected image uses Cloud-Init.

During installation, select Cloud-Init Yes when required, username root, password password, DHCP networking, and no SSH public keys. Do not accept a randomly generated password or invent some strategy to keep the password out of the transcripts, we dont care, this is a throwaway. 

After the installer finishes, use the Proxmox host shell to apply these Cloud-Init settings. Substitute the actual VMID assigned during installation.

```

VMID=REPLACE_WITH_ACTUAL_VMID
qm set "$VMID" --ciuser root --cipassword 'password' --ipconfig0 ip=dhcp --ciupgrade 0
qm set "$VMID" --delete nameserver,searchdomain,sshkeys
qm cloudinit update "$VMID"

```

Verify that the Proxmox Cloud-Init configuration matches these settings:

User: root
Password: password
DNS domain: Use host settings
DNS servers: Use host settings
SSH public key: none
Upgrade packages: No
IP Config (net0): ip=dhcp

Verify the Cloud-Init drive is attached and inspect the configuration using:

```

VMID=REPLACE_WITH_ACTUAL_VMID
qm config "$VMID"
qm cloudinit dump "$VMID" user
qm cloudinit dump "$VMID" network

```

Do not start the VM until all Cloud-Init settings are correct.

Once verified, start the VM using:

```
qm start "$VMID"
```

After starting the VM, perform all guest configuration through QEMU Guest Agent directly from the Proxmox host. Do not use xterm.js, noVNC, or interactive console login unless QEMU Guest Agent fails.

community-scripts normally attempts to install QEMU Guest Agent in the image before booting, but verify that it is operational.

Use the actual VMID:

Wait for the guest agent to respond:

```
VMID=REPLACE_WITH_ACTUAL_VMID
qm guest cmd "$VMID" ping
```

If unavailable, retry periodically for up to 3 minutes.

If the agent remains unavailable, investigate the installation. Use offline image customization or the Proxmox console only as a fallback.

Once the guest agent responds, wait for Cloud-Init:

```
qm guest exec "$VMID" --timeout 600 -- cloud-init status --wait
```

If Cloud-Init is not installed because a NoCloud image was selected, skip this command.

Check that Cloud-Init completed successfully before proceeding.

Next, if you are on my lan and you can ping 10.0.0.25 earlier, execute my initialization script inside the VM through QEMU Guest Agent:

```
qm guest exec "$VMID" --timeout 600 -- bash -lc 'set -o pipefail; (command -v curl >/dev/null && curl -fsSL https://gitlab.randomdevelopment.biz/-/snippets/44/raw || wget -qO- https://gitlab.randomdevelopment.biz/-/snippets/44/raw) | sed "s/\r$//" | bash'
```

Then explicitly set the root password to password, regardless of whether Cloud-Init already configured it:

```
qm guest exec "$VMID" --timeout 60 -- bash -lc 'printf "root:password\n" | chpasswd && passwd -S root'
```

Install OpenSSH:

```
qm guest exec "$VMID" --timeout 900 -- bash -lc 'apt-get update && apt-get install -y openssh-server'
```

Enable root password authentication:

```
qm guest exec "$VMID" --timeout 120 -- bash -lc 'mkdir -p /etc/ssh/sshd_config.d && printf "PermitRootLogin yes\nPasswordAuthentication yes\n" > /etc/ssh/sshd_config.d/00-throwaway-root-login.conf && sshd -t && systemctl restart ssh'
```

Verify the effective SSH settings:

```
qm guest exec "$VMID" --timeout 60 -- bash -lc "sshd -T | grep -E '^(permitrootlogin|passwordauthentication) '"
```

Both settings must be yes. Investigate any conflicting SSH configuration.

IMPORTANT: Verify the exit code of every guest-agent command. An exitcode of 0 indicates successful completion.

If a command returns a PID because it exceeded the synchronous wait timeout, use:

```
qm guest exec-status "$VMID" PID
```

Substitute the actual returned PID. Continue checking until the command exits. Do not start dependent commands until the previous command completes successfully.

Retrieve the VM's IP address:

```
qm guest cmd "$VMID" network-get-interfaces
```

Verify that the VM has a valid DHCP address and working network connectivity.

Test SSH login from the Proxmox host or another LAN machine using root and password.

Once SSH is verified, prepare to remove Cloud-Init.

First, verify that the guest has a persistent DHCP network configuration that will work without Cloud-Init. Identify the actual network interface and network configuration method. Do not assume networking will survive removal.

Disable future Cloud-Init execution inside the VM:

```
qm guest exec "$VMID" --timeout 60 -- touch /etc/cloud/cloud-init.disabled
```

Shut down the VM using the Proxmox host:

```
qm shutdown "$VMID" --timeout 120
```

Wait until the VM is fully stopped:

```
qm wait "$VMID" --timeout 120
```

Inspect the VM hardware configuration and identify the Cloud-Init drive. Remove only that drive using the Proxmox host. Do not remove the main system disk or any other storage.

Restart the VM:

```
qm start "$VMID"
```

Verify QEMU Guest Agent responds again:

```
qm guest cmd "$VMID" ping
```

Verify that the VM retains DHCP networking and the correct IP address. Confirm root SSH password authentication still works.

Do not consider provisioning complete unless networking, root login, and SSH all work after the restart.

Use the Proxmox host and QEMU Guest Agent for all subsequent guest configuration, including APT caching where possible.

Only use xterm.js or noVNC when the guest agent is unavailable and other recovery methods have failed. Do not unnecessarily open an interactive console.

Do not remove any ordinary system disks.

If community-scripts already started the VM before the configuration was applied, investigate whether Cloud-Init already executed. Regenerating Cloud-Init after the first boot may not apply the new settings.

If the selected VM image does not use Cloud-Init, skip the Cloud-Init commands and configure root access and SSH directly inside the guest instead.

Do not consider provisioning complete until root login, SSH, and networking have been verified after a reboot.

Root password authentication using password is intended only for temporary VMs on my trusted LAN. Do not expose SSH to the public Internet.

Once complete, report the IP address, MAC address, RAM, CPU, VMID, username, and password to me.

Also time how long this took. Tell me at the end how long it took and what took the longest. 

If you're a big-brain LLM, provide feedback on how this prompt could be improved.
