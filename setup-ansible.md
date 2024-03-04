# Ansible to assist with server maintenance

- Requires a controller server but you can make one of your managed servers the controller if you want.
- Ansible only needs to be installed on the controller server.
- Best practice is to create a specific user as a service account and use it exclusively, this guide uses the 'ansible' user.
- This was all tested in Ubuntu 22.04.
- Other distros: https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-distros
- Note comments below are behind the # symbol! When you see `sudo dosomething ## this command does something` you only have to run the command `sudo dosomething` and then READ the comment to see more information.

## Install Ansible:
This is on the controller server.
```
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

## Create user 'ansible' and an ssh key-pair to use with it:
This is on the controller server. You will use this key-pair with EVERY managed server you control.
```
sudo adduser -c "Ansible Service Account" ansible ## When prompted for password, create any strong password, you won't be using it. Enter on all the questions, no need to fill out.
sudo visudo ## Look for '# User privilege specification'. Under root, add the following:  ansible   ALL=(ALL:ALL) NOPASSWD:ALL. It should look like this when done:
    # User privilege specification
    root            ALL=(ALL:ALL) ALL
    ansible         ALL=(ALL:ALL) NOPASSWD:ALL

    Note: to save, type :wq and the Enter key.

sudo su ansible ## This switched to the ansible user you just created
cd ~
mkdir -p ~/.ssh && chmod 700 ~/.ssh && cd ~/.ssh
ssh-keygen -t ed25519 ## DO NOT ADD A PASSPHRASE! Save it to /home/ansible/.ssh/ (should be default path shown, just hit Enter).
```

Note! If you create lots of key-pairs over time, all named `id_ed25519.pub` and `id_ed25519` you are eventually going to have a bad time figuring out which is which. Highly recommended you add the date and name of the account like so (this guide assumes you did this):
```
mv id_ed25519.pub id_ed25519-20240228-ansible.pub ## This is your public key.
mv id_ed25519 id_ed25519-20240228-ansible.key ## This is your private key, keep safe.
```

Note: If you already have an authorized_keys file, you probably know not to do this next part. Simply add your new public key to your authorized_keys file.
```
cp id_ed25519-20240228-ansible.pub authorized_keys ## copying the public key into a format ssh wants to see.
chmod 0600 authorized_keys
```

## Create user 'ansible' on managed servers:

This is on EVERY managed server you want to control. If you have 20 Evernode servers, you do this 20 times.
```
sudo adduser -c "Ansible Service Account" ansible ## When prompted for password, create any strong password, you won't be using it. Enter on all the questions, no need to fill out.
sudo visudo ## Look for '# User privilege specification'. Under root, add the following:  ansible   ALL=(ALL:ALL) NOPASSWD:ALL. It should look like this when done:
    # User privilege specification
    root            ALL=(ALL:ALL) ALL
    ansible         ALL=(ALL:ALL) NOPASSWD:ALL

    Note: to save, type :wq and the Enter key.

sudo su ansible ## This switches to the ansible user you just created
cd ~
mkdir -p ~/.ssh && chmod 700 ~/.ssh && cd ~/.ssh
```

STOP! DO NOT create another key-pair, you use the one you already created on the controller server. Copy the authorized_key file to the .ssh folder and you MUST fix the permissions `chmod 0600 authorized_keys`. You can simply create a new file and paste the contents or physically copy the file off the controller and on to your managed server, your choice. DO NOT move the key, you don't need it on 

Be sure to also update your /etc/ssh/sshd_config file if using an AllowUsers directive and restart the service `sudo systemctl restart sshd.service`

When all is done, you should have a user 'ansible' with an 'authorized_keys' on every server you want to manage.

## Add your managed hosts to /etc/ansible/hosts file
This is back on the controller server. You stay on the controller now, you are done touching the managed servers.

Note: This is partially where we get into preference. I do not do any anything as user 'root' unless I absolutely have to, I use a regular user. I copied my ansible private key to my regular user's .ssh folder because I want to run ansible playbooks from my regular user and not switch to the user 'ansible' all the time. Below, my regular user is 'jojo'. What you do is up to you, this is how I did it:
```
cd ~/.ssh
mkdir ansible && cd ansible
sudo cp /home/ansible/.ssh/id_ed25519-20240228-ansible.key .
chown jojo:jojo id_ed25519-20240228-ansible.key # change ownership from root to my regular user.
chmod 0600 id_ed25519-20240228-ansible.key # make sure permissions are set correctly.
```

## Now it's time to add everything needed to the /etc/ansible/hosts file
This is an EXAMPLE! Your hosts file will be different IP addresses, different key path, etc. I am showing how to add two groups, PLEASE edit as needed. Are you using a custom port for ssh like me? That's why that setting is included.
```
[evernode]
evr-node01 ansible_host=22.22.22.22     ansible_user=ansible ansible_port=22    ansible_ssh_private_key_file=/home/jojo/.ssh/ansible/id_ed25519-20240228-ansible.key
evr-node02 ansible_host=55.55.55.55     ansible_user=ansible ansible_port=22    ansible_ssh_private_key_file=/home/jojo/.ssh/ansible/id_ed25519-20240228-ansible.key
evr-node03 ansible_host=88.88.88.88     ansible_user=ansible ansible_port=22    ansible_ssh_private_key_file=/home/jojo/.ssh/ansible/id_ed25519-20240228-ansible.key

[xahaud]
xah-node01 ansible_host=123.45.67.89    ansible_user=ansible ansible_port=22    ansible_ssh_private_key_file=/home/jojo/.ssh/ansible/id_ed25519-20240228-ansible.key
```

## Configure ansible.cfg
Open /etc/ansible/ansible.cfg and add the following:
```
[ssh_connection]
ssh_args = -o StrictHostKeyChecking=accept-new

[defaults]
# Use the YAML callback plugin.
stdout_callback = yaml
# Use the stdout_callback when running ad-hoc commands.
bin_ansible_callbacks = True
```

Ansible should now be configured.
