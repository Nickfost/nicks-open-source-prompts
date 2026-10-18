
```
Run this https://nckfst.us/18
```

I want you to spin up a new VM on my Proxmox. If I have more than one PVE server on this lan, use whichever has the most available resources, unless i specify otherwise now at the end of the prompt.

Use the instructions from https://community-scripts.org/scripts/debian-vm?utm_source=nickfost.com to start the installation. If I say I need Docker, use https://community-scripts.org/scripts/docker-vm?utm_source=nickfost.com instead. 

You must follow the interactive prompts during installation or it won't work, You need to follow the advanced config route. 

Select the latest debian unless i specified otherwise.

Its virtual machine ID should not use PVE's default sequential numbering (###). Instead, it should follow a YYMM## format. For example, if it's October 2026, 261001 would be the first ID used that month. If 261001 is already in use or you have a record of it previously in use, use 261002, and so on. I prefer these because PVE will reassign old numbers, which makes headaches with backups. 

if you can reach 10.0.0.16, you're probably on Nick Foster's lan and you need to use 10.0.0.16:3142 for the apt-cache-ng prompt on the community-script installer.

Select TruenasNFS for the disk storage and temp files. if this isnt available, use what the other vms are using. if this is the first vm, ask! 

if my disk is full, STOP and tell me, dont guess what what to do.

Use whatever system requirements i may paste at the end of this prompt or community-scripts minimum requirements, whichever is greater.

Once the initial installation using the Community Scripts script is complete, if it's an Ubuntu VM or requires cloud-init, follow the instructions here to set a username and password: https://github.com/community-scripts/ProxmoxVE/discussions/272?utm_source=nickfost.com. Log in, set the root user's password to "password", enable SSH login for root, and remove the defined cloud-init configuration, or clean it up so it doesnt interupt again.

After the installation is fully complete, follow the instructions at https://apt.nickfost.net to set up apt-cacher-ng. (this wont work on any other lan besides nick's lan)

Once complete report the IP address, MAC address, RAM, CPU, VMID, username, and password to me.

Also time how long this took. Tell me (at the end) how long it took and what took the longest. 

If you're a high-cost LLM model, provide feedback on how this prompt could be improved.
