# ansible-playbooks

```
git clone https://github.com/go140point6/ansible-playbooks.git
cd ansible-playbooks
```

STOP! If you haven't set up ansible according to [setup-ansible](setup-ansible.md), do it now. These plays are worthless if you don't setup and configure ansible specific to your environment.

```
ansible-playbooks/
|
|--common/ (common server plays)
|--evernode/ (evernode specific plays)
|--plugin/ (plugin specific plays)
|--xahaud/ (xahaud specific plays)
|--xinfin/ (xdc network specific plays)
README.md
setup-ansible.md
```

All plays are self-documenting by using the `--tags help` command-line option:

```
ansible-playbook evernode/config_extrafee.yml --tags help
```

The two ways to run this particular play:
```
ansible-playbook evernode/config-extrafee.yml -e "nodes=evernode" ## show the current extrafee for the group "evernode" as defined in /etc/ansible/hosts.
ansible-playbook evernode/config-extrafee.yml -e "nodes=evr-node02,evr-node05 extrafee=3600" ## configure the extrafee on only those two nodes to 3600 drops.
```
