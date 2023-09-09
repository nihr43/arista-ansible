# arista-ansible

This project presents a proof-of-concept for deploying declarative `startup-config` templates on Arista EOS datacenter switches.

This differs from the Ansible Network [Arista.Eos Collection](https://docs.ansible.com/ansible/latest/collections/arista/eos/index.html) in that we are not using imperative tasks to modify running config, merge ('replace'? 'override'?) configs, run cli commands, etc - we are simply landing the entire startup-config as a template and reloading it; no guesswork.  In the long term, this will prove be more sustainable and scalable.

---

We're able to do this on EOS because Arista presents ‘Full Access to Linux shell and tools’ as a first-class feature, meaning all we have to do is land an ssh key, and we can treat it like any other linux box.  (the linux system itself is [a bit weird](https://signal.nih.earth/posts/arista_linux/) however)

To get started with this project from a base config, log in as admin, get an ip, set a default route and dns:

```
en
conf t
ip name-server vrf default 1.1.1.1
interface Ethernet48
   no switchport
   ip address 172.30.184.98/27
   end
ip route 0.0.0.0/0 172.30.184.97
```

Now we can enter bash, and create the magic `/mnt/flash/rc.eos` script to land an ssh key on the ephemeral root filesystem at boot:

```
en
bash
-bash-4.1# cat /mnt/flash/rc.eos 
#!/bin/sh

mkdir -p /root/.ssh
echo 'ssh-rsa keyyyy' > /root/.ssh/authorized_keys
```

You should now be able to ssh root@ .

With ssh set-up, we can run `ansible-playbook main.yml --diff`:

```
~/git/arista-ansible$ ansible-playbook main.yml --diff

PLAY [switches] ************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************
[WARNING]: Platform linux on host 172.30.184.98 is using the discovered Python interpreter at /usr/bin/python2.7, but
future installation of another Python interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-core/2.15/reference_appendices/interpreter_discovery.html for more information.
ok: [172.30.184.98]

TASK [eos : declare interfaces list] ***************************************************************************************
ok: [172.30.184.98]

TASK [eos : detect switchports] ********************************************************************************************
skipping: [172.30.184.98] => (item=ma1) 
skipping: [172.30.184.98] => (item=txraw) 
ok: [172.30.184.98] => (item=et34)
ok: [172.30.184.98] => (item=et35)
ok: [172.30.184.98] => (item=et43)
ok: [172.30.184.98] => (item=et49)
skipping: [172.30.184.98] => (item=et48) 
ok: [172.30.184.98] => (item=et29)
ok: [172.30.184.98] => (item=et28)
ok: [172.30.184.98] => (item=et42)
ok: [172.30.184.98] => (item=et25)
ok: [172.30.184.98] => (item=et24)
ok: [172.30.184.98] => (item=et41)
ok: [172.30.184.98] => (item=et40)
ok: [172.30.184.98] => (item=et47)
ok: [172.30.184.98] => (item=et20)
ok: [172.30.184.98] => (item=et45)
ok: [172.30.184.98] => (item=et22)
ok: [172.30.184.98] => (item=et2)
ok: [172.30.184.98] => (item=et3)
ok: [172.30.184.98] => (item=et1)
ok: [172.30.184.98] => (item=et6)
ok: [172.30.184.98] => (item=et7)
ok: [172.30.184.98] => (item=et4)
ok: [172.30.184.98] => (item=et5)
ok: [172.30.184.98] => (item=et21)
ok: [172.30.184.98] => (item=et8)
ok: [172.30.184.98] => (item=et9)
ok: [172.30.184.98] => (item=et46)
ok: [172.30.184.98] => (item=et23)
ok: [172.30.184.98] => (item=et27)
ok: [172.30.184.98] => (item=et44)
skipping: [172.30.184.98] => (item=vxlan) 
skipping: [172.30.184.98] => (item=mirror4) 
skipping: [172.30.184.98] => (item=mirror5) 
skipping: [172.30.184.98] => (item=mirror6) 
skipping: [172.30.184.98] => (item=mirror7) 
skipping: [172.30.184.98] => (item=mirror0) 
skipping: [172.30.184.98] => (item=mirror1) 
skipping: [172.30.184.98] => (item=mirror2) 
skipping: [172.30.184.98] => (item=mirror3) 
ok: [172.30.184.98] => (item=et52)
skipping: [172.30.184.98] => (item=mirror8) 
skipping: [172.30.184.98] => (item=mirror9) 
ok: [172.30.184.98] => (item=et32)
ok: [172.30.184.98] => (item=et33)
ok: [172.30.184.98] => (item=et30)
ok: [172.30.184.98] => (item=et17)
ok: [172.30.184.98] => (item=et26)
ok: [172.30.184.98] => (item=et37)
ok: [172.30.184.98] => (item=et12)
ok: [172.30.184.98] => (item=et13)
ok: [172.30.184.98] => (item=et50)
ok: [172.30.184.98] => (item=et51)
ok: [172.30.184.98] => (item=et38)
ok: [172.30.184.98] => (item=et39)
ok: [172.30.184.98] => (item=et18)
ok: [172.30.184.98] => (item=et19)
skipping: [172.30.184.98] => (item=fabric) 
ok: [172.30.184.98] => (item=et14)
skipping: [172.30.184.98] => (item=mirror13) 
ok: [172.30.184.98] => (item=et15)
ok: [172.30.184.98] => (item=et16)
ok: [172.30.184.98] => (item=et10)
ok: [172.30.184.98] => (item=et36)
ok: [172.30.184.98] => (item=et31)
skipping: [172.30.184.98] => (item=lo) 
skipping: [172.30.184.98] => (item=mirror11) 
skipping: [172.30.184.98] => (item=tunnelTap0) 
skipping: [172.30.184.98] => (item=mirror14) 
skipping: [172.30.184.98] => (item=mirror15) 
skipping: [172.30.184.98] => (item=mirror12) 
skipping: [172.30.184.98] => (item=cpu) 
skipping: [172.30.184.98] => (item=mirror10) 
ok: [172.30.184.98] => (item=et11)

TASK [eos : land startup-config] *******************************************************************************************
--- before: /mnt/flash/startup-config
+++ after: /home/nhensel/.ansible/tmp/ansible-local-558044s51t4wwe/tmpy1wxwzve/startup-config
@@ -5,7 +5,7 @@
 !
 transceiver qsfp default-mode 4x10G
 !
-ip name-server vrf default 1.1.1.1
+ip name-server vrf default 8.8.8.8
 !
 spanning-tree mode mstp
 !

changed: [172.30.184.98]

TASK [eos : land init script] **********************************************************************************************
ok: [172.30.184.98]

RUNNING HANDLER [eos : copy startup-config running-config] *****************************************************************
changed: [172.30.184.98]

PLAY RECAP *****************************************************************************************************************
172.30.184.98              : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

You probably don't want my inventory or template.  Fork and go nuts.

---

See also [Sustainable, Declarative Datacenter Switch Configuration with Arista EOS](https://signal.nih.earth/posts/arista_config/)
