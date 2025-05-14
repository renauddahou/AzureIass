Un **Availability Set** sur Azure est un mécanisme permettant d'assurer la haute disponibilité des machines virtuelles (VMs). Il fonctionne en répartissant les VMs sur plusieurs **Fault Domains** et **Update Domains**, afin de minimiser les risques d'indisponibilité en cas de panne ou de mise à jour.

### 🔹 **Concepts clés d'un Availability Set** :
1. **Fault Domain (FD) :**  
   - Correspond à un groupe de VMs qui partagent le même matériel physique (rack, alimentation, réseau).  
   - Azure garantit que les VMs d'un Availability Set sont réparties sur au moins **2 à 3 Fault Domains**, évitant ainsi qu'une panne matérielle affecte toutes les VMs en même temps.

2. **Update Domain (UD) :**  
   - Désigne un groupe de VMs qui peuvent être mises à jour ou redémarrées en même temps.  
   - Azure répartit les VMs sur plusieurs Update Domains (jusqu’à 5 par défaut), afin que les mises à jour de maintenance ne redémarrent pas toutes les VMs d'un coup.

### 🎯 **Pourquoi utiliser un Availability Set ?**
✅ Éviter un **Single Point of Failure (SPOF)**  
✅ Assurer une **tolérance aux pannes matérielles**  
✅ Réduire les interruptions lors des **mises à jour d’Azure**  
✅ Bénéficier du **Service Level Agreement (SLA) de 99,95 %** de disponibilité pour les VMs dans un Availability Set  

### 📌 **Limitations & Remarques** :
- Un **Availability Set ne fonctionne que dans une seule zone de disponibilité**. Si on veut une redondance entre différentes régions, il faut utiliser **Azure Availability Zones**.
- Il est nécessaire de **créer un Availability Set avant de déployer les VMs** (on ne peut pas l'ajouter après).


** - Methode template**


````
az group create -l westeurope -n renaudgroup

az deployment group create --resource-group renaudgroup --template-file availabilitySet.json
````

** - AZCLI  créer un Availabilityset et ajouter une vm**

```
$RESOURCE_GROUP="renaudgroup"
$AVAILABILITY_SET_NAME="MyAvailabilitySet"
$LOCATION="westeurope"  # Remplace par la région souhaitée

az vm availability-set create --name $AVAILABILITY_SET_NAME --resource-group $RESOURCE_GROUP --location $LOCATION --platform-fault-domain-count 2 --platform-update-domain-count 2


az vm create \
  --resource-group renaudgroup \
  --name MyVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --size Standard_DS1_v2 \
  --availability-set MyAvailabilitySet


az vm create --resource-group renaudgroup --name MyVM --image Ubuntu2204 --admin-username azureuser --ssh-key-values "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCXhkjJATVZ4gwS+cfk3UuUO3sY1J3NDuDOElTDicaAgAA6STmOIyhYMzjP0Cp0W7zhaXo9xHsCPJBGRSMH0I1ZtaqrGFQV43J7eOsrZ2bdzqCsbj8qstxxMigfaZ4ds9DSE8Od/KflNx/SWlgPCps8gJAfDtzvrmnbUB88h25ZboVHODkpeb3BhqeoNrG+qZ4bypgVqv8A/FpUqM/Tse43Tt743kY2KdF2HBHbXGfpHshtfTOI09qY77O6Ua3ew/ZyhoVUtBstQuhZZj67gqOt4aZS7iiqaxvFvpr7PWnqc7iHZfUGsh5HrBepwobod3ONneieLZypLBE6V6xTq7pHqa8eFVCZZtxCBk8tHIPd9WBxxvbsope1SOl5vbSjr2+iB6lL4We+PRUyuvyYZZIfpu+JECbYYOoef4uDgvpTnKRBTm4FsUwsFsArSnvdSuMxTUz7V3bVTYJxJuDRf0qxxeJI541yCMFjUWUpwY/uu8n9qFBUEeytzSXDI/fvyi0= generated-by-azure" --size Standard_DS1_v2 --availability-set MyAvailabilitySet


ssh -i trainvm_key.pem azureuser@52.166.99.8

````



**Methode template et ajouter une vm**

````
az deployment group create --resource-group renaudgroup --template-file ./template_availableset.json --parameters ./parameters.json

az vm image list --output table --publisher Canonical # voir les image disponible en cas d'erreur d'image

````

**Utiliser un Availability Set :** Si vous souhaitez utiliser un Availability Set, vous devez le spécifier sans inclure les Availability Zones.

**Utiliser une Availability Zone :** Si vous souhaitez utiliser une Availability Zone, vous devez retirer les configurations relatives à l'Availability Set
