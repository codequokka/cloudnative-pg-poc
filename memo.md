```console
❯ export KUBECONFIG=$(mktemp)

❯ kind get kubeconfig --name cloudnative-pg-poc > "$KUBECONFIG"
```

```console
❯ docker pull ghcr.io/cloudnative-pg/postgresql:17.5
17.5: Pulling from cloudnative-pg/postgresql
<omit>
Digest: sha256:2ec17b4c713d5293cb869f8d7b8aca2a9096ad6a775a77b14c35434e598ac210
Status: Downloaded newer image for ghcr.io/cloudnative-pg/postgresql:17.5
ghcr.io/cloudnative-pg/postgresql:17.5

❯ docker images ghcr.io/cloudnative-pg/postgresql --digests
REPOSITORY                          TAG       DIGEST                                                                    IMAGE ID       CREATED      SIZE
ghcr.io/cloudnative-pg/postgresql   17.5      sha256:2ec17b4c713d5293cb869f8d7b8aca2a9096ad6a775a77b14c35434e598ac210   334d8b1aedb0   2 days ago   608MB
```
