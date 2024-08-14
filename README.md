# mongodbrepo
this repo include all the steps need to create and deploy mongodb along with opsmanager
This is for RHEL 9. Have 50G space and 4 CPU. Typically t2.xlarge instance. 
#any line starting with '#' is NOT a command

#untar the downloaded file


tar -zxvf kube.tar.gz


#Install Docker first. 

sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl start docker; sudo systemctl enable docker;

#Install Docker Engine:

sudo yum install -y docker-ce docker-ce-cli containerd.io


#Start and enable Docker:

sudo systemctl start docker
sudo systemctl enable docker

#Installing kind:
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/bin/kind


#Installing kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/bin/kubectl

#install git if missing
yum install git

#Install kind cluster
curl -Lo kind-config.yaml https://raw.githubusercontent.com/bhartiroshan/OpsManager/master/kind-config.yaml

#create the kind cluster
kind create cluster --config kind-config.yaml --retain --image "kindest/node:v1.21.14"

kubectl cluster-info --context kind-kind
git clone https://github.com/mongodb/mongodb-enterprise-kubernetes.git
kubectl create namespace mongodb
kubectl apply -f mongodb-enterprise-kubernetes/crds.yaml
kubectl apply -f mongodb-enterprise-kubernetes/mongodb-enterprise.yaml
#Cluster will be ready in 2-3 minutes

#In the current repo, opsmanager version configured is 6.0.15 with appdb 6.0.4-ent.

#First create a secret for credentials.
kubectl create secret generic ops-manager-admin-secret \
  --from-literal=Username="testuser@test.com" \
  --from-literal=Password="Testdeploy@123" \
  --from-literal=FirstName="test" \
  --from-literal=LastName="user"

kubectl apply -f opsmanager.yaml
