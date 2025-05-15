
### **Scénario 1 : Utiliser un Service Principal avec Python**

Dans ce scénario, vous allez utiliser un **Service Principal** pour vous authentifier dans Azure et accéder à des ressources telles qu'un **Key Vault**.

#### Prérequis :
- Vous devez avoir déjà créé un **Service Principal** sur Azure.
- Vous devez disposer de l'`appId`, du `tenant` et du `password` générés lors de la création du Service Principal.

#### Étapes :

1. **Installer la bibliothèque Azure SDK pour Python** :

   Tout d'abord, installez les bibliothèques nécessaires :
   ```bash
   pip install azure-identity azure-keyvault-secrets
   ```

2. **Authentification avec le Service Principal** :

   Utilisez la classe `ClientSecretCredential` du module `azure-identity` pour vous authentifier à Azure avec les informations du Service Principal.

   Exemple de code Python pour se connecter à un **Key Vault** et récupérer un secret :

   ```python
   from azure.identity import ClientSecretCredential
   from azure.keyvault.secrets import SecretClient

   # Informations de votre Service Principal
   tenant_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  # Identifiant de votre tenant
   client_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"  # Identifiant de votre app (Service Principal)
   client_secret = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  # Clé secrète du Service Principal

   # Identifiant de votre Key Vault
   key_vault_url = "https://<your-keyvault-name>.vault.azure.net/"

   # Se connecter à Azure avec le Service Principal
   credential = ClientSecretCredential(tenant_id, client_id, client_secret)
   
   # Créer un client pour accéder au Key Vault
   client = SecretClient(vault_url=key_vault_url, credential=credential)

   # Récupérer un secret
   secret_name = "TestSecret"
   secret = client.get_secret(secret_name)

   print(f"Le secret est : {secret.value}")
   ```

   Dans ce code :
   - Le `ClientSecretCredential` vous permet de vous authentifier avec les informations du Service Principal.
   - `SecretClient` est utilisé pour interagir avec Azure Key Vault.
   - Vous récupérez un secret stocké dans le Key Vault avec `get_secret()`.

#### Résultat attendu :
- Vous devriez obtenir le secret stocké dans Azure Key Vault.

---

### **Scénario 2 : Utiliser une Identité Managée avec Python**

Dans ce scénario, vous allez utiliser une **Identité Managée** (MSI) attachée à une ressource Azure (comme une machine virtuelle) pour accéder à des ressources Azure, telles qu'un **Key Vault**, sans gérer de secrets.

#### Prérequis :
- Vous devez avoir une **machine virtuelle** ou un **App Service** avec une **Identité Managée** activée.

#### Étapes :

1. **Installer la bibliothèque Azure SDK pour Python** :

   Si vous ne l'avez pas déjà fait, installez les bibliothèques nécessaires :
   ```bash
   pip install azure-identity azure-keyvault-secrets
   ```

2. **Authentification avec une Identité Managée (MSI)** :

   Utilisez la classe `ManagedIdentityCredential` pour obtenir un jeton d'authentification à partir de l'Identité Managée.

   Exemple de code Python pour se connecter à un **Key Vault** et récupérer un secret avec l'Identité Managée :

   ```python
   from azure.identity import ManagedIdentityCredential
   from azure.keyvault.secrets import SecretClient

   # URL de votre Key Vault
   key_vault_url = "https://<your-keyvault-name>.vault.azure.net/"

   # Se connecter à Azure avec l'Identité Managée
   credential = ManagedIdentityCredential()

   # Créer un client pour accéder au Key Vault
   client = SecretClient(vault_url=key_vault_url, credential=credential)

   # Récupérer un secret
   secret_name = "TestSecret"
   secret = client.get_secret(secret_name)

   print(f"Le secret est : {secret.value}")
   ```

   Dans ce code :
   - `ManagedIdentityCredential` gère l'authentification en utilisant l'Identité Managée de la machine virtuelle ou de l'application.
   - Vous n'avez pas besoin de gérer de secrets dans votre code, car l'authentification est effectuée automatiquement via Azure.
   - `SecretClient` vous permet d'interagir avec Azure Key Vault.

#### Résultat attendu :
- Vous devriez obtenir le secret stocké dans Azure Key Vault, tout en utilisant l'Identité Managée pour l'authentification.

---

### **Comparaison et choix entre SPN et Identité Managée :**

- **Service Principal** (SPN) :
  - Utilisé lorsque vous devez gérer des secrets ou lorsque votre application ou service se trouve en dehors d'Azure (par exemple, sur des machines locales ou des serveurs dans d'autres clouds).
  - Nécessite la gestion des secrets dans votre code ou dans un autre service sécurisé (Azure Key Vault).
  
- **Identité Managée** (MSI) :
  - Utilisé principalement pour les ressources Azure telles que les machines virtuelles, les App Services ou les fonctions Azure.
  - Aucune gestion de secrets nécessaire dans le code, car Azure gère automatiquement l'authentification.

### **Avantages de chaque méthode** :
- **Service Principal** : Idéal pour les scénarios où vous avez un contrôle total sur l'authentification et les permissions.
- **Identité Managée** : Plus simple et plus sécurisée pour les services Azure car elle évite de gérer des clés secrètes manuellement.

