# ansible_snaplock_malicious_act

When combined with NetApp's Lab on Demand service, this repository delivers an easy way to demonstrate ONTAP's ability to protect data by creating immutable backups using the SnapLock for Malicious Act Protection (SnapLock for SnapVault) feature.

## Set up instructions
1. Log in to NetApp's Lab on Demand portal and reserve the lab "Preview Lab for ONTAP 9.8 v1.1" (https://labondemand.netapp.com/lab/sl10628)
1. Launch the reserved lab
1. Configure onboard key manager on NetApp cluster1
    1. Run PuTTY and double-click on cluster1
    1. Log in as admin with Netapp1! as the password
    1. Type: security key-manager onboard enable
    1. Type a phrase (at least 32 characters)
    1. Re-type the phrase
    1. Type: exit
1. Repeat step 3 for cluster2
1. Install Ansible and prep the lab
    1. Run PuTTY with the following input:
        1. Host Name: 192.168.0.61
        1. Port: 22
    1. When provided with the PuTTY Security Alert pop-up chose Yes
    1. Log in as root with Netapp1! as the password
    1. Type: yum install -y ansible
    1. Type: curl https://bootstrap.pypa.io/get-pip.py - get-pip.py
    1. Type: python get-pip.py
    1. Type: pip install netapp-lib
    1. Type: ansible-galaxy collection install netapp.ontap
    1. Type: mkdir Ansible
    1. Type: cd Ansible
    1. Type: git clone https://github.com/git4regev/ansible_snaplock_malicious_act
    1. Type: cd ansible_snaplock_malicious_act
    1. Type: ansible-playbook prep_lab_playbook.yml
    1. Type: ansible-playbook Deploy_malicious_act_protection.yml

## Demo highlights
The lab is now ready for the demo. There is a pre-created volume called vol_data in cluster1 being protected by SnapLock for SnapVault on cluster2.
If you wish to create additional volumes, don't forget to update the immutable_backups_config.yml IaC (Infrastructure-as-Code) file by either adding the new volumes to the (under the prod_volumes variable), or alternatively, comment out the list of volumes (but do not comment out the variable itself) to dynamically protect all NAS volumes.

When ready to demonstrate the immutable snapshot feature:
1. Either manually (CLI, GUI, ...) update the SnapMirror relationship, or run the Deploy_malicious_act_protection.yml Ansible playbook (this will trigger a SnapMirror update for all relevant relationships)
1. Either from CLI, GUI, API, ... try to delete one of the newer snapshots (on cluster2) and show that the operation will fail due to the snapshot being locked by SnapLock
1. Either from CLI, GUI, API, ... try to delete one of the oldest snapshots (on cluster2) and that operation should succeed. Remember that unless you changed the default in the immutable_backups_config.yml IaC file, the protection retention is set to 3 minutes
1. From CLI, show the expiration time of the snapshots (on cluster2) by running the following command: snapshot show -vserver svm_protect -volume <volume name> -fields snaplock-expiry-time,owners 
