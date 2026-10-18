

Openclaw Spin up a new throaway VM Prompt

I want you to spin up a new VM on PVE. If we have more than one PVE server on this lan, use whichever has the most available resources, unless i specify otherwise now at the end of the prompt

It's ID should not be PVE's defualt ### serquentially, it should be YYMM##  So if it's October 2026, 261001 would be the first ID to use that month, if 261001 was already in use use 261002 and so on. 

Use instruction from this webpage https://community-scripts.org/scripts/debian-vm to kickoff an install. if i say i needed docker, use https://community-scripts.org/scripts/docker-vm instead. you must follow it's interactive prompts. 

Select TruenasNFS for the disk storage and temp files. if this isnt available, use what the other vms are using. if this is the first vm, ask! 

Use whatever system requirements i may paste at the end of this prompt or community-scripts minimum requirements, whichever is greater. 




