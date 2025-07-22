# Commands Log

## 2025-06-24

| System | Directory | Command                                                                           | Description                    | Notes/Output                                    |
| ------ | --------- | --------------------------------------------------------------------------------- | ------------------------------ | ----------------------------------------------- |
| Ubuntu | ~         | `wget https://…img.zst`                                                           | Download NixOS SD-card image   | already done on previous iteration **see notes  |
| Ubuntu | ~         | `zstd -d https://…img.zst`                                                        | decompress disk image          | already done on previous iteration  **see notes |
| Ubuntu | ~         | `lsblk`                                                                           | check mount                    | sd cards auto-mount upon insertion              |
| Ubuntu | ~         | `sudo umount /media/kimdev/FIRMWARE /media/kimdev/NIXOS_SD`                       | un mounting the previous image |                                                 |
| Ubuntu | ~         | `sudo dd if=<file_name> of=/dev/mmcblk0 bs=4M status=progress conv=fsync && sync` | writing to sd card             | **see notes                                     |
| Ubuntu | ~         | `sudo mkdir -p /mnt/pi-root`                                                      | creating mounting partition    | already done on previous iteration              |
| Ubuntu | ~         | `sudo mkdir -p /mnt/pi-boot`                                                      | creating mounting partition    | already done on previous iteration              |
| Ubuntu | ~         | `sudo mount /dev/mmcblk0p1 /mnt/pi-boot`                                          | mounting the partition         | already done on previous iteration              |
| Ubuntu | ~         | `sudo mount /dev/mmcblk0p2 /mnt/pi-root`                                          | mounting the partition         | already done on previous iteration              |
**Notes:** 
  - full address for compressed image
    1. wget https://hydra.nixos.org/build/298477550/download/1/nixos-image-sd-card-25.05.802746.7282cb574e06-aarch64-linux.img.zst
    2. nixos-image-sd-card-25.05.802746.7282cb574e06-aarch64-linux.img

## 2025-06-25
| System  | Directory | Command                                                                                | Description                  | Notes/Output                                    |
| ------- | --------- | -------------------------------------------------------------------------------------- | ---------------------------- | ----------------------------------------------- |
| Ubuntu  | ~         | `ssh-keygen -t ed25519 -C "pi-admin"`                                                  | creating public key          | already done on previous iteration ** see notes |
| Ubuntu  | ~         | `cat ~/.ssh/id_ed25519.pub`                                                            | print key to terminal        | already done on previous iteration              |
| Ubuntu  | ~         | `sudo umount /mnt/pi-root /mnt/pi-boot`                                                |                              |                                                 |
| Ubuntu  | ~         | `lsblk`                                                                                | list block devices           | sanity check                                    |
| Nix     | ~         | `ip addr show`                                                                         | List IP address on sys       | Need to know IP address to SSH to               |
| Nix     | ~         | `sudo -i`                                                                              | change to root user          | Need to change password                         |
| Nix     | ROOT      | `passwd root`                                                                          | setting new password         | Temp password created - will disable later      |
| Ubuntu  | ~         | `ssh-keygen -f ~/.ssh/known_hosts -R 169.254.148.87`                                   | clear out old key            | **see notes                                     |
| Ubuntu  | ~         | `ssh root@<pi-ip-address>`                                                             | travel over ethernet to pi   | Enter password here of course                   |
| ssh Nix | ROOT      | `nano etc/nixos/configuration.nix`                                                     | creating config file         | I created a custom config                       |
| ssh Nix | ROOT      | `nixos-generate-config --root /  `                                                     | run generator                | To create the hardware-config file              |
| ssh Nix | ROOT      | `nixos-generate-config --show-hardware-config > /etc/nixos/hardware-configuration.nix` | creating the hardware config |                                                 |
| ssh Nix | ROOT      | `nixos-rebuild switch`                                                                 | rebuild system               | recieved many errors - lucky me  **see notes    |
**Notes:**
  - I had to clear out a stale host key from my last try.
  - Had errors because of the configuration.nix file and lack of internet connection.
  - Left command in because it is because it is correct if you have connection to a router - I do not.

## 2025-06-26
| System  | Directory | Command                                                        | Description                   | Notes/Output                                  |
| ------- | --------- | -------------------------------------------------------------- | ----------------------------- | --------------------------------------------- |
| Ubuntu  | ~         | `sudo sysctl -w net.ipv4.ip_forward=1`                         | Enabling IP forwarding        | internet access for the pi thru my pc         |
| Ubuntu  | ~         | `sudo iptables -t nat -A POSTROUTING -o wlp59s0 -j MASQUERADE` | Masquerading outbound traffic | internet access for the pi thru my pc         |
| Ubuntu  | ~         | `nmcli device status`                                          | network manager               | changing ethernet address                     |
| Ubuntu  | ~         | `sudo nmcli connection modify "Wired connection 1" \`          | Modifying existing interface  | Start command here **see notes                |
|         |           | `ipv4.method manual \`                                         |                               | **see notes                                   |
|         |           | `ipv4.addresses 192.168.4.1/24 \`                              |                               | **see notes                                   |
|         |           | `ipv4.gateway "" \`                                            |                               | **see notes                                   |
|         |           | `ipv4.dns 1.1.1.1,8.8.8.8 \`                                   |                               | **see notes                                   |
|         |           | `ipv4.ignore-auto-dns yes`                                     |                               | end command                                   |
| Ubuntu  | ~         | `sudo nmcli connection down "Wired connection 1"`              | bringing link down            | this is to apply changes                      |
| Ubuntu  | ~         | `sudo nmcli connection up   "Wired connection 1"`              | bringing link back up         | this is to apply changes                      |
| Nix     | ~         | `ip addr show end0`                                            | showing pi's ip address       | Trouble with ssh IP addresses are missaligned |
| Nix     | ~         | `sudo ip addr add 192.168.4.2/24 dev end0`                     | assigning static IP           |                                               |
| Nix     | ~         | `sudo ip link set dev end0 up`                                 | ensuring IP is up             |                                               |
| Nix     | ~         | `sudo ip route add default via 192.168.4.1 dev end0`           | adding default gateway        |                                               |
| ssh Nix | ROOT      | `ping -c 3 8.8.8.8`                                            | RAW IP test                   | Success                                       |
| ssh Nix | ROOT      | `ping -c 3 cache.nixos.org`                                    | DNS Test                      | Failed                                        |  |

**Notes:**  
  - It is assumed that if I am working on the pi that I have already run the command to ssh in.
  - I already had a network interface set up from previous tries but for a new one:
    - `sudo nmcli connection add type ethernet ifname enx00e04c68018d con-name <name> autoconnect yes ipv4.method manual ipv4.addresses 192.168.4.1/24 ipv4.gateway "" ipv4.dns "1.1.1.1,8.8.8.8" ipv4.ignore-auto-dns yes`
  - I have broken the whole command into seperate lines for readability - should be entered in one chunck like the above example with nmcli add.
sudo nmcli connection modify "Wired connection 1" ipv4.method shared

## 2025-06-27
| System  | Directory | Command                                            | Description                          | Notes/Output                      |
| ------- | --------- | -------------------------------------------------- | ------------------------------------ | --------------------------------- |
| ssh Nix | ROOT      | `sudo cp /etc/resolv.conf /etc/resolv.conf.backup` | making back up of resolv.conf        | just incase                       |
| ssh Nix | ROOT      | `echo "nameserver 8.8.8.8" >> /etc/resolv.conf`    | appending line to end of resolv.conf | add `sudo sh -c` if not root user |
| ssh Nix | ROOT      | `nixos-rebuild switch`                             | initiating an update                 |                                   |

**Notes:**

## 2025-07-03
| System | Directory                       | Command                                                                                          | Description                   | Notes/Output                               |
| ------ | ------------------------------- | ------------------------------------------------------------------------------------------------ | ----------------------------- | ------------------------------------------ |
| Ubuntu | ~                               | `scp root@192.168.4.2:/etc/nixos/configuration.nix Documents/yellow_ball_project/nixos`          | secure copy protocol          | for git                                    |
| Ubuntu | ~                               | `scp root@192.168.4.2:/etc/nixos/hardware-configuration.nix Documents/yellow_ball_project/nixos` | secure copy protocol          | for git                                    |
| Ubuntu | ~/Documents/yellow_ball_project | `git init`                                                                                       | create intial repo            |                                            |
| Ubuntu | ~/Documents/yellow_ball_project | `git add `                                                                                       | add new files                 |                                            |
| Ubuntu | ~/Documents/yellow_ball_project | `git status`                                                                                     | checking what is staged       |                                            |
| Ubuntu | ~/Documents/yellow_ball_project | `git commit -m "<**see notes>"`                                                                  | commiting new files with note |                                            |
| Ubuntu | ~/Documents/yellow_ball_project | `git branch -m master main`                                                                      | chabging branch name to main  |                                            |
| Ubuntu | ~/Documents/yellow_ball_project | `ssh-keygen -t ed25519 -C "kim-lancaster@users.noreply.github.com" -f ~/.ssh/id_ed25519_github`  | creating ssh key for repo     | has passphrase                             |
| Ubuntu | ~/Documents/yellow_ball_project | `git remote add origin git@github.com:kim-lancaster/yellow-ball-project.git`                     |                               |                                            |
| Ubuntu | ~/Documents/yellow_ball_project | `cat ~/.ssh/id_ed25519_github.pub`                                                               | print key to terminal         | add to github                              |
| Ubuntu | ~/Documents/yellow_ball_project | `sudo nano ~/.ssh/config`                                                                        | creating a config file        | github need it so it know which key to use |
| Ubuntu | ~/Documents/yellow_ball_project | `sudo chmod 600 ~/.ssh/config`                                                                   | changing permissions          | GPT recommended but not sure if necessary  |
| Ubuntu | ~/Documents/yellow_ball_project | `eval "$(ssh-agent -s)"`                                                                         | start agent                   | **see notes                                |
| Ubuntu | ~/Documents/yellow_ball_project | `ssh-add ~/.ssh/id_ed25519_github`                                                               | add private key               | **see notes                                |
| Ubuntu | ~/Documents/yellow_ball_project | `ssh -T git@github.com`                                                                          | connecting over ssh           | verifying key match                        |
| Ubuntu | ~/Documents/yellow_ball_project | `git push -u origin main`                                                                        | push to github                |                                            |

**Notes:**
  - "Initial project structure and import of my log files and NixOS config files."
  - Had to add public key to Github
  - I have to run `ssh-add ~/.ssh/id_ed25519_github` everytime I open a new shell or it I can't push to github.

## 2025-07-10
| System  | Directory                     | Command                                                                                                  | Description                 | Notes/Output             |
| ------- | ----------------------------- | -------------------------------------------------------------------------------------------------------- | --------------------------- | ------------------------ |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `git init`                                                                                               | *This following git init    |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `git add `                                                                                               | *was for a seperate repo    |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `git status`                                                                                             | *that contained just the    |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `git commit -m "<message>"`                                                                              | *nix config and flake       |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `git branch -m master main`                                                                              | *files in keeping with the  |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `ssh-keygen -t ed25519 -C "kim-lancaster@users.noreply.github.com" -f ~/.ssh/id_ed25519_github`          | *industry standard          | has passphrase           |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `git remote add origin git@github.com:kim-lancaster/yellow-ball-nixos.git`                               | *                           |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `eval "$(ssh-agent -s)"`                                                                                 | *                           |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `ssh-add ~/.ssh/id_ed25519_github`                                                                       | *                           |                          |
| Ubuntu  | ~/Documents/yellow_ball_nixos | `ssh -A root@192.168.4.2`                                                                                | ssh with forwarding agent   | so I can ssh into github |
| ssh Nix | /etc                          | `nix-shell -p git --run "git clone git@github.com:kim-lancaster/yellow-ball-nixos.git /etc/nixos-flake"` | cloning repo to nixos-flake |                          |
| ssh Nix | ROOT                          | `nixos-rebuild switch`                                                                                   | rebuild to add flakes       |                          |
| ssh Nix | ROOT                          | `nix flake show /etc/nixos-flake`                                                                        | syntax check                | sanity check             |
| ssh Nix | ROOT                          | `sudo nixos-rebuild switch --flake /etc/nixos-flake#ybp-pi`                                              | flake rebuild               | FAILED **see notes       |
| ssh Nix | /etc/nixos-flake              | `nix flake update --recreate-lock-file`                                                                  | updagte flake.lock          | because of half build    |
| ssh Nix | /etc/nixos-flake              | `sudo nixos-rebuild switch --flake .#ybp-pi`                                                             | flake rebuild               | inside flake folder      |

**Notes:**
  - Before the rebuild there were modifications made to the configuration.nix file so that flakes could be used moving forward.
  - There was some error in the above, you do not need to run the flake update if the flake rebuilds correctly the first time.
  - As you can see, the all Nix files were moved to their own repo.

## 2025-07-11
| System  | Directory | Command                                                                                       | Description                                                         | Notes/Output                       |
| ------- | --------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ---------------------------------- |
| ssh Nix | ROOT      | `du -s /nix/store \| numfmt --to=iec`                                                         | checking disk space of nix/store                                    | **see notes                        |
| Ubuntu  | ~         | `lscpu ...`                                                                                   | checking cpu virtualization                                         | **see notes                        |
| Ubuntu  | ~         | `nproc`                                                                                       | checking number of cpu cores                                        | for VM **see notes1                |
| Ubuntu  | ~         | `uptime`                                                                                      | check how long system up and load                                   | for VM **see notes2                |
| Ubuntu  | ~         | `ls -l /dev/kvm`                                                                              | comfirm KVM exist                                                   | for VM                             |
| Ubuntu  | ~         | `ls -l /dev/net/tun`                                                                          | Verifies that the TUN/TAP driver is present                         | for VM                             |
| Ubuntu  | ~         | `grep -E --color ...`                                                                         | checking that CPU exposes Virtualization                            | for VM **see notes4                |
| Ubuntu  | ~         | `sudo apt update && sudo apt install -y qemu-system-x86 qemu-utils bridge-utils dnsmasq-base` | Installs the user-space emulator, helper utilities and bridge-utils | for VM                             |
| Ubuntu  | ~         | `sudo adduser $USER kvm`                                                                      | Add my account to KVM                                               | for VM  can start VMs without sudo |
| Ubuntu  | ~         | `ip link show`                                                                                | checking for LAN                                                    |                                    |
| Ubuntu  | ~         | `sudo ip link add name br0 type bridge`                                                       | makes a new logical “switch” inside the kernel                      |                                    |
| Ubuntu  | ~         | `sudo ip link set dev enx00e04c68018d master br0`                                             | Removes enx00e04c68018d from its old network and attaches it to br0 |                                    |
| Ubuntu  | ~         | `sudo ip link set dev enx00e04c68018d up`                                                     | Activate interface enx                                              |                                    |
| Ubuntu  | ~         | `sudo ip link set dev br0 up`                                                                 | Activate interface br0                                              |                                    |

**Notes:**
  1. The \ is used to escape the | character for the purpose of this document only - omit during use!
  2. Issue with character escape.  Whole command: `lscpu | grep -E '(Virtualization|vmx|svm)'`
  3. Used these two commands together to see how much resources I can spare to the Nix VM
  4. `grep -E --color "vmx|svm" /proc/cpuinfo | head`
  5. Remember that all ip commands are not persistant and need to be re run to set up br0

## 2025-07-15
| System | Directory                     | Command                                                                | Description                                             | Notes/Output                                      |
| ------ | ----------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------- |
| Ubuntu | ~                             | `sudo ip addr add 192.168.4.1/24 dev br0`                              | assigning static address to bridge interface            |                                                   |
| Ubuntu | ~                             | `sudo ip link set br0 up`                                              | bringing up previously assigned interface               |                                                   |
| Ubuntu | ~                             | `curl -L https://nixos.org/nix/install \| sh`                          | grabbing nix from address and piping to shell           | shell must be restarted                           |
| Ubuntu | ~                             | `. /home/kimdev/.nix-profile/etc/profile.d/nix.sh`                     | ensuring environment variables are set                  |                                                   |
| Ubuntu | ~                             | `nix --version`                                                        | checking nix is installed                               | nix (Nix) 2.30.1                                  |
| Ubuntu | ~                             | `mkdir -p ~/.config/nix`                                               | creating folder for nix.conf                            | becaue build failed                               |
| Ubuntu | ~                             | `nano ~/.config/nix/nix.conf`                                          | creating nix.conf                                       | Add: `experimental-features = nix-command flakes` |
| Ubuntu | ~                             | `exec $SHELL`                                                          | reload shell                                            |                                                   |
| Ubuntu | ~/Documents/yellow_ball_nixos | `nix build .#nixosConfigurations.dev-vm.config.system.build.qemuImage` | creating flake and VM                                   | FAILED                                            |
| Ubuntu | ~/Documents/yellow_ball_nixos | `nix build .#nixosConfigurations.dev-vm.config.system.build.vm`        | QEMU doesn't expose a qemuImage target used .vm instead | FAILED                                            |

**Notes:**
  - Because the ip commands are non-persistant these attributes will be set again later with nmcli (probably)

## 2025-07-16
| System | Directory                           | Command                                                                    | Description                                  | Notes/Output   |
| ------ | ----------------------------------- | -------------------------------------------------------------------------- | -------------------------------------------- | -------------- |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `mkdir -p nixos/secrets`                                                   | making folder for .pub file                  |                |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `ssh-keygen -t ed25519 -C "nix-vm-admin" -f ./nixos/secrets/root-ssh-keys` | creating key and saving to specific folder   | has passphrase |
| Ubuntu | ~/Documents/yellow_ball_nixos/nixos | `sudo nano dev-vm.nix`                                                     | create configuration file for vm             |                |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `git add .`                                                                | staging all files                            | **see note     |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `git add -f nixos/secrets/root-ssh-keys.pub`                               | force add file from .gitignore               | **see note     |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `git commit -m "<message>"`                                                | commiting change                             | **see note     |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `nix build .#nixosConfigurations.dev-vm.config.system.build.vm`            | build the vm                                 |                |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `git rm --cached nixos/secrets/root-ssh-keys.pub`                          | removes ssh key fit index but leaves on disk |                |
| Ubuntu | ~/Documents/yellow_ball_nixos       | `git commit -m "Stop tracking VM public key"`                              | re-commiting for future push                 |                |

**Notes:**
  - The files have to be Git indexed so that they will survive the flake-source copy.
  - The public key has to be commited locally which adds extra steps to key it from the remote repo

## 2025-07-17
| System | Directory | Command                                                                                                              | Description                             | Notes/Output                    |
| ------ | --------- | -------------------------------------------------------------------------------------------------------------------- | --------------------------------------- | ------------------------------- |
| Ubuntu | ~         | `nmcli connection delete <interface name>`                                                                           | deleting old previous network interface | Run for all related to project  |
| Ubuntu | ~         | `nmcli connection add con-name br0 type bridge ifname br0 stp no ipv4.method manual ipv4.addresses 192.168.4.1/24`   | creating br0 bridge /switch             |                                 |
| Ubuntu | ~         | `nmcli connection add con-name br0-enx00 type ethernet ifname enx00e04c68018d master br0`                            | creating enx00 and connecting to br0    |                                 |
| Ubuntu | ~         | `nmcli connection add type tun ifname tap0 con-name br0-tap0 mode tap owner "$(id -u)" slave-type bridge master br0` | Create tap and tell NM it's a bridge    | other ways didn't work this did |
| Ubuntu | ~         | `nmcli connection up br0`                                                                                            | bringing br0                            |                                 |
| Ubuntu | ~         | `nmcli connection up br0-enx00`                                                                                      | bringing enx00 up                       |                                 |
| Ubuntu | ~         | `nmcli connection up br0-tap0`                                                                                       | bringing tap0 up                        |                                 |
| Ubuntu | ~         | ``                                                                                                                   |                                         |                                 |

**Notes:**
  - I had to delete all previous network interfaces to re-setup my PC as a switch.  I tried to keep the steps as clean as possible but I've done them over many days.
  - Using `nmcli connection delete <interface name> is the should be run to start from scratch.

## 2025-07-18
| System | Directory                     | Command                                                                                                                            | Description               | Notes/Output                                         |
| ------ | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------- | ---------------------------------------------------- |
| Ubuntu | ~                             | `echo '. "$HOME/.nix-profile/etc/profile.d/nix.sh"' >> ~/.bashrc`                                                                  | Adding nix to path        | Did not realize the earlier change was not permanent |
| Ubuntu | ~                             | `source ~/.bashrc`                                                                                                                 | Immediately apply changes |                                                      |
| Ubuntu | ~/Documents/yellow_ball_nixos | `rm -f /tmp/nixos-*.raw`                                                                                                           | Delete old working disk   | **see notes                                          |
| SEE    | NOTES FOR                     | MISSING STEPS                                                                                                                      |                           |                                                      |
| Ubuntu | ~/Documents/yellow_ball_nixos | `./result/bin/run-dev-vm-vm -netdev tap,id=net0,ifname=tap0,script=no,downscript=no -device virtio-net-pci,netdev=net0 -nographic` |                           |                                                      |


**Notes:**
  - After the `rm -f /tmp/nixos-*.raw` on row 3 I again had to run the following commands:
    1. `git add -f nixos/secrets/root-ssh-keys.pub`
    2. `git commit -m "<message>"`
    3. `nix build .#nixosConfigurations.dev-vm.config.system.build.vm`
    4. `git rm --cached nixos/secrets/root-ssh-keys.pub`
    5. `git commit -m "Stop tracking VM public key"`
   - This is because I have to have the ssh key indexed for now - looking for solution


## 2025-07-22
| System | Directory                     | Command                                  | Description                 | Notes/Output                        |
| ------ | ----------------------------- | ---------------------------------------- | --------------------------- | ----------------------------------- |
| Ubuntu | ~/Documents/yellow_ball_nixos | `./result/bin/run-dev-vm-vm -nographic`  | Launch the VM               | without flags after config update   |
| Nix VM | ROOT                          | `mkdir -p /root/.ssh`                    | create ssh folder           |                                     |
| Nix VM | ROOT                          | `chmod 700 /root/.ssh`                   | change permissions          |                                     |
| Nix VM | ROOT                          | `nano /root/.ssh/authorized_keys`        | adding public key           | **see notes                         |
| Nix VM | ROOT                          | `chmod 600 /root/.ssh/authorized_keys`   | putting perms back          |                                     |
| Ubuntu | ~/Documents/yellow_ball_nixos | `sudo chown kimdev:kimdev ~/.ssh/config` | changing ownership for .ssh | Admin work to add ssh config for VM |
| Ubuntu | ~/Documents/yellow_ball_nixos | `chmod 600 ~/.ssh/config`                | changing perms              |                                     |
| Ubuntu | ~/Documents/yellow_ball_nixos | `nano ~/.ssh/config`                     |                             |                                     |
| Ubuntu | ~/Documents/yellow_ball_nixos | `ssh dev-vm`                             | VM has name now             | **see notes                                    |

**Notes:**
- I tried to pass the public ssh key inside of the dev-vm.nix config file but for some reason hadd issue this is the path of least resistance.
- `# Dev VM\ 
    Host dev-vm\
    HostName 192.168.4.3\
    User root\
    IdentityFile /home/kimdev/Documents/yellow_ball_nixos/nixos/secrets/root-ssh-keys`

## 2025-07-??
| System | Directory | Command | Description | Notes/Output |
| ------ | --------- | ------- | ----------- | ------------ |
|        |           | ``      |             |              |
|        |           | ``      |             |              |

**Notes:**

## 2025-07-??
| System | Directory | Command | Description | Notes/Output |
| ------ | --------- | ------- | ----------- | ------------ |
|        |           | ``      |             |              |
|        |           | ``      |             |              |

**Notes:**