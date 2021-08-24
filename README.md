# YOU MUST FOLLOW THE MAINTENANCE PRINCIPLES

### MAINTENANCE DESCRIPTION
We are replacing the RSI:Apps registry certificate.

### HOW ARE WE SOLVING THE PROBLEM
Updating the route for the registry to use the new certificate.

### WHY THIS SOLUTION
Replacing SSL certificates is a standard operations procedure


### ESCALATION PATH
- Who: SRE Team
- What: RSI:Apps Replace Router SSL
- Technical Escalation: Joel Nelson, Edward Rodriguez
- Managerial Escalation: Brian Walsh
- Vendor Escalation: N/A  (no vendor)
- Dependency Escalation: Open Source Community (We're on our own if this goes south)

### PRE-MAINTENANCE CHECKLIST
#### Prior to the start of this maintenance the following groups will be contacted
* rsi_stakeholders@rackspace.com
* tes_sre@rackspace.com

#### Comms

RSI:Apps Stakeholders,

The RSI:Apps ORD cluster will undergo a maintenance to perform Registry Certificate renewal process. This is a standard operations procedure and will be performed on a ORD registry route.

CHG: CHG0128884
Date: 2021-08-26
Time: 09:00:00 CT

Impact is expected to be minimal:

* During route update process, ORD Registry might be unavailable for couple of seconds. Due to this there will be issues pulling the docker image from registry.

Impact Mitigation: Direct application traffic to the other cluster at https://rsi.rackspace.net

Please reach out to teams channel https://rax.io/rare-mst-apps if you have any questions or concerns.

#### Dependencies/Requirements (external applications, other Change Requests, etc...)
* `docker` installed on the toolbox node
* Pre-staged patch files on the ORD toolbox to install both the new and old certs
  
  * `/certs/registry.ord.rsi.rackspace.net.patch.yml`
  * `/certs/oldcerts/registry.ord.rsi.rackspace.net.patch.yml`


#### Test Plan and Results
Successfully tested on our staging RSI:Apps cluster in LON


### MAINTENANCE PREP

#### Perform Pre-Maintenance Audit
1. Pre-check device(s) and equipment involved in the Maintenance.
1. Ensure dependencies and requirements are met
1. Establish communication channels (DC, NOC, Vendors, etc.)
1. Enable Alert Suppression: https://get-zabbix.corp.rackspace.net/maintenance.php?ddreset=1

#### Prepare for the Maintenance
##### Setup the Workspace
**All work should be performed on the toolbox server**

1. Setup the workspace

       screen
       export CHG=<CHG>
       export WORKDIR=~/${CHG}
       mkdir $CHG && cd $CHG

##### Create Backup of Current Route and Cert

1. Back up the current registry route and certs

       oc login rsi.rackspace.net
       oc project default
       oc get route docker-registry-new -o yaml --export | tee ord-registry-route-backup.yml

##### Verify expected configuration

1. Verify the current cert CA chain

 * https://ssltool.rackspace.com/remote/registry.ord.rsi.rackspace.net/443
 * There should only be 2 certificates, the host cert and the issuing cert

2. Verify that the registry is working
 * Login and pull an image

       sudo docker images
       sudo docker login -p $(oc whoami -t) -u unused registry.ord.rsi.rackspace.net
       sudo docker pull registry.ord.rsi.rackspace.net/openshift/nginx
       sudo docker images

 * Delete the image and logout

       sudo docker rmi $(sudo docker images | grep nginx | awk '{print $3}')
       sudo docker images
       sudo docker logout registry.ord.rsi.rackspace.net

##### Stage the patch files

1. Copy the prepared patch files from the /certs directory on the toolbox node

       cp /certs/registry.ord.rsi.rackspace.net.patch.yml install-patch.yml
       cp /certs/oldcerts/registry.ord.rsi.rackspace.net.patch.yml backout-patch.yml

#### Execute Maintenance
1. Patch the route to use the new Cert and CA

       oc apply -f install-patch.yml 2>&1 | tee install-patch.log

1. Verify the new cert CA chain is in use

 * https://ssltool.rackspace.com/remote/registry.ord.rsi.rackspace.net/443
 * There should only be 3 certificates, the host cert, the issuing cert, and the root cert

1. Verify that the new route is working
 * Login and pull an image

       sudo docker images
       sudo docker login -p $(oc whoami -t) -u unused registry.ord.rsi.rackspace.net
       sudo docker pull registry.ord.rsi.rackspace.net/openshift/nginx
       sudo docker images

 * Delete the image and logout

       sudo docker rmi $(sudo docker images | grep nginx | awk '{print $3}')
       sudo docker images
       sudo docker logout registry.ord.rsi.rackspace.net

#### Rollback Plan
1. Patch the route to use the old Cert and CA

       oc apply -f backout-patch.yml 2>&1 | tee backout-patch.log

1. Verify the old cert is in use

 * https://ssltool.rackspace.com/remote/registry.ord.rsi.rackspace.net/443
 * There should only be 2 certificates, the host cert and the issuing cer   

1. Verify that the old route is working
 * Login and pull an image

       sudo docker images
       sudo docker login -p $(oc whoami -t) -u unused registry.ord.rsi.rackspace.net
       sudo docker pull registry.ord.rsi.rackspace.net/openshift/nginx
       sudo docker images

 * Delete the image and logout

       sudo docker rmi $(sudo docker images | grep nginx | awk '{print $3}')
       sudo docker images
       sudo docker logout registry.ord.rsi.rackspace.net

#### Validation
1. Verify the new cert is in use

 * https://ssltool.rackspace.com/remote/registry.ord.rsi.rackspace.net/443
 * Expiration Date should be 2022-08-25

1. Verify that the new route is working
 * Login and pull an image

       sudo docker images
       sudo docker login -p $(oc whoami -t) -u unused registry.ord.rsi.rackspace.net
       sudo docker pull registry.ord.rsi.rackspace.net/openshift/nginx
       sudo docker images

 * Delete the image and logout

       sudo docker rmi $(sudo docker images | grep nginx | awk '{print $3}')
       sudo docker images
       sudo docker logout registry.ord.rsi.rackspace.net
