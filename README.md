# Simple Raspberry Pi Cluster Setup

I needed to create a 5xPi cluster to run some distributed Erlang.

The stuff here gets me to that point.

### Create the Bootable SD Cards

We're going to make sure we can ssh in, and that it will connect to your local network.

1. Download a Raspian Stretch image into `images/` and unzip it.

2. Mount it, and

   a. touch an empty file called `ssh` in the `/boot` filesystem
   b. update the ssid and password in `wpa_supplicant.wpa` and copy
      it into the same filesystem.

3. Unmount it, and burn the image onto an SD card for each Pi.

       dd bs=1m conv=sync if=2018-03-13-raspbian-stretch-lite.img of=/dev/rdisk5

### Bootstrap the Ansible Environment

Load the SD cards into the Pis (I numbered mine with a sharpie). Power them up
one at a time. Using your router's management screen or nmap, find its WiFi ipaddress.

If you want to confirm it, try

    $ ssh pi@192.168.0.xxx
    Password:  raspberry

Now go into the `ansible` directory.

For each pi, run the following:

    ansible-playbook -i "192.168.0.xx," -u pi -k playbooks/00-initial-setup.yml -e "ip=11 host=n1"

The IP address following the `-i` option is the WiFi address of the Pi being set up. The `ip=` parameter
is the static IP address that will be assigned to the hardwired eth0 interface (the prefix will be
192.168.3.). The `host=` parameter sets the hostname for that card.

Double check the interface was set by ssh-ing in to `pi@192.168.3.11` (or whatever IP you used). I'm
using a MacBook, so I had to manually set the IP address on my Thunderbolt interface and enable internet
sharing.

Repeat for each Pi in your cluster.

### Do the Final Setup

Now we'll use Ansible to load up the rest of the environments.

1. Create an inventory file. I call mine `hosts`

        n2
        n3
        n4
        n5

        [builder]
        n1

  The `n1, n2`... are the names of my nodes (set using the `host=` parameter to
  `ansible_playbook` above. I nominate one machine to be the build server for
  deployments.

2. Tell ansible the IP addresses of each node. Create a file
   `host_vars/«nodename».yml` for each one. For example, here's my `n1.yml`

       ---
       ansible_host: "192.168.3.11"

3. Run the playbooks:

       ansible-playbook -i hosts  -v -t d  playbooks/01-install-collectd.yml
       ansible-playbook -i hosts  -v -t d  playbooks/02-erlang.yml

### Do something cool

. . .