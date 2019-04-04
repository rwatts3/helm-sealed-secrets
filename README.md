# helm-sealed-secrets

This plugin is used to generate sealed secrets out of secret. It supports template files with vault paths.
This way, you can store both the template and their rendered representation in git.

### Installation

```bash
helm install https://github.com/actano/helm-sealed-secrets
```

### Usage
```
NAME:
   sealed-secret-template - A new cli application

USAGE:
   sealed-secret-template [global options] command [command options] [arguments...]

COMMANDS:
     enc      encrypt a secret template into a sealed secret
     enc-dir  encrypt all secret templates in a directory structure
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config-file value                          Config file to configure the other flags (default: ".sealed-secrets.yaml")
   --vault.token-file value                     Location of the vault token file (default: "~/.vault-token")
   --vault.address value                        Vault API endpoint [$VAULT_ADDR]
   --sealed-secrets.controller-namespace value  The namespace in which the sealed secrets controller runs
   --help, -h                                   show help
```

### Examples

Read these examples to see how the program works.

#### Single file

Specify a secret template `my-secret.template.yaml`.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret-name
  namespace: dev
type: Opaque
data:
  username: {{ vault "secret/myservice/admin-user" "username" }}
  password: {{ vault "secret/myservice/admin-user" "password" }}
```

Executing

```bash
helm sealed-secrets enc my-secret.template.yaml my-secret.yaml
```

gives you a file `my-secret.yaml`

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: my-secret-name
  namespace: dev
spec:
  encryptedData:
    username: 7tgrVWorKLqoZc...
    password: LbeaMTWxTpWAKD...
```

#### Folders

The names of your secret templates must match the pattern `<name>.template.yaml`.

Given this file structure
```
└── secret-templates
    └── releases
        ├── dev
        │   └── my-secret.template.yaml
        └── prod
            └── my-secret.template.yaml
```

Executing

```bash
helm sealed-secrets --vault.token-file /Users/myuser/.vault-token enc-dir ./secret-templates ./secret-sealed
```

will create the folder structure below `./secret-sealed` and write the sealed secrets in the corresponding folders as `<name>.sealed.yaml`.

```
└── secret-sealed
    └── releases
        ├── dev
        │   └── my-secret.sealed.yaml
        └── prod
            └── my-secret.sealed.yaml
```
