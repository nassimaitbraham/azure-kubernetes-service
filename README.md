<h1> Création d'un kluster Kubernetes et deploiement d'une application springboot dockerisée</h2>
     
	 Répertoire : Kubernetes -> Create-cluster-deploy-springboot-docker-images

<h2> 1 - Create sshKeyResourceGroup </h2>

	nas@Azure:~$ az group create -l eastus -n sshKeyResourceGroup
	nas@Azure:~$ az group create -l eastus -n kubernetes

<h2> 2 - Create ssk key </h2>

	nas@Azure:~$ az sshkey create --location "eastus" --resource-group "sshKeyResourceGroup" --name "mySSHKeyForAKS"
	 
    Return : 
	 {
		"id": "/subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/SSHKEYRESOURCEGROUP/providers/Microsoft.Compute/sshPublicKeys/mySSHKeyForAKS",
		"location": "eastus",
		"name": "mySSHKeyForAKS",
		"publicKey": "Cle a copier dans playbook",
		"resourceGroup": "SSHKEYRESOURCEGROUP",
		"tags": null,
		"type": null
	}
<h2> 3 - Creation d'une app register (Azure Portail) </h2>

	a - Azure active directory -> App registrations -> New registrations
		Name : Kubernetes
		Supported account types : Accounts in this organizational directory only (Répertoire par défaut only - Single tenant)
		
	b - Register
	C - Azure active directory -> App registrations -> Owned applications -> Select application
	D - Certificates & secrets -> New client secret
	
	    Description : kubernetesSecretClient
		Value : XXXX.XXXXX.XXX-X.XXXXXXXX.XXXXXXXX
	 
<h2> 4 - Connexion au cluster Kubernetes avec kubectl </h2>

		nas@Azure:~$ az aks get-credentials -g kubernetes -n myAKSCluster
		
<h2> 5 - Lancer le script : </h2>
 
		nas@Azure:~$ ansible-playbook ansible_create_cluster.yaml
	 
<h2> 5 - Reccupéré la liste des nodes </h2>
 
		nas@Azure:~$ kubectl get nodes
		Result : 2 noeuds
				
			NAME                     STATUS   ROLES   AGE     VERSION
			aks-default-11482510-0   Ready    agent   3m56s   v1.19.9  ==> IP : 10.240.0.4
			aks-default-11482510-1   Ready    agent   3m46s   v1.19.9  ==> IP : 10.240.0.5
			
		
<h2>  6 - Deploiement de pods </h2>
    
		nas@Azure:~$ kubectl apply -f azure_deployment.yaml
		
	    deployment.apps/spring-kubernetes-deployment created
        service/spring-kubernetes created
		
<h2>  7 - Affichage des pods </h2>

		nas@Azure:~$ kubectl get pods
		
		NAME                                            READY   STATUS    RESTARTS   AGE
		spring-kubernetes-deployment-6ccfb4f579-tt47h   1/1     Running   0          112s     ==> 10.240.0.4
		spring-kubernetes-deployment-6ccfb4f579-v22p5   1/1     Running   0          112s     ==> 10.240.0.4
		spring-kubernetes-deployment-6ccfb4f579-x5lns   1/1     Running   0          112s     ==> 10.240.0.5
		
<h2>  8 - Affichage des service </h2>

		nas@Azure:~$ kubectl get service
		
		NAME                TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
		kubernetes          ClusterIP      10.0.0.1      <none>          443/TCP          12m
		spring-kubernetes   LoadBalancer   10.0.218.87   52.152.137.84   8080:30353/TCP   3m24s
		
<h2>  9 - Affichage des podes </h2>	
	
		nas@Azure:~$ kubectl describe pods
					
		![Node-1](https://user-images.githubusercontent.com/5339905/127657114-8e2fbe1e-6c6e-4e31-87d7-047547ff1875.jpg)
		
		![node-2](https://user-images.githubusercontent.com/5339905/127657139-ebe2ce5f-2b77-489b-a311-f4c9a60dd8b2.jpg)
		
		![pods-3](https://user-images.githubusercontent.com/5339905/127657147-d20a89f7-9d35-42f4-9fb5-04dae9d474d9.jpg)


<h2>  10 - Suppression du noeud : aks-default-11482510-1</h2>	
		
		- Suppression d'un nodes AKS pour simuler un worker nodeà l'état down
		
			nas@Azure:~$ kubectl delete nodes aks-default-11482510-1   ==> IP : 10.240.0.5 
			
		- On a qu'un seul nodes qui rest	
		
			nas@Azure:~$ kubectl get nodes (Il ne reste plus qu'un worker nodes dans le cluster)
			
			NAME                     STATUS   ROLES   AGE   VERSION
			aks-default-11482510-0   Ready    agent   42m   v1.19.9 ==> IP : 10.240.0.4
			
		- Après un court instant on regarde combien de pods on a dans notre cluster :
		
		    nas@Azure:~$ kubectl get pods
			
			NAME                                            READY   STATUS    RESTARTS   AGE
			spring-kubernetes-deployment-6ccfb4f579-r6p4t   1/1     Running   0          84s    ==> 10.240.0.4
			spring-kubernetes-deployment-6ccfb4f579-tt47h   1/1     Running   0          36m    ==> 10.240.0.4
			spring-kubernetes-deployment-6ccfb4f579-v22p5   1/1     Running   0          36m    ==> 10.240.0.4
			
		==> On remarque que on a toujorus trois podes ce qui prouve que le master nodes a démarré un nouveau pod vu que le champ replicas du déploiement n'est pas satisfait.
			
			On regarde a quel node appartiennent les podes:
				
				nas@Azure:~$ kubectl describe pods <br/>
				
				![pods-2-2](https://user-images.githubusercontent.com/5339905/127657261-b5432f82-71f7-469c-87ae-c745b625ab0d.jpg)
				![pods-1-1](https://user-images.githubusercontent.com/5339905/127657230-3333e151-9f9e-48f3-81a9-db7a55dfb204.jpg)
				![pods-3-3](https://user-images.githubusercontent.com/5339905/127657295-697d0392-13cd-4dd0-a591-3026aed5533f.jpg)


			
			On voit bien un nouveau pods "spring-kubernetes-deployment-6ccfb4f579-r6p4t" qui été de nouveau crée mais dans le node restant 

<h2>  11 - Delete resource group </h2>
			as@Azure:~$ az group delete --name kubernetes
			as@Azure:~$ az group delete --name MC_kubernetes_myAKSCluster_eastus



