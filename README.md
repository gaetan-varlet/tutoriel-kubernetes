# Kubernetes

## Introduction

- logiciel opensource créé par Gooogle
- orchestrateur : permet d'automatiser le déploiement et la gestion d'applications tournant dans des conteneurs
- concurrent : Docker Swarm et Docker Compose

Quelques notions importantes :
- **Container Linux** : c'est un processus isolé, avec une vision limitée du sytème (grâce au *namespace*), et un contrôle des ressources qu'il peut utiliser (grâce aux *control groups*)
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

- **Cluster** : ensemble de **nodes** (machines physiques ou virtuelles) Linux ou Windows
    - Node **master** en charge de la gestion du cluster ; expose l'**API Server**
    - Node **worker** en charge de faire tourner les applications
- **Pods** : plus petite unité applicative sur Kubernetes. C'est un ensemble de containers qui partagent une stack réseau et du stockage.
    - le nombre d'instance d'un pod est appelé **replicas**
    - gestion des pods identiques : utilisation d'une ressource **Deployment** pour spécifier la version de l'image, le nombre de replicas...
- **Service** : regroupement de pods similaires, qui expose les pods à l'intérieur ou à l'extérieur du cluster, en définissant de règles au niveau réseau
- à chaque ressource, possibilité d'attacher des informations supplémentaires avec les **Labels** et les **Annotations**, sous forme de clé et de valeur
- communcation avec Kubernetes via la commande **kubectl** qui envoie des requêtes HTTPS à l'API Server
