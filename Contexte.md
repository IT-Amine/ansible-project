# Contexte Technique et Stratégie d'Automatisation

## 1. Contexte Initial et Problématique
L'infrastructure physique repose sur un commutateur d'accès (Catalyst 2960 - L2) et un commutateur de cœur de réseau (Catalyst 9200L - L3). 
Le besoin opérationnel exige de basculer l'intégralité du réseau (VLANs, affectation des ports d'accès, Trunks, et interfaces de routage SVI) d'un contexte "CUB" vers un contexte "ECOCERT", et inversement.

La difficulté majeure de cette automatisation réside dans le **nettoyage de l'ancienne configuration**. L'utilisation de modules déclaratifs stricts (mode `overridden`) permet d'effacer automatiquement les VLANs obsolètes. Cependant, si le port et le VLAN d'administration utilisés par Ansible ne sont pas explicitement protégés lors de ce processus, l'outil "scie la branche sur laquelle il est assis", entraînant une coupure immédiate de la session SSH et l'échec du script.

## 2. La Stratégie de Résolution (Data-Driven Approach)
Pour répondre à cette problématique, le projet adopte une architecture modulaire basée sur la fusion de variables au moment de l'exécution.

### A. Le "Socle Intouchable" (Management)
Le VLAN de management (VLAN 98) et les ports de connexion SSH dédiés (`Fa0/1` pour le L2 et `Gi1/0/1` pour le L3) sont définis en dur dans le dossier `group_vars/`. Ces variables représentent l'accès vital aux équipements et sont systématiquement chargées par Ansible, quel que soit le projet en cours de déploiement.

### B. L'Isolation par Projet
Les architectures spécifiques à chaque contexte sont isolées dans le dossier `vars/` (`project_cub.yml` et `project_ecocert.yml`). 
Conformément aux bonnes pratiques réseau (optimisation Spanning-Tree et sécurité), **chaque équipement ne reçoit que les paramètres dont il a strictement besoin**. Par exemple, un switch L2 ne recevra pas la définition d'un VLAN serveurs s'il n'héberge aucun serveur. Les interfaces de routage (SVI) ne sont déclarées que pour le commutateur L3.

### C. La Fusion et l'Idempotence
Le playbook `deploy_project.yml` effectue une addition mathématique entre le "Socle Intouchable" et les "Variables Projet". 
Lorsqu'Ansible applique la configuration avec le paramètre `state: overridden` :
1. Il maintient ou crée l'accès d'administration (VLAN 98).
2. Il déploie la nouvelle topologie (VLANs, Ports, SVIs).
3. Il calcule le différentiel et supprime automatiquement toute configuration appartenant à l'ancien projet.

## 3. Spécificités d'Architecture Réseau
- **Management Hors-Bande Logique :** Le lien de management (VLAN 98) est physiquement séparé du flux de production sur les ports numéro 1.
- **Trunk Filtré :** Le lien Trunk entre les commutateurs L2 et L3 ne transporte strictement que les VLANs du projet en cours. Le VLAN 98 en est explicitement exclu pour forcer le passage par les ports de management dédiés.
- **Sécurité des identifiants :** Les informations sensibles (mots de passe enable et SSH) ne sont pas stockées en clair dans les playbooks. Il est recommandé de chiffrer ces variables sensibles via **Ansible Vault**.
