#
##git clone repo :

need to install local:
kubectl:                  -- https://kubernetes.io/docs/tasks/tools/install-kubectl/
aws-iam-authenticator:    -- https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html
terraform:                -- https://learn.hashicorp.com/terraform/getting-started/install.html
helm:                     -- https://helm.sh/docs/using_helm/

##terraform to setup eks with two worker nodes on Mutli AZ  
cd eks-cluster/
terraform init  
terraform plan
terraform apply


terraform output kubeconfig > ../kubernetes/kubeconfig
terraform output config-map-aws-auth > ../kubernetes/aws-auth.yaml


#kuberntes setup to create dashboard,tomcat and couchdb using helm.

cd ../kubernetes/

export KUBECONFIG=kubeconfig
kubectl config current-context

kubectl get svc -->to check k8s service running

kubectl get nodes 

kubectl label node <nodename> app=worker

kubectl apply -f aws-auth.yaml


#tiller service account for helm (pods on multiAZ)

kubectl create -f tillerRBAC.yaml

helm init --service-account tiller



#install dashboard using helm

helm install --name DashBoard ./kubernetes-dashboard  --namespace kube-system

kubectl get svc -n kube-system  -- to get external loadbalancer dns name

check: https://<externalip or loadbalancer dns>

login token: kubectl describe secret k8dashboard-kubernetes-dashboard-token-7725b -n kube-system
 
Above serviceaccount have limited access on DashBoard. If you nedd complete access create serviceaccount and Bind to clusterroleBinding(clusterRole), then will get full access.


#install tomcat -- pods on mutliAZs (pvc as EBS volumes)

helm install --name tomcat ./tomcat

kubectl get svc -- to get external loadbalancer dns name

check: https://<externalip or loadbalancer dns>



#install couchdb using statefulset On mutliAZS with volumeClaimTemplates( EBS volumes will create accordingly to it  pods multiAZ )

helm install --name couchdb ./couchdb

kubectl get svc -- to get external loadbalancer dns name

check: https://<externalip or loadbalancer dns>:5984/_utils/index.html



# to check EBS volumes as pvc
 kubectl get pvc

##HELM:

helm we can add our github as repo


helm repo add helm-example https://example.io/helm-example
