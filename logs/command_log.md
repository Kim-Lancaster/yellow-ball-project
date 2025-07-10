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
  - 1. wget https://hydra.nixos.org/build/298477550/download/1/nixos-image-sd-card-25.05.802746.7282cb574e06-aarch64-linux.img.zst
  - 2. nixos-image-sd-card-25.05.802746.7282cb574e06-aarch64-linux.img

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
- 1. I had to clear out a stale host key from my last try.
- 2. Had errors because of the configuration.nix file and lack of internet connection.
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
-  1. It is assumed that if I am working on the pi that I have already run the command to ssh in.
-  2. I already had a network interface set up from previous tries but for a new one:
  - ``sudo nmcli connection add \
  type ethernet \
  ifname enx00e04c68018d \
  con-name <name> \
  autoconnect yes \
  ipv4.method manual \
  ipv4.addresses 192.168.4.1/24 \
  ipv4.gateway "" \
  ipv4.dns "1.1.1.1,8.8.8.8" \
  ipv4.ignore-auto-dns yes``
- 3. I have broken the whole command into seperate lines for readability - should be entered in one chunck like the above example with nmcli add.
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
| Ubuntu | ~/Documents/yellow_ball_project | `cat ~/.ssh/id_ed25519_github.pub`                                                               | print key to terminal         | add to github                              |
| Ubuntu | ~/Documents/yellow_ball_project | `sudo nano ~/.ssh/config`                                                                        | creating a config file        | github need it so it know which key to use |
| Ubuntu | ~/Documents/yellow_ball_project | `sudo chmod 600 ~/.ssh/config`                                                                   | changing permissions          | GPT recommended but not sure if necessary  |
| Ubuntu | ~/Documents/yellow_ball_project | `eval "$(ssh-agent -s)"`                                                                         | start agent                   | **see notes                                |
| Ubuntu | ~/Documents/yellow_ball_project | `ssh-add ~/.ssh/id_ed25519_github`                                                               | add private key               | **see notes                                |
| Ubuntu | ~/Documents/yellow_ball_project | `ssh -T git@github.com`                                                                          | connecting over ssh           | verifying key match                        |
| Ubuntu | ~/Documents/yellow_ball_project | `git push -u origin main`                                                                        | push to github                |                                            |
|        |                                 | ``                                                                                               |                               |                                            |
**Notes:**
- "Initial project structure and import of my log files and NixOS config files."
- Had to add public key to Github
- I have to run `eval "$(ssh-agent -s)"` and `ssh-add ~/.ssh/id_ed25519_github` everytime I open a new shell or it I can't push to github.
## YYYY-MM-DD
| System | Directory | Command | Description | Notes/Output |
| ------ | --------- | ------- | ----------- | ------------ |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
|        |           | ``      |             |              |
**Notes:**