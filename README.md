# Automatisation Infrastructure Réseau : Bascule CUB / ECOCERT

## Présentation du Projet
Ce projet vise à automatiser la reconfiguration complète d'une infrastructure réseau composée de commutateurs Cisco (Accès L2 et Cœur de réseau L3). 

L'objectif principal est de pouvoir basculer l'ensemble du réseau entre deux environnements de production distincts (**CUB** et **ECOCERT**) de manière fiable, rapide et sécurisée. L'automatisation garantit que l'ancienne configuration est proprement nettoyée avant l'application de la nouvelle, tout en maintenant l'accès d'administration continu aux équipements.

## Arborescence Ansible

L'architecture du projet respecte les standards de l'Infrastructure as Code (IaC) :

```text
ansible-projet/
├── README.md                  # Ce fichier (présentation et utilisation)
├── Contexte.md                # Explications techniques détaillées de la stratégie
├── inventory/
│   └── hosts.yml              # Inventaire des switchs L2 et L3
├── group_vars/
│   ├── all.yml                # Variables globales (VLAN de management)
│   ├── l2_switches.yml        # Protection du port d'accès L2
│   └── l3_switches.yml        # Protection du port d'accès L3
├── vars/
│   ├── project_cub.yml        # Architecture complète du projet CUB
│   └── project_ecocert.yml    # Architecture complète du projet ECOCERT
└── playbooks/
    └── deploy_project.yml     # Le moteur d'automatisation
```

## Comment utiliser l'automatisation ?

Le déploiement se fait en appelant le playbook principal et en lui passant le nom du projet cible via une variable externe (`target_project`).

**Pour déployer l'environnement CUB :**
```bash
ansible-playbook -i inventory/hosts.yml playbooks/deploy_project.yml -e target_project=cub
```

**Pour déployer l'environnement ECOCERT :**
```bash
ansible-playbook -i inventory/hosts.yml playbooks/deploy_project.yml -e target_project=ecocert
```
