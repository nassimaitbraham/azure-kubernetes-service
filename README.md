<h1> Création d'un kluster Kubernetes et deploiement d'une application springboot dockerisée</h2>

Répertoire : azure-kubernetes-service -> Kubernetes-Cluster-SpringBootApp-docker <br/>


<h2> 1 - Create sshKeyResourceGroup </h2>

<p>nas@Azure:~$ az group create -l eastus -n sshKeyResourceGroup </p>
nas@Azure:~$ az group create -l eastus -n kubernetes <br/>



<h2> 2 - Create ssk key </h2>

nas@Azure:~$ az sshkey create --location "eastus" --resource-group "sshKeyResourceGroup" --name "mySSHKeyForAKS"<br/>
	 
Return : <br/>
{<br/>
	"id": "/subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/SSHKEYRESOURCEGROUP/providers/Microsoft.Compute/sshPublicKeys/mySSHKeyForAKS",<br/>
	"location": "eastus",<br/>
	"name": "mySSHKeyForAKS",<br/>
	"publicKey": "Cle a copier dans playbook",<br/>
	"resourceGroup": "SSHKEYRESOURCEGROUP",<br/>
	"tags": null,<br/>
	"type": null<br/>
}<br/>
<h2> 3 - Creation d'une app register (Azure Portail) </h2>

a - Azure active directory -> App registrations -> New registrations<br/>
Name : Kubernetes
Supported account types : Accounts in this organizational directory only (Répertoire par défaut only - Single tenant)<br/>
		
b - Register<br/>
C - Azure active directory -> App registrations -> Owned applications -> Select application<br/>
D - Certificates & secrets -> New client secret<br/>
	
Description : kubernetesSecretClient<br/>
Value : XXXX.XXXXX.XXX-X.XXXXXXXX.XXXXXXXX<br/>
	 
<h2> 4 - Connexion au cluster Kubernetes avec kubectl </h2>

nas@Azure:~$ az aks get-credentials -g kubernetes -n myAKSCluster<br/>
		
<h2> 5 - Lancer le script : </h2>
 
nas@Azure:~$ ansible-playbook ansible_create_cluster.yaml<br/>
	 
<h2> 5 - Reccupéré la liste des nodes </h2>
 
nas@Azure:~$ kubectl get nodes<br/>
Result : 2 noeuds<br/>
				
NAME                     | STATUS   |ROLES  |AGE     |VERSION<br/>
aks-default-11482510-0   | Ready    |agent  |3m56s   | v1.19.9  ==> IP : 10.240.0.4<br/>
aks-default-11482510-1   | Ready    |agent  | 3m46s  |v1.19.9  ==> IP : 10.240.0.5<br/>
			
		
<h2>  6 - Deploiement de pods </h2>
    
nas@Azure:~$ kubectl apply -f azure_deployment.yaml<br/>
deployment.apps/spring-kubernetes-deployment created<br/>
service/spring-kubernetes created<br/>
		
<h2>  7 - Affichage des pods </h2>

nas@Azure:~$ kubectl get pods<br/>
		
NAME                                            |READY   |STATUS    |RESTARTS   |AGE<br/>
spring-kubernetes-deployment-6ccfb4f579-tt47h   |1/1     |Running   |0          |112s     ==> 10.240.0.4<br/>
spring-kubernetes-deployment-6ccfb4f579-v22p5   |1/1     |Running   |0          |112s     ==> 10.240.0.4<br/>
spring-kubernetes-deployment-6ccfb4f579-x5lns   |1/1     |Running   |0          |112s     ==> 10.240.0.5<br/>
		
<h2>  8 - Affichage des service </h2>

nas@Azure:~$ kubectl get service<br/>
		
NAME                |TYPE           |CLUSTER-IP    |EXTERNAL-IP     |PORT(S)          |AGE<br/>
kubernetes          |ClusterIP      |10.0.0.1      |<none>          |443/TCP          |12m<br/>
spring-kubernetes   |LoadBalancer   |10.0.218.87   |xx.xxxx.xxxx.xx |8080:30353/TCP   |3m24s<br/>
		
<h2>  9 - Affichage des podes </h2>	
	
nas@Azure:~$ kubectl describe pods<br/>
		
<a></a>
![Node-1](https://user-images.githubusercontent.com/5339905/127657674-f5c9ac4e-ad15-44f9-89b5-516cb9a716e5.jpg)<br/>

![node-2](https://user-images.githubusercontent.com/5339905/127658605-50b0d069-c861-4f9c-8843-be8511fe060c.jpg)<br/>

![pods-3](https://user-images.githubusercontent.com/5339905/127658661-60a67043-7dea-4a8d-80d3-394eb0f6974a.jpg)<br/>
		
		
<h2>  10 - Suppression du noeud : aks-default-11482510-1</h2>	
	
- Suppression d'un nodes AKS pour simuler un worker nodeà l'état down<br/>
		
nas@Azure:~$ kubectl delete nodes aks-default-11482510-1   ==> IP : 10.240.0.5 <br/>
			
- On a qu'un seul nodes qui reste<br/>

nas@Azure:~$ kubectl get nodes (Il ne reste plus qu'un worker nodes dans le cluster)<br/>
			
NAME                     STATUS   ROLES   AGE   VERSION<br/>
aks-default-11482510-0   Ready    agent   42m   v1.19.9 ==> IP : 10.240.0.4<br/>
			
- Après un court instant on regarde combien de pods on a dans notre cluster :<br/>
		
nas@Azure:~$ kubectl get pods<br/>
			
NAME                                            |READY   |STATUS    |RESTARTS   |AGE<br/>
spring-kubernetes-deployment-6ccfb4f579-r6p4t   |1/1     |Running   |0          |84s    ==> 10.240.0.4<br/>
spring-kubernetes-deployment-6ccfb4f579-tt47h   |1/1     |Running   |0          |36m    ==> 10.240.0.4<br/>
spring-kubernetes-deployment-6ccfb4f579-v22p5   |1/1     |Running   |0          |36m    ==> 10.240.0.4<br/>
	
<a></a>		
On remarque que on a toujorus trois podes ce qui prouve que le master nodes a démarré un nouveau pod vu que le champ replicas du déploiement n'est pas satisfait.<br/>
On regarde a quel node appartiennent les podes:<br/>
				
nas@Azure:~$ kubectl describe pods<br/>
<a></a>

![pods-1-1](https://user-images.githubusercontent.com/5339905/127658876-719adea8-7c94-4a14-a0a7-45869e754ca8.jpg)<br/>
![pods-2-2](https://user-images.githubusercontent.com/5339905/127658894-2d398527-7a23-4a27-99e0-0d4165373d6d.jpg)<br/>
![pods-3-3](https://user-images.githubusercontent.com/5339905/127658909-ee0dfb02-47c0-4377-babc-5889e01df5b5.jpg)<br/>

On voit bien un nouveau pods "spring-kubernetes-deployment-6ccfb4f579-r6p4t" qui été de nouveau crée mais dans le node restant<br/> 

<h2>  11 - Delete resource group </h2>
as@Azure:~$ az group delete --name kubernetes<br/>
as@Azure:~$ az group delete --name MC_kubernetes_myAKSCluster_eastus<br/>
	
<h2>  12 - Intégration d’Azure Active Directory géré par AKS </h2>
a - Créer un group AD pour les administrateurs de cluster</br>
    nas@Azure:~$ az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup</br></br>
	
b- Vérifier que le groupe admin a été crée.</br></br>
    nas@Azure:~$ az ad group list --filter "displayname eq 'myAKSAdminGroup'" -o table</br>
    Return :</br>
	DisplayName      MailEnabled    MailNickname     ObjectId                              ObjectType    Odata.type                         SecurityEnabled</br>
	---------------  -------------  ---------------  ------------------------------------  ------------  ---------------------------------  -----------------</br>
	myAKSAdminGroup  False          myAKSAdminGroup  ba2cc1eb-8673-4887-a60e-d16ff8add4ae  Group         Microsoft.DirectoryServices.Group  True</br></br>
C- Activer l’intégration de Azure AD géré par AKS sur notre cluster</br>
<p></p>
    nas@Azure:~$ az aks update -g kubernetes -n myAKSCluster --enable-aad --aad-admin-group-object-ids ba2cc1eb-8673-4887-a60e-d16ff8add4ae --aad-tenant-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx <br/><br/>
Return :</br>
 {<br/>
  "aadProfile": {<br/>
    "adminGroupObjectIDs": [<br/>
      "ba2cc1eb-8673-4887-a60e-d16ff8add4ae"<br/>
    ],<br/>
    "clientAppId": null,<br/>
    "enableAzureRbac": null,<br/>
    "managed": true,<br/>
    "serverAppId": null,<br/>
    "serverAppSecret": null,<br/>
    "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"<br/>
  },<br/>
  ...<br/>
  }<br/>
 D - Se connecter au cluster AKS<br/><br/><br/>
     nas@Azure:~$ az aks get-credentials --resource-group kubernetes --name myAKSCluster<br/>
     A different object named clusterUser_kubernetes_myAKSCluster already exists in your kubeconfig file.<br/>
     Overwrite? (y/n): y<br/>
 E - Affichage de la liste des nodes<br/>
     <p></p>
     nas@Azure:~$ kubectl get nodes<br/>
     To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AZERTYUIO to authenticate.<br/>
     - Aller sur le lien afficher<br/>
     - Renseigner le code : AZERTYUIO<br/>
     - Choisir un compte pour ce connecter<br/>
     - Confirmer sur continuer<br/>
     	Affichge sur AZ CLI:<br/> 
	<p></p>
 	nas@Azure:~$ kubectl get nodes<br/>
     	To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code DQDK7D26X to authenticate.<br/>
     	Error from server (Forbidden): nodes is forbidden: User "462c842d-3cc6-4d7b-a76b-b6ce41a6c81f" cannot list resource "nodes" in API group "" at the cluster scope.<br/>
     ==> Le compte que je viens d'utiliser n'a pas le groupe d'admin : myAKSAdminGroup<br/><br/>
  F - Ajout de mon compte au groupe : myAKSAdminGroup<br/><br/>
       <p></p>
  	nas@Azure:~$ az ad group member add --group myAKSAdminGroup --member-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx<br/><br/>
  G - Affichage de la liste de nodes du cluster AKS<br/><br/>
  	<p></p>
  	nas@Azure:~$ kubectl get nodes
	To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code CKEWSZRFU to authenticate.<br/><br/>
	<p></p>
	NAME                     STATUS   ROLES   AGE   VERSION<br/>
	aks-default-11482510-0   Ready    agent   36m   v1.19.9<br/>
	<p></p>
	==> On a bien à présent les autorisations sur les nodes du cluster AKS.<br/>
  
     
     
     





