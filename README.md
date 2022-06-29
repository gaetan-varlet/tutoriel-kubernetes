# Kubernetes

## Introduction

- logiciel opensource créé par Google
- orchestrateur : permet d'automatiser le déploiement et la gestion d'applications tournant dans des conteneurs
- concurrent : Docker Swarm et Docker Compose

Quelques notions importantes :
- **Container Linux** : c'est un processus isolé, avec une vision limitée du système (grâce au *namespace*), et un contrôle des ressources qu'il peut utiliser (grâce aux *control groups*)
- **Docker** a facilité l'utilisation des conteneurs
    - notion d'**image** : modèle qui package une application et ses dépendances, instancié dans un conteneur, qui permettent de lancer le ou les processus de l'application définie dans l'image
    - **Matrix from hell** : Docker permet de déployer n'importe quel application sur une multitude d'environnements
    - **Docker Hub** : bibliothèque d'images Docker prêtes à l'emploi, on peut également y déposer nos images
- **Architecture monolithique vs micro-services**
    - découpage de l'application en multiples services
    - taille réduite : facilite le développement, les tests et la maintenance
    - possibilité de choisir des technologies différentes selon les services
    - chaque service a une fonctionnalité bien définie, facilite la maintenance
    - interconnexion des services nécessite des interfaces bien définies
    - complexité déplacée au niveau de l'orchestration de l'application globale
- **Application Cloud Native** : application orientée microservices, packagée dans des containers
- **Devops** : moyen de réduire le temps de livraison d'une fonctionnalité
    - rapprochement des développeurs et des opérateurs des plateformes de production
    - automatisation des processus via des pipelines


## La plateforme

### Le projet

- Kubernetes / k8s / kube
- signifie "Homme de barre" ou "Pilote" en grec
- sortie en 2015
- orchestration de containers : gestion d'applications tournant dans des containers (déploiement, montée en charge, mise à jour...)
- gestion des services stateless et stateful
- gestion de la configuration et des secrets
- lancement de batchs
- spécification des ressources dans des fichiers au format **yaml**

### Les concepts de base

- **Cluster** : (groupe en français) ensemble de **nodes** (machines physiques ou virtuelles) Linux ou Windows
  - Node **master** en charge de la gestion du cluster : expose l'**API Server**, le point d'entrée pour la gestion du cluster
  - Node **worker** en charge de faire tourner les applications
- **Pods** : plus petite unité applicative sur Kubernetes. C'est un ensemble de containers qui partagent une stack réseau et du stockage.
  - le nombre d'instances d'un pod est appelé **replicas**
- **Deployment** :ressoure qui permet la gestion des pods identiques
  - permet de spécifier la version de l'image, le nombre de replicas...
- **Service** : regroupement de pods similaires, qui expose les pods à l'intérieur ou à l'extérieur du cluster, en définissant des règles au niveau réseau

À chaque ressource, il est possible d'attacher des informations supplémentaires avec les **Labels** (pour la sélection d'objets) et les **Annotations** (utilisées par des outils et librairies clientes), sous forme de clé et de valeur

Communcation avec Kubernetes en envoyant des requêtes HTTPS à l'API Server :
- **kubectl** : outil en ligne de commande
  - récupération du fichier de configuration du serveur (URL du serveur, éléments d'authentification)
  - configuration du client avec ce fichier `${HOME}/.kube/config`, `${KUBECONFIG}`, `--kubeconfig` (à chaque commande *kubectl* pour ce dernier)
- utilisation d'autres interfaces (Web / Command-Line Interface)

### Architecture

**Katacoda** : plateforme d'apprentissage des technologies "Cloud Native", notamment Kubenertes
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
- **container-runtime** : environnement d'exécution des containers (Docker par défaut)

Un **namespace** permet de grouper et d'isoler des ressources
- `kubectl get pods` permet de voir les pods qui tournent dans le cluster dans le namespace par défaut
- `kubectl get pods -A` permet de lister les pods qui tournent dans tous les namespaces, notamment le namespace **kube-system** qui sont les composants qui permettent la gestion du cluster
- `kubectl get pod -n kube-system`  permet de lister les pods d'un namespace particulier

**API Server**
- API Rest
- permet de lister, créer, supprimer des ressources dans le cluster
- appel via des requêtes HTTP, ou via **kubectl**, via une interface web...
- chaque requête passe dans un pipeline : authentification, autorisation, admission contrôleurs

Configuration pour travailler avec plusieurs clusters Kubernetes
- définition du **Context** pour définir à quel cluster on s'adresse et avec quel utilisateur
- l'utilitaire **kubectx** simplifie cela

## Cluster de développement

**Multipass**
- utilitaire permettant de créer des machines virtuelles Ubuntu facilement et peut utiliser différents hyperviseur de façon native (Hyper-V, HyperKit, KVM, VirtualBox)
- `multipass launch -n node1` crée une VM nommée *node1* avec 1Go de Ram, 1 CPU et 5Go de disque
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
    - création d'une VM Ubuntu : `multipass launch --name microk8s --mem 4G`
    - installation de microk8s dans la VM : `multipass exec microk8s -- sudo snap install microk8s --classic`
    - récupération sur notre machine locale du fichier de configuration généré par microk8s : `multipass exec microk8s -- sudo microk8s.config > microk8s.yaml`
    - positionnement de la variable d'environnement KUBECONFIG sur le fichier de configuration : `export KUBECONFIG=$PWD/microk8s.yaml`
    - test en regardant les nodes du cluster : `kubectl get nodes`
- **K3S** : distribution Kubernetes très light, également à installer dans une VM


## Cluster de production

- en production, il faut un cluster hautement disponible. En cas de problème, il faut qu'il continue à tourner correctement
- **Cluster Kubernetes managé** : administré par une entreprise tierce (gestion du cluster et des mises à jour), par exemple *Google Kubernetes Engine*, *Azure Container Service*, *Amazon Elastic Container Service*, *Digital Ocean*, *OVH*...
-  installation de Kubernetes soi-même sur des machines physiques ou virtuelles, avec des outils comme *kubeadm*, *kops*...


## L'objet Pod

- plus petite unité applicative dans Kubernetes
- groupe de containers tournant dans un même contexte d'isolation
- partagent la stack réseau (les containers peuvent communiquer sur localhost) et le stockage (via des volumes)
- chaque Pod a une adresse IP dédiée dans le cluster
- peut contenir des containers (souvent un seul container applicatif) et des volumes
- Scaling horizontal via le nombre de replica d'un Pod

### Spécification

Exemple de spécification d'un Pod dans lequel est lancé un container basé sur l'image nginx (serveur web)

```yml
$ cat nginx-pod.yaml
apiVersion: v1 # définition de la version de l'API utilisé par la version des Pod
kind: Pod # type d'objet
metadata: # ajout d'un nom, d'autres metadata peuvent être ajoutées (labels,...)
  name: www 
spec: # spécifications du Pod (containers, volumes utilisées dans le Pod)
  containers: # définition d'un seul container qu'on appelle www basé sur l'image nginx en version 1.12.2
  - name: nginx
    image: nginx:1.12.2
```

### Gestion du cycle de vie d'un Pod

- lancement d'un Pod : `kubectl create -f POD_SPECIFICATION.yaml`
- liste des Pods : `kubectl get pod` (par défaut sur le namespace default)
- description d'un Pod : `kubectl describe pod POD_NAME` donne les détails du Pod
- logs d'un container d'un Pod : `kubectl logs POD_NAME [-c CONTAINER_NAME]`
- lancement d'une commande dans un container d'un Pod existant `kubectl exec POD_NAME [-c CONTAINER_NAME] -- COMMAND`
- suppression d'un Pod `kubectl delete pod POD_NAME`
- publication du port d'un Pod sur la machine hôte : `kubectl port-forward POD_NAME HOST_PORT:CONTAINER_PORT`, utilisé pour le développement et le debugging
- container pause
    - `docker ps | grep www` pour afficher les containers sur la machine en filtrant sur le nom www
    - il y a 2 containers : un container nginx et un container basé sur l'image *pause*
    - garant des namespaces qui est utilisé pour isoler les processus du pod
    - les containers du pod utilisent ces mêmes namespaces pour communiquer entre eux sur l'interface localhost


Exemple

```bash
# Lancement du Pod
kubectl create -f nginx-pod.yaml

# Liste des Pods présents pour vérifier que notre Pod est présent
kubectl get pods

# Lancement d'une commande dans un Pod (pas besoin de préciser le container comme c'est le seul dans le pod)
kubectl exec www -- nginx -v
kubectl exec www -c nginx -- nginx -v # équivalent en précisant le container dans le pod
# nginx version: nginx/1.12.3

# Shell interactif dans un Pod (utilisé pour faire du débug)
kubectl exec -t -i www -- /bin/bash
# root@nginx:/#

# Détails du pod
kubectl describe pod www

# le port 80 du container nginx qui tourne dans le pod www est exposé sur le port 8080 de la machine hôte
# Accès à la page d'accueil de nginx via localhost:8080
kubectl port-forward www 8080:80

# Suppression du Pod
kubectl delete pod www
```

### Pod avec plusieurs containers

- exemple avec Wordpress (pas utilisable en production car non scalable)
- définition de 2 containers dans le même pod
    - moteur Wordpress
    - base de données MySQL
- Définition d'un volume pour la persistence des données de la base
    - type **emptyDir** : associé au cyclé de vie du Pod
    - répertoire vide créé sur la machine hôte lors de la création du Pod, et utilisé par le Pod pour y stocker les fichiers de MySQL
    - créé dans le Pod et utilisé dans un ou des containers du pod. Les données du container vont donc être stockées dans le volume et non dans le container, comme ça si le container meurt, les données ne sont pas perdus car le volume est lié au cycle de vie du Pod et non du container

Exemple du fichier de spécifications du Pod : 2 containers (Wordpress et MySQL)

```yml
apiVersion: v1
kind: Pod
metadata:
  name: wp
spec:
  containers:
  - image: wordpress:4.9-apache
    name: wordpress
    env: # ajout de 2 variables d'environnement dans le container WORDPRESS
    - name: WORDPRESS_DB_PASSWORD
      value: mysqlpwd
    - name: WORDPRESS_DB_HOST
      value: 127.0.0.1
  - image: mysql:5.7
    name: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: mysqlpwd
    volumeMounts: # définition d'un volume dans le pod en plus des 2 containers
    - name: data
      mountPath: /var/lib/mysql # répertoire de la machine hôte que l'on monte dans le répertoire indiqué du container MySQL
  volumes:
  - name: data
    emptyDir: {}
```

- création du pod avec les 2 containers : `kubectl create -f wordpress-pod.yaml`
- exposition du port 80 du container wordpress : `kubectl port-forward wp 8080:80`, et accès via localhost:8080 (les 2 containers communiquent donc entre-eux en local)


## L'objet Service

## L'objet Deployment

## L'objet Namespace

## Exemple : Application micro-services

##  Kubectl

## L'objet ConfigMap

## Exemple : Stack Elastic (ELK)

## L'objet Secret

## Utilsateurs et droits d'accès - RBAC

## Interface web de gestion

## DaemonSet

## Job - CronJob

## L'objet Ingress

## Kubernetes Operators

## Workload stateful

## Helm

## Exemple : Intégration et Déploiement continue

## ServiceMesh
