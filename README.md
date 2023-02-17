# actions-docker-image-sync

Action for docker image sync between repositories

## Examples

### Syncing from Google cloud to China

Syncing images to the china registry before attempting production deploys

```
      - name: 🚀 Docker push to ACR
        uses: tradeshift/actions-docker-image-sync@v1
        with:
          image: "tradeshift-base/<component>:${{ env.GIT_HEAD_SHA }}"
          source-repo: eu.gcr.io
          target-repo: tradeshift-registry.eu-central-1.cr.aliyuncs.com
          target_password: ${{ secrets.ACR_PASSWORD }}
          target_user: ${{ secrets.ACR_USR }}
```

The source-repo is assumed to be already authenticated ealier in the job like this

```
      - name: Docker auth
        uses: tradeshift/actions-docker@v1
        with:
          password: ${{ secrets.GCLOUD_SERVICE_ACCOUNT_KEY_NOBASE64 }}
          auth-only: 'true'
```

### Syncing from Google cloud to amazon ECR via AWS roles

We can pre-authenticate with the registries using the auth-only mode, and
skip passing user and password to the sync action. This allows us to auth
via AWS roles, letting us push images to Amazon Elastic container registry.

```yaml
- name: Docker auth (GCR)
  uses: tradeshift/actions-docker@v1
  with:
    password: ${{ secrets.GCLOUD_SERVICE_ACCOUNT_KEY_NOBASE64 }}
    auth-only: "true"
- name: Docker auth (ECR)
  uses: tradeshift/actions-docker@v1
  with:
    repository: 063399264027.dkr.ecr.eu-west-1.amazonaws.com
    auth-only: "true"
- name: 🚀 Docker push to ECR
  uses: tradeshift/actions-docker-image-sync@v1
  with:
    image: "tradeshift-base/<component>:${{ env.GIT_HEAD_SHA }}"
    source-repo: eu.gcr.io
    target-repo: 063399264027.dkr.ecr.eu-west-1.amazonaws.com
```

### Renaming an image while syncing

If the image needs to exist under a different name on the target registry, it
can be renamed by providing `target-image` as well.

```yaml
- name: 🚀 Docker push to ECR (tradeshift-public)
  uses: tradeshift/actions-docker-image-sync@v1
  with:
    image: "tradeshift-base/<component>:${{ env.GIT_HEAD_SHA }}"
    target-image: "tradeshift-public/<component>:${{ env.GIT_HEAD_SHA }}"
    source-repo: eu.gcr.io
    target-repo: 063399264027.dkr.ecr.eu-west-1.amazonaws.com
```
