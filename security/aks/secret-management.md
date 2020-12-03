# Secret Management with Azure Kubernetes Service

Problem: Secure way to store/access Azure Key Vault secrets with Azure Kubernetes Service

Desired Goals:

- Avoid a direct application code dependency on Azure Key Vault for getting secrets, and adhere to the [12 Factor App principle for configuration](https://12factor.net/config)

General Notes:

- None of the tools below are officially supported by Azure technical support
- None of the tools below notify pods of Secret resource changes that may occur if you are syncing with Azure Key Vault. On Key Vault secret updates/changes pods will have to be re-created or you can use something like the [Wave controller](https://github.com/pusher/wave) to get the changes.
- Be aware of the [risks documented with using Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#risks). If any of these are a concern, consider using the [akv2k8s Env Injector](https://akv2k8s.io/how-it-works/#the-env-injector) or [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- Base64 encoded Key Vault values in Kubernetes Secrets should be considered the same as plain text
- If using Kubernetes Secrets use RBAC to forbid low-privilege users from reading the Secrets

## Tools

### [CSI Driver](https://github.com/Azure/secrets-store-csi-driver-provider-azure)

- Uses [Kubernetes Secrets Store CSI Driver](https://github.com/kubernetes-sigs/secrets-store-csi-driver) which allows Kubernetes to mount multiple secrets, keys, and certs stored in enterprise-grade external secrets stores into pods as a volume.
- Offers [four modes for accessing a Key Vault instance](https://github.com/Azure/secrets-store-csi-driver-provider-azure#provide-identity-to-access-key-vault)
- Mounted Key Vault secrets can be mirrored as Kubernetes Secrets and then additionally exposed as environment variables to the pods

### [Azure Key Vault to Kubernetes](https://github.com/SparebankenVest/azure-key-vault-to-kubernetes)

- [Authenticates](https://akv2k8s.io/security/authentication/) with the default AKS credentials on the cluster (service principal or managed identity)
- Two ways to use:
  - akv2k8s Controller:
    - Stores secrets, certificates and keys from Azure Key Vault to native Kubernetes Secrets.
    - Use if you need native Kubernetes Secrets (e.g. using a 3rd party helm chart which expects a Secret)
    - Periodically polls Azure Key Vault for changes to apply to the Kubernetes Secret
  - akv2k8s Env Injector
    - Securely and transparently injects Azure Key Vault secrets as environment variables into applications, without having to use native Kubernetes Secrets
    - Use if the application running in the container supports getting secrets as environment variables
    - Secret environment variable values are not revealed to Kubernetes resources like Pod specs, stored on disks, visible in logs or exposed in any way other than in-memory for the application

### [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) (not recommended)

- Stores encrypted secrets as a `SealedSecret` which is safe to store - even to a public repository.
- The SealedSecret can be decrypted only by the controller running in the target cluster and nobody else (not even the original author) is able to obtain the original Secret from the SealedSecret.
- SealedSecrets can be given different [scopes](https://github.com/bitnami-labs/sealed-secrets#scopes) depending on the security reuirements of the cluster
- Sealed Secrets can be updated manually
- **Does not interface directly with Azure Key Vault.**
