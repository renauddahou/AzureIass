
### **Objectif du TP :**
- **Comprendre** les concepts des Service Principals et des Identités Managées.
- **Apprendre** à créer et utiliser un Service Principal pour accéder aux ressources Azure.
- **Apprendre** à configurer une Identité Managée pour une machine virtuelle et accéder à des ressources Azure sans gestion explicite de secrets.
- **Comprendre** quand et pourquoi choisir l'une ou l'autre méthode en fonction des besoins.

---

### **Prérequis :**
- Un **compte Azure** actif.
- L'outil **Azure CLI** installé sur votre machine locale.
- Une **machine virtuelle** ou un **App Service** créé sur Azure.


### **Scénario 1 : Utiliser un Service Principal pour accéder à un Azure Key Vault**

**But :** Créer un Service Principal pour une application, puis l'utiliser pour accéder à un Key Vault sécurisé.

#### Étape 1 : Créer un Service Principal

1. Ouvrez une fenêtre de terminal ou PowerShell.
2. Créez un Service Principal pour une application en utilisant la commande Azure CLI suivante :

   ```
   az ad sp create-for-rbac --name "MyAppServicePrincipal" --role "Reader" --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}
   ```
   Remplacez `{subscription-id}` et `{resource-group-name}` par vos valeurs.
3. Cette commande retournera un JSON avec les informations suivantes :
   - `appId`: Identifiant de l'application (Service Principal).
   - `tenant`: Identifiant du tenant.
   - `password`: Clé secrète pour l'authentification.

4. **Notez** les informations importantes retournées (en particulier l'`appId` et le `password`) car vous en aurez besoin pour configurer l'authentification de votre application.

#### Étape 2 : Configurer l'accès au Key Vault avec le Service Principal

1. Créez un **Key Vault** dans Azure si vous n'en avez pas déjà un :

   ```
NB: Utilisez un nom unique pour votre keyvault

   az keyvault create --name "MyKeyVaultrenaud" --resource-group {resource-group-name} --location {location}

   az keyvault create --name "MyKeyVaultrenaud" --resource-group TP_Ressources --location westeurope
   ```
2. Dans le **Key Vault**, stockez un secret pour tester l'accès :

 
   az keyvault secret set --vault-name "MyKeyVaultrenaud" --name "TestSecret" --value "MySecretValue"
   ```

![Texte alternatif](./images/i30.PNG)
![Texte alternatif](./images/i31.png)


3. **Attribuez** au Service Principal l'accès à ce Key Vault :
   ```
   az keyvault set-policy --name "MyKeyVaultrenaud" --spn {appId} --secret-permissions get
   ```

4. Vérifiez que l'application peut accéder au secret avec le Service Principal :
   - Utilisez un script ou une application (en Python, Node.js, etc.) qui utilise l'`appId` et `password` pour s'authentifier auprès de Azure et récupérer le secret du Key Vault.



### **Scénario 2 : Utiliser une Identité Managée pour accéder à un Azure Key Vault depuis une Machine Virtuelle**

**But :** Utiliser une identité managée pour accéder à un Key Vault depuis une machine virtuelle sans gérer de secrets.

#### Étape 1 : Créer une machine virtuelle avec une Identité Managée

1. Créez une machine virtuelle sur Azure et activez son identité managée par le système.

 ```
   az vm create --name MyVM --resource-group {resource-group-name} --image UbuntuLTS --assign-identity --admin-username azureuser --generate-ssh-keys

 
az vm create --resource-group TP_Ressources --name MyVM --image Ubuntu2204 --admin-username azureuser --ssh-key-values "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCXhkjJATVZ4gwS+cfk3UuUO3sY1J3NDuDOElTDicaAgAA6STmOIyhYMzjP0Cp0W7zhaXo9xHsCPJBGRSMH0I1ZtaqrGFQV43J7eOsrZ2bdzqCsbj8qstxxMigfaZ4ds9DSE8Od/KflNx/SWlgPCps8gJAfDtzvrmnbUB88h25ZboVHODkpeb3BhqeoNrG+qZ4bypgVqv8A/FpUqM/Tse43Tt743kY2KdF2HBHbXGfpHshtfTOI09qY77O6Ua3ew/ZyhoVUtBstQuhZZj67gqOt4aZS7iiqaxvFvpr7PWnqc7iHZfUGsh5HrBepwobod3ONneieLZypLBE6V6xTq7pHqa8eFVCZZtxCBk8tHIPd9WBxxvbsope1SOl5vbSjr2+iB6lL4We+PRUyuvyYZZIfpu+JECbYYOoef4uDgvpTnKRBTm4FsUwsFsArSnvdSuMxTUz7V3bVTYJxJuDRf0qxxeJI541yCMFjUWUpwY/uu8n9qFBUEeytzSXDI/fvyi0= generated-by-azure" --size Standard_DS1_v2  --assign-identity

 ```

2. Activez l'identité managée pour cette machine virtuelle :

   ```
   az vm identity assign --name MyVM --resource-group {resource-group-name}
   az vm identity assign --name MyVM --resource-group  TP_Ressources
   ```

3. Vérifiez que l'identité managée est bien assignée à la machine virtuelle :

   ```
   az vm identity show --name MyVM --resource-group {resource-group-name}
   az vm identity show --name MyVM --resource-group TP_Ressources
   ```
![Texte alternatif](./images/i32.png)
#### Étape 2 : Accéder au Key Vault avec l'Identité Managée

1. Retournez dans le **Key Vault** et attribuez-lui l'accès via l'identité managée de la VM :

```
az keyvault set-policy --name "MyKeyVaultrenaud" --object-id {object-id} --secret-permissions get

az vm show --name MyVM --resource-group TP_Ressources --query identity.principalId --output tsv
az keyvault set-policy --name "MyKeyVaultrenaud" --object-id 64983b02-3bc2-4bdf-bf50-11e75a27e2cd --secret-permissions get

```
```
az keyvault show --name "MyKeyVaultrenaud" --query "properties.accessPolicies"
```
![Texte alternatif](./images/i33.png)

2. Connectez-vous à la machine virtuelle via SSH :

   ```
   ssh azureuser@{vm-ip}
   ```

3. Utilisez la commande Azure CLI sur la VM pour tester l'accès au Key Vault avec l'identité managée :

```
sudo apt update && sudo apt install -y ca-certificates curl apt-transport-https lsb-release gnupg

curl -sL https://packages.microsoft.com/keys/microsoft.asc | sudo gpg --dearmor -o /usr/share/keyrings/microsoft.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/azure-cli.list

sudo apt update && sudo apt install -y azure-cli

az login --identity --allow-no-subscriptions

az keyvault secret show --name "TestSecret" --vault-name "MyKeyVaultrenaud"
````

4. Vérifiez que vous pouvez récupérer le secret sans utiliser de clés ou de certificats manuellement dans le code.



**Attribuer role nécéssaire à la VM (en cas de soucis mais peu probable)**

```
az role assignment create --assignee f54bd873-831c-4033-90de-7d7abf27d5fe --role "Reader" --scope /subscriptions/2f8f37d3-c9e1-4092-bab8-7990c96abfe8/resourceGroups/TP_Ressources
````
---

### **Scénario 3 : Comparaison entre SPN et Identité Managée**

1. **Service Principal** : Utilisé dans des scénarios où vous avez une application ou un service en dehors d'Azure, ou si vous devez gérer manuellement les secrets d'authentification. Par exemple, une application qui doit interagir avec Azure Key Vault depuis un serveur sur site.
2. **Identité Managée** : Utilisée pour simplifier l'authentification lorsque vous utilisez des services Azure, comme une machine virtuelle ou un App Service, et ne souhaitez pas gérer les secrets vous-même.

---

### **Évaluation des compétences** :
- Créer et configurer des **Service Principals** pour une application dans Azure.
- Utiliser un **Service Principal** pour accéder à des ressources sécurisées dans Azure (Key Vault).
- Créer une **machine virtuelle** avec une **identité managée**.
- Accéder à des ressources Azure via **identité managée** sans nécessiter de gestion de secrets.

---

### **Conclusion :**
À la fin de ce TP, vous aurez acquis des compétences pratiques sur la gestion des identités dans Azure, vous comprendrez quand utiliser un **Service Principal** ou une **Identité Managée**, et vous serez en mesure de configurer et gérer l'authentification sécurisée pour vos applications et services Azure.