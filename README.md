# Configuration d'un Cluster Kubernetes Multi-Nœuds avec Kubeadm
Ce fichier README fournit des instructions détaillées pour configurer un cluster Kubernetes multi-nœuds en utilisant Kubeadm.

## Vue d'ensemble
Ce guide fournit des instructions détaillées pour configurer un cluster Kubernetes multi-nœuds à l'aide de Kubeadm. Le guide inclut des instructions pour installer et configurer containerd et Kubernetes, désactiver le swap, initialiser le cluster, installer Flannel et ajouter des nœuds au cluster.

## Prérequis
Avant de commencer le processus d'installation, assurez-vous que les prérequis suivants sont remplis :

- Vous avez au moins des machines Linux buntu 18.04 ou plus récents disponibles pour créer le cluster.
- Chaque machine dispose d'au moins 2 Go de RAM et 2 cœurs CPU.
- Les machines ont une connectivité réseau entre eux.
- Vous avez un accès root à chaque machine.

## Etapes d'installation
Voici les étapes d'installation détaillées pour configurer un cluster Kubernetes multi-nœuds avec Kubeadm :

Mettez à jour la liste des paquets du système et installez les dépendances nécessaires en utilisant les commandes suivantes :

```
sudo apt-get update
sudo apt install apt-transport-https curl -y
```

## Installer containerd
Pour installer Containerd, utilisez les commandes suivantes:

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
```

## Créer la configuration de containerd
Ensuite, créez le fichier de configuration de containerd en utilisant les commandes suivantes :

```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

## Modifier le fichier /etc/containerd/config.toml
Modifiez le fichier de configuration de containerd pour définir SystemdCgroup à true. Utilisez la commande suivante pour ouvrir le fichier :

```
sudo nano /etc/containerd/config.toml
```

Définissez SystemdCgroup à true :
```
SystemdCgroup = true
```

Ou utilisez cette commande :
```
sudo sed -i -e 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

Redémarrez containerd :
```
sudo systemctl restart containerd
```

## Installer Kubernetes
Pour installer Kubernetes, utilisez les commandes suivantes :

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

## Désactiver le swap
Désactivez le swap en utilisant la commande suivante :

```
sudo swapoff -a
```

S'il y a des entrées de swap dans le fichier /etc/fstab, supprimez-les en utilisant un éditeur de texte comme nano :
```
sudo nano /etc/fstab
```

Activez les modules du noyau :
```
sudo modprobe br_netfilter
```

Ajoutez quelques paramètres à sysctl :
```
sudo sysctl -w net.ipv4.ip_forward=1
```
## Initialiser le Cluster (à exécuter uniquement sur le maître):
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Créez un répertoire .kube dans votre répertoire personnel :
```
mkdir -p $HOME/.kube
```

Copiez le fichier de configuration Kubernetes dans votre répertoire personnel :
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

Changez les permissions du fichier :
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Installer Flannel (à exécuter uniquement sur le maître)
Utilisez la commande suivante pour installer Flannel :

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## Vérifier l'installation
Vérifiez que tous les pods sont en cours d'exécution :

```
kubectl get pods --all-namespaces
```

## Ajouter des nœuds
Pour ajouter des nœuds au cluster, exécutez la commande kubeadm join avec les arguments appropriés sur chaque nœud. La commande fournira un token qui pourra être utilisé pour joindre le nœud au cluster.
