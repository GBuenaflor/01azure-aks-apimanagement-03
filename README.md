----------------------------------------------------------
# Use Azure API Management with microservices (WCF and Web API) deployed in AKS - Episode 03

[Episode1](https://github.com/GBuenaflor/01azure-aks-apimanagement/) - Build the infrastructure using Azure Terraform and Generate the Lets Encrypt Certificate 

[Episode2](https://github.com/GBuenaflor/01azure-aks-apimanagement-02/) - Create and contenerize ASP.Net Core Web API and WCF app then deploy to AKS ( Windows and Linux Node Pool)

> Episode3 - Configure API Management External / Internal Endpoints and publish API's that runs from AKS
 
----------------------------------------------------------
### High Level Architecture Diagram for the 3 Episodes:

![Image description](https://github.com/GBuenaflor/01azure-aks-apimanagement/blob/master/Images/GB-AKS-API02C.png)

----------------------------------------------------------

#### Episode 3 - Configure API Management External / Internal Endpoints and publish API's that runs from AKS

1. Publish the WCF Service Application deployed from AKS to API Management Service
2. Publish the ASP.net Core Web API deployed from AKS to API Management Service
3. Configure the API Management External API Endpoints
4. Configure and test the interconnectivity the API Management Internal API Endpoints
 
----------------------------------------------------------
### 1. Publish the WCF Service Application deployed from AKS to API Management Service

#### 1.1 Option 1 - Connect API deployed from AKS to API Management using a Loadbalancer IP, configure Service section in the 03wcf-Ext-Int.yaml AKS manifest file

```diff
apiVersion: v1
kind: Service
metadata:
  labels: #PODS
    app: aks01-wcf
  name: aks01-wcf-ext
  namespace: default
  
spec:
  selector:
    app: aks01-wcf
  ports:
  - protocol: TCP
    port: 8084
    targetPort: 80 
  type: LoadBalancer  
```

#### 1.2 Option 2 - Connect API deployed from AKS to API Management using a NodePort IP, configure Service section in the 03wcf-Ext-Int.yaml AKS manifest file

```diff
apiVersion: v1
kind: Service
metadata:
  labels: #PODS
    app: aks01-wcf
  name: aks01-wcf-int
  namespace: default
  
spec:
  selector:
    app: aks01-wcf
  ports:
  - protocol: TCP
    port: 8084
    targetPort: 80
    nodePort: 30020 
  type: NodePort 
```


#### 1.3 Deploy the 03wcf-Ext-Int.yaml AKS manifest 

```
kubectl apply --namespace default -f "03wcf.yaml" --force
```

#### 1.4 In the API Management, create a blank API and configure the API Settings 


 ![Image description](https://github.com/GBuenaflor/01azure-aks-apimanagement-03/blob/master/Images/GB-AKS-API-E3-01A.png)

##### Note: You can also use the API custom DNS for the Webservice URL to utilize the nodes dynamically.
  
----------------------------------------------------------
### 2. Publish the ASP.net Core Web API deployed from AKS to API Management Service
 
#### 2.1 Option 1 - Connect API from AKS to API Management using a LoadBalancer IP, configure Service section in the 02webapi.yaml AKS manifest file

```diff
apiVersion: v1
kind: Service
metadata:
  labels: #PODS
    app: aks01-webapi
  name: aks01-webapi
  namespace: default
  
spec:
  selector:
    app: aks01-webapi
  ports:
  - protocol: TCP
    port: 8083
    targetPort: 80 
  type: LoadBalancer
```

#### 2.2 Option 2 - Connect API from AKS to API Management using a NodePort IP, configure Service section in the 02webapi.yaml AKS manifest file

```diff
apiVersion: v1
kind: Service
metadata:
  labels: #PODS
    app: aks01-webapi
  name: aks01-webapi
  namespace: default
  
spec:
  selector:
    app: aks01-webapi
  ports:
  - protocol: TCP
    port: 8083
    targetPort: 80 
  type: NodePort
```

#### 2.3 Deploy the 03wcf-Ext-Int.yaml AKS manifest 

```
kubectl apply --namespace default -f "02webapi.yaml" --force
```

#### 2.4 In the API Management,create a blank API and configure the API Settings 


 ![Image description](https://github.com/GBuenaflor/01azure-aks-apimanagement-03/blob/master/Images/GB-AKS-API-E3-02A.png)

 
##### Note: You can also use the API custom DNS for the Webservice URL to utilize two nodes dynamically.
 

----------------------------------------------------------
### 3. Configure the API Management External API Endpoints

#### 3.1 Configure the Azure DNS Zone  

##### Add new "A" Enrty used For API Service,
```
Type  : A
Name  : api
Value : 52.142.19.160 - [IP Address of API Management Private IP]
```


##### Add new "A" Enrty used For API Portal Service,
```
Type  : A
Name  : portal
Value : 52.142.19.160 - [IP Address of API Management Private IP]
```

#### 3.3 Configure the API Management Custom DNS

##### Custom DNS use for API Service
```
EndPoint    : Gateway
HostName    : api.ask01-web.xxxxx.net
Certificate : > Upload the .pfx certificate that we use in Episode 1
```

##### Custom DNS use for API Portal
```
EndPoint    : Developer portal
HostName    : portalEXT.ask01-web.xxxxx.net
Certificate : > Upload the .pfx certificate that we use in Episode 1
```


#### 3.4 Configure and test the interconnectivity of Azure DNS, App Gateway and API Management components

![Image description](https://github.com/GBuenaflor/01azure-aks-apimanagement-03/blob/master/Images/GB-AKS-API-E3-03.png)
 
##### View the external API 

![Image description](https://github.com/GBuenaflor/01azure-aks-apimanagement-03/blob/master/Images/GB-AKS-API-E3-04A.png)
 
----------------------------------------------------------
### 4. Configure and test the interconnectivity of API Management Internal API Endpoints

#####  Provisioned VM inside the VNet, edit the host File from C:\Windows\System32\drivers\etc.
#####  The 10.0.2.5 is the Private IP Address of API Management

```
10.0.2.5 az-apim01.azure-api.net      
10.0.2.5 az-apim01.portal.azure-api.net    
10.0.2.5 az-apim01.developer.azure-api.net
10.0.2.5 az-apim01.management.azure-api.net
10.0.2.5 az-apim01.scm.azure-api.net
```
##### The API Portal can be edited inside Network.
![Image description](https://github.com/GBuenaflor/01azure-aks-apimanagement-03/blob/master/Images/GB-AKS-API-E3-05A.png)
 
 
End of the Episode!...
------------------------------------------------------------------------------
 
#### Go to other Episodes:


[Episode1](https://github.com/GBuenaflor/01azure-aks-apimanagement/) - Build the infrastructure using Azure Terraform and Generate the Lets Encrypt Certificate 

[Episode2](https://github.com/GBuenaflor/01azure-aks-apimanagement-02/) - Create and contenerize ASP.Net Core Web API and WCF app then deploy to AKS ( Windows and Linux Node Pool)

> Episode3 - Configure API Management External / Internal Endpoints and publish API's that runs from AKS
 

------------------------------------------------------------------------------
 
Link to other Microsoft Azure projects
https://github.com/GBuenaflor/01azure
  
Note: My Favorite -> Microsoft :D
