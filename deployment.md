# deploying cfcr 0.24 on azure

```bash
export BOSH_BOOTLOADER=~/GIT/bosh-bootloader
bbl plan --name azure-cfcr
cp -r $BOSH_BOOTLOADER/plan-patches/cfcr-azure/. .
bbl up --debug
eval "$(bbl print-env)"

git clone https://github.com/cloudfoundry-incubator/kubo-deployment.git
export KD=$(pwd)/kubo-deployment
bosh upload-stemcell https://bosh.io/d/stemcells/bosh-azure-hyperv-ubuntu-xenial-go_agent
```

## create cloud config

```bash
export deployment_name="azurecfcr"
bosh update-config --name ${deployment_name} \
./ops/cfcr-vm-extensions.yml \
--type cloud \
-v deployment_name=${deployment_name} \
-l <(bbl outputs)
```

## deploy cfcr manifest single(single)

```bash
bosh -n -d ${deployment_name} deploy ${KD}/manifests/cfcr.yml \
-o ${KD}/manifests/ops-files/misc/single-master.yml \
-o ${KD}/manifests/ops-files/add-hostname-to-master-certificate.yml \
-o ${KD}/manifests/ops-files/use-runtime-config-bosh-dns.yml \
-o ${KD}/manifests/ops-files/rename.yml \
-o ./ops/use-vm-extensions.yml \
-o ./ops/single-worker.yml \
-o ./ops/use-cfcr-subnet.yml \
-o ./ops/cloud-provider.yml \
-v deployment_name=${deployment_name} \
-v cfcr_location=${BBL_AZURE_REGION} \
-l <(bbl outputs)
```

## or full deployment

```bash
bosh -n -d ${deployment_name} deploy ${KD}/manifests/cfcr.yml \
-o ${KD}/manifests/ops-files/add-hostname-to-master-certificate.yml \
-o ${KD}/manifests/ops-files/use-runtime-config-bosh-dns.yml \
-o ${KD}/manifests/ops-files/rename.yml \
-o ./ops/use-vm-extensions.yml \
-o ./ops/use-cfcr-subnet.yml \
-o ./ops/cloud-provider.yml \
-v deployment_name=${deployment_name} \
-v cfcr_location=${BBL_AZURE_REGION} \
-l <(bbl outputs)
```

```bash
bosh -d ${deployment_name} run-errand apply-specs
```

## get the Director name

```bash
bosh env
```

export DIRECTOR_NAME=<DirectorName>
## export

```bash
export DIRECTOR_NAME=bosh-azure-kubo
${KD}/bin/set_kubeconfig ${DIRECTOR_NAME}/${deployment_name} https://${api_hostname}:8443
```


## clone into a demo, like guestbook

