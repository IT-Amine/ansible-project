# Automatisation Infrastructure Réseau : Bascule CUB / ECOCERT

## Présentation du Projet
Ce projet vise à automatiser la reconfiguration complète d'une infrastructure réseau composée de commutateurs Cisco (Accès L2 et Cœur de réseau L3). 

L'objectif principal est de pouvoir basculer l'ensemble du réseau entre deux environnements de production distincts (**CUB** et **ECOCERT**) de manière fiable, rapide et sécurisée. L'automatisation garantit que l'ancienne configuration est proprement nettoyée avant l'application de la nouvelle, tout en maintenant l'accès d'administration continu aux équipements.

Pour plus de détails sur la stratégie de migration et les mécanismes techniques de bascule, vous pouvez consulter le document explicatif complet [en cliquant ici](./Contexte.md).

## Installation et Prérequis

Avant d'exécuter les playbooks, vous devez installer les dépendances nécessaires sur votre machine. Le projet s'appuie sur des bibliothèques Python spécifiques (`paramiko`, `ansible-pylibssh`) en plus d'Ansible, afin de garantir une communication SSH performante et stable avec les équipements Cisco IOS.

Pour installer ces prérequis, exécutez la commande suivante à la racine du projet :

```bash
pip install -r requirements.txt
```

## Arborescence Ansible

L'architecture du projet respecte les standards de l'Infrastructure as Code (IaC) et la philosophie **KISS** (un playbook simple par contexte, avec des rôles réutilisables) :

```text
ansible-projet/
├── ansible.cfg                # Configuration Ansible (inventaire par défaut, chemin des rôles)
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
├── roles/                     # Rôles réutilisables
│   ├── vlans/                 # Rôle pour la configuration des VLANs
│   ├── l2_interfaces/         # Rôle pour les ports physiques L2
│   └── l3_interfaces/         # Rôle pour les interfaces de routage SVI (L3)
└── playbooks/
    ├── deploy_cub.yml         # Déploiement spécifique au contexte CUB
    └── deploy_ecocert.yml     # Déploiement spécifique au contexte ECOCERT
```

## Comment utiliser l'automatisation ?

Le déploiement est grandement simplifié grâce à la configuration présente dans `ansible.cfg` (qui charge l'inventaire automatiquement) et à l'organisation par rôles. 

Chaque contexte possède son propre playbook qui orchestre les rôles de configuration (VLANs, L2, L3) sans nécessiter d'inputs de variables complexes dans la ligne de commande.

**Pour déployer l'environnement CUB :**
```bash
ansible-playbook playbooks/deploy_cub.yml
```

**Pour déployer l'environnement ECOCERT :**
```bash
ansible-playbook playbooks/deploy_ecocert.yml
```
