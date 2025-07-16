#Daily log: (from newest to oldest)

### 2025-07-16
**Hours Worked:** 4
**From Last Time:** Build issues for the vm
**What I Did:** Solved the build issues. Solution: the files had to be commited to the local repo to survive the flake-source copy.  Immediately ran into an issue with pushing my private ssh key to the remote repo.  Solved this stupid mistake by scrubbing the history and creating new keys.  Ran into the same build issue again because my public ssh keys has not been commited to the local repo - they are in the .gitignore file.  Solution: when back and forced the commit for the public key, built the vm, then used git rm --cached to keep it local but remove it from what will be pushed.  This worked.
**Open questions:**
**Next steps:** Run the VM successfully

### 2025-07-15
**Hours Worked:** 2
**From Last Time:**
**What I Did:** Did work getting the vm configuration correct inside the flake file.  Had a lot of errors regarding the build.  Did not solve this build issue.
**Open questions:** What is causing the build issue
**Next steps:** Keep working on the build

### 2025-07-11
**Hours Worked:** 3
**From Last Time:** Started on the VM on my PC
**What I Did:** Decided what resources to allocate to the VM - 8vCPU and 20 GiB, based off current resources available.  Installed QEMU.  Added Account to KVM.  Currently setting up a TAP + Bridge.
**Open questions:** Ran in to questions about the LAN set up for the VM and Pi.  Want to consult mentor before I continue. 
**Next steps:** Continue getting the VM set up

### 2025-07-10
**Hours Worked:**  5.5
**From Last Time:** Continued with flakes - decided to switch all systems to nix flakes
**What I Did:** Split my nix files into their own seperate remote repo.  Spent the most of the day making the transfer from plain nix on the pi over to a flake.  There were issues with sytax, git and the ssh key.  Had a big issue with the ssh key and flake.lock file.  Currently a I have had a successful flake rebuild.  This has removed git from the system unfortunately.
**Open questions:**
**Next steps:** Adding git back to the nixos-flake folder.  Creating a VM on my pc using QEMU.

### 2025-07-09
**Hours Worked:**  3
**From Last Time:**  Continued working on learning about flakes.
**What I Did:** Learned put Nix Flakes and created first one to handle the Pi and VM on PC.  Added public config ssh key to secret file. Solved minor issue with github and ssh again (learned more in the process).
**Open questions:**  Do I create the VM on my PC first or connect pi peripherals first.
**Next steps:** Get pi environment set up.  Get PC envirnment set up.

### 2025-07-03
**Hours Worked:**  3
**From Last Time:** --
**What I Did:** Got git and github set up.  Set up ssh keys for github.  Created config file inside /.ssh for github. Decided with GPT that a hybrid approach to Nix, using both plain on the pi and flake on my pc, is the approach that I will go with.
**Open questions:** --
**Next steps:** Learning about nix flakes so I can integrate it on with QEMU on my PC.

### 2025-07-02
**Hours Worked:** 1.5 
**From Last Time:** Finished the first milestone of the project - Getting NixOS on the pi with custom config. Successfully rebuild.
**What I Did:** Cleaned house. Orginiazed project for next steps.  Began learning about Nix Flakes. Compiled a master list of milestones in project.
**Open questions:** --
**Next steps:** Learn more about Flakes and if it is the right move to switch over.

### 2025-06-27
**Hours Worked:** 3 
**From Last Time:** There were two issues DNS on the Pi and a miss configured masquerade
**What I Did:** Worked with gpt to figure out where the issue from yesterday was.  I knew that the dhs was a part of it.  The second issue was discovered through trouble shooting.  I had my first successful rebuild.
**Open questions:** Now what do I do.
**Next steps:** Figuring out the next chunk of the project.  Where to go next.

### 2025-06-26
**Hours Worked:** 3 
**From Last Time:** Did realise that the error was from lack of internet, Nix is trying to do something related to the Nix store.
**What I Did:** Tried and failed to us my PC as a bridge for the pi to access the internet.  Was able to ping 8.8.8.8. Which is a partial success, but I was unable to access anything with a web address.  Have since learned that this might mean that there is an issue with the dns.
**Open questions:** Why is there an issue between NixOS and the DNS settings.  Have been told that this came up with my mentor as well.
**Next steps:** Tackle the last hurdle of using my PC as a bridge.  Then immediatly switch to using a switch to connect my pi and PC to the internet.  I want to know both - so I will do it both ways.

### 2025-06-25 
**Hours Worked:** 5.5
**From Last Time:** Did use a moniter and keyboard just to find the ip address.  After this was able to ssh in from PC - did use nixos-generate-config but did not us chroot to move forward (this is because I did it on the pi and not my PC).
**What I Did:**  
- 1. Finished flashing a base version of NixOS 25.05 onto the pi.
  - Did not use Nix generate to generate a config on my pc.  
- 2. Used a moniter to get ip address and change the root password.
- 3. Connect to pi via ssh portal
- 4. Generated a generic hardware-configuration file
  - `nixos-generate-config --show-hardware-config > /etc/nixos/hardware-configuration.nix`
- 5. Attempted to add custom configuration file.
  - Ran into multiple problems with the initial file - syntax error
- 6. Attempted to rebuild multiple times - still trouble shooting.
**Open questions:**  Did I fix syntax errors?  Are we having issues because of lack of internet connection to the pi?
**Next steps:** See if I can fix the current error that is causing the rebuild to fail

### 2025-06-24
**Hours Worked:** 4.5
**From Last Time:**: Restarted the whole process of flashing the pi and creating the config file over from scratch.  
**What I Did:**  Worked with GPT to set up an initial configuration file for the pi. Tackles networking, timezone, hostname, ssh attributes and public key, GPIO fan and UART attribute.  Began reflashing NixOS 25.05 to the SDCard.  Was prepared to put the custom configuration.nix in the file system where it was recommended before but this is not possible.  I was missinformed and would need to either boot once on pi and then replace it - which is not possible over ssh or I would have to run a NixOS generator on my pc to generate the file on the SDCard.
**Open questions:**  What is the best approach.  Using a moniter and booting first on the pi is easiest. But, it was not the challenge presented when I was given this project...
**Next steps:** Attempt to generate the files on the SDCard using nixos-generate-config and chroot.

