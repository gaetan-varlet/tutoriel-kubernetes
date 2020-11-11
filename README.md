# Kubernetes

## Introduction

- logiciel opensource créé par Gooogle
- orchestrateur : permet d'automatiser le déploiement et la gestion d'applications tournant dans des conteneurs
- concurrent : Docker Swarm et Docker Compose

Quelques notions importantes :
- **Container Linux** : c'est un processus isolé, avec une vision limitée du système (grâce au *namespace*), et un contrôle des ressources qu'il peut utiliser (grâce aux *control groups*)
- **Docker** a facilité l'utilisation des conteneurs
    - notion d'**image** : modèle qui package une application et ses dépendances, instancié dans un conteneur, qui permettent de lancer le ou les processus de l'application définie dans l'image
    - **Matrix from hell** : Docker permet de déployer n'importe quel application sur une multitude d'environnements
    - **Docker Hub** : bibliothèque d'image Docker prête à l'emploi, on peut également y déposer nos images
- **Architecture monolithique vs micro-services**
    - découpage de multiples services
    - possibilité de choisir des technologies différentes selon les services
    - chaque service a une fonctionnalité bien définie, facilite la maintenance
    - interconnexion des services nécessite des interfaces bien définies
    - complexité déplacée au niveau de l'orchestration de l'application globale
- **Application Cloud Native** : application orientée microservies, packagée dans des containers
- **Devops** : moyen de réduire le temps de livraison d'une fonctionnalité
    - rapprochement des développeurs et des opérateurs des plateformes de production
    - automatisation des processus via des pipelines


## La plateforme

### Le projet

- Kubernetes / k8s / kube
- signifie "Homme de barre" ou "pilote" en grec
- sortie en 2015
- orchestration de containers : gestion d'applications tournant dans des containers (déploiement, montée en charge, mise à jour...)
- gestion des services stateless et stateful
- gestion de la configuration et des secrets
- spécification des ressources en **yaml**

### Les concepts de base

- **Cluster** : (groupe en français) ensemble de **nodes** (machines physiques ou virtuelles) Linux ou Windows
    - Node **master** en charge de la gestion du cluster : expose l'**API Server**
    - Node **worker** en charge de faire tourner les applications
- **Pods** : plus petite unité applicative sur Kubernetes. C'est un ensemble de containers qui partagent une stack réseau et du stockage.
    - le nombre d'instance d'un pod est appelé **replicas**
    - gestion des pods identiques : utilisation d'une ressource **Deployment** pour spécifier la version de l'image, le nombre de replicas...
- **Service** : regroupement de pods similaires, qui expose les pods à l'intérieur ou à l'extérieur du cluster, en définissant de règles au niveau réseau
- à chaque ressource, possibilité d'attacher des informations supplémentaires avec les **Labels** et les **Annotations**, sous forme de clé et de valeur
- communcation avec Kubernetes via la commande **kubectl** qui envoie des requêtes HTTPS à l'API Server

### Architecture

- **Katacoda** : plateforme d'apprentissage des technologies "Cloud Native", notamment Kubenertes
- `kubectl version` permet de connaître la version
- `kubectl get nodes` permet de voir les noeuds du cluster

Processus tournant sur les nodes **Master**. Ils fournissent le plan de contrôle (control plane) du cluster :
- **API Server** : point d'entrée d'un cluster Kubernetes, en envoyant des requêtes à cette API
- **Scheduler** : sélectionne le node sur lequel un Pod sera lancé
- **Controller Manager** : ensemble de contrôleurs surveillant l'état des ressources et effectuant des actions correctives si des applications ne tournent pas correctement
- **etcd** : key-value store (BDD) contenant l'ensemble des données du cluster

Processus tournant sur les nodes **Master** et **Worker** :
- **kubelet** : assure que les containers d'un pod tournent conformément à la spécification, redémarre les containers en cas de crash
- **kube-proxy** : gère les règles réseau pour l'exposition des services
- **container-runtime** : environnement d'exécution des containers (Docker)

Un *namespace* permet de grouper et d'isoler des ressources
- `kubectl get pods` permet de voir les pods qui tournent dans le cluster dans le namespace par défaut
- `kubectl get pods -A` permet de lister les pods qui tournent dans tous les namespaces, notamment le namespace *kube-system* qui sont les composants qui permettent la gestion du cluster
- `kubectl get pod -n kube-system`  permet de lister les pods d'un namespace particulier

**API Server**
- API Rest
- permet de lister, créer, supprimer des ressources dans le cluster
- appel via des requêtes HTTP, ou via *kubectl*, via une interface web...
- chaque requête passe dans un pipeline : authentification, autorisation, admission contrôleurs

Lorsqu'on communique avec plusieurs clusters Kubernetes
- définition du **Context** pour définir à quel cluster on s'adresse et avec quel utilisateur
- par défaut, défini dans `${HOME}/.kube/config`
- possibilité de le définir dans la variable d'environnement `$KUBECONFIG` ou dans chaque commande avec l'option `--kubeconfig`
- l'utilitaire **kubectx** simplifie cela


## Cluster de développement

**Multipass**
- utilitaire permettant de créer des machines virtuelles Ubuntu facilement et peut utiliser différents hyperviseur de façon native (Hyper-V, HyperKit, KVM, VirtualBox)
- `multipass launch -n node1` crée une VM nommée *node1* avec 1Go de Ram, 1 cpu et 5Go de disque
    - `multipass launch -n node2 -c 2 -m 3G -d 10G` permet de personnaliser la conf
- `multipass list` liste des VM créés
- `multipass info node1` donne des infos sur la conf de la VM *node1*
- `multipass shell node1` permet de lancer un shell dans la VM *node1* avec l'utilisateur *ubuntu* qui est dans le groupe sudo
- `multipass exec node1 -- ls` lance une commande dans la VM *node1*, ici voir le contenu du répertoire courant
    - `multipass exec node1 -- /bin/bash -c "curl -sSL https://get.docker.com | sh"` installe Docker dans *node1*
- `multipass mount /home/gaetan/test node1:/home/ubuntu/test` permet de monter un répertoire local dans une VM
    - `multipass umount node1:/home/ubuntu/test` supprime le point de montage
- `start`, `stop`, et `delete` permettent de gérer le cycle de vie des VM
    - `multipart delete node1`
- `multipass purge` supprime définitivement les VM supprimées

- **Minikube** : logiciel ayant besoin d'un hyperviseur, qui peut lancer une machine virtuelle sur laquelle tournera l'ensemble des processus de Kubernetes (cluster ayant un seul node)
- **Kind** : (Kubernetes in Docker) logiciel permettant de lancer un Kubernetes en local. Chaque node du cluster va tourner dans un conteneur Docker
- **MicroK8s** : distribution légère à installer dans une VM
- **K3S** : distribution Kubernetes très light, également à installer dans une VM


## Cluster de production

- en production, il faut un cluster hautement disponible. En cas de problème, il faut qu'il continue à tourner correctement
- **Cluster Kubernetes managé** : administré par une entreprise tierce (gestion du cluster et des mises à jour), par exemple *Google Kubernetes Engine*, *Azure Container Service*, *Amazon Elastic Container Service*, *Digital Ocean*, *OVH*...
-  installation de Kubernetes soi-même sur des machines physiques ou virtuelles, avec des outils comme *kubeadm*, *kops*...
