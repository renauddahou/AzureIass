#### Attribution des rôles via un Service Principal (SPN) et Azure Active Directory (AAD)
1. **Création d'un Service Principal (SPN)** :
   - Allez dans **Azure Active Directory** et créez un Service Principal (SPN) pour représenter une application ou un service.
   
2. **Attribuer des rôles via le Service Principal** :
   - Allez dans **Contrôle d’accès (IAM)** et attribuez un rôle spécifique au SPN. Par exemple, attribuer le rôle "Contributeur de machine virtuelle" à un SPN pour qu'il puisse gérer les ressources.
   
3. **Test d’attribution des rôles via AAD** :
   - Utilisez le SPN pour exécuter une tâche nécessitant un rôle spécifique (par exemple, déployer une machine virtuelle) et vérifiez les permissions.

![Texte alternatif](./images/i19.png)

![Texte alternatif](./images/i24.png)

![Texte alternatif](./images/i25.png)

![Texte alternatif](./images/i26.png)

![Texte alternatif](./images/i27.png)

![Texte alternatif](./images/i28.png)

![Texte alternatif](./images/i29.png)

**AZ CLI**

```
Commande pour créer un Service Principal (SPN) et lui attribuer un rôle:

az ad sp create-for-rbac --name <AppName> --role "Contributeur de machine virtuelle" --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}


az ad sp create-for-rbac --name my_s3_spn --role "Virtual machine contributor" --scopes /subscriptions/2f8f37d3-c9e1-4092-bab8-7990c96abfe8/resourceGroups/TP_Ressources


{
  "appId": "0d54935c-f91c-4ce1-bd62-5af0ea3c881e",
  "displayName": "my_s3_spn",
  "password": "_Xb8Q~clT6~0jRiNiA7VTsxhbkS2RXZs3TtiDchV",
  "tenant": "ec908c0d-0e11-421c-ba92-c90cc696e789"
}

````


**Cela permet à des applications ou services de s'authentifier et d'opérer avec des privilèges définis par les rôles dans Azure Active Directory.**

````
az login --service-principal -u <appId> -p <password> --tenant <tenant>
````

````
pip install azure-identity azure-mgmt-resource

from azure.identity import ClientSecretCredential
from azure.mgmt.resource import SubscriptionClient

# Remplissez les informations depuis votre JSON
tenant_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
client_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
client_secret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Authentification via Service Principal
credential = ClientSecretCredential(
    tenant_id=tenant_id,
    client_id=client_id,
    client_secret=client_secret
)

# Créer un client pour interagir avec les ressources Azure (exemple : SubscriptionClient)
subscription_client = SubscriptionClient(credential)

# Exemple : Liste des abonnements Azure
subscriptions = subscription_client.subscriptions.list()

for subscription in subscriptions:
    print(subscription.subscription_id)
````
