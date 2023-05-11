# Collection: trilio.trilio_kubernetes
Set of Ansible Roles and Playbooks for Trilio Cloud-Native Intelligent Recovery products

# Examples

``` bash
ansible-playbook -e @secrets.enc -e @auth.enc --vault-ask-pass tvk-utility.yaml
```

```bash
ansible-playbook -e @secrets.enc -e @auth.enc --vault-ask-pass tvk-utility.yaml --tags "auth"
ansible-playbook -e @secrets.enc -e @auth.enc --vault-ask-pass tvk-utility.yaml --tags "check"
ansible-playbook -e @secrets.enc -e @auth.enc --vault-ask-pass tvk-utility.yaml --tags "backup"
ansible-playbook -e @secrets.enc -e @auth.enc --vault-ask-pass tvk-utility.yaml --tags "smoketest"
```

# Configuration
Configuration is done in *tvk-config.yaml* to allow for automation.<br>

``` yaml
# Specify environment/auth details - username/pass (in a vault specified on cli)
# Username/Pass
# Note: 'openshift' assumes kubectl and oc tools installed
kube_distro: openshift # kubernetes | openshift
kube_auth_type: password # password | kubeconfig
# kube_auth_api: https://auth_endpoint:6443/ # auth API server if not using kubeconfig
                                             # Recommend to store in auth.enc with credentials and
                                             # encrypt and pass as -e @auth.enc
# Kubeconfig
kubeconfig: # path to kubeconfig file if kube_auth_type is 'kubeconfig'

# Always authenticate unless you have a reason to skip this
tvk_auth: true
# Performs a check of the license and other prequisites
tvk_check: true
# Uses the values in the config file to create a demo app, target, backuppplan, backup and restore.
# Set this value for first installation test
tvk_smoketest: true

# Search for suggested NS to backup
tvk_suggest: true
# Auto create NS Backup Plan 
tvk_autoprotect: true
# List of NS to exclude
tvk_autoprotect_exclude: ['openshift-storage','openshift-operators','default']

# app yaml to deploy
tvk_deploy_demo_app: false
tvk_namespace: smoketest
tvk_demo_app_yaml: files/demo-mysql-app.yaml

#
# Backup Target
#
# Currently assumes AWS S3
# IMPORTANT: See templates/target.yaml.j2 to modify for other targets
# TODO: Specify NFS or S3
tvk_create_target: true
tvk_target_type: S3
tvk_target_name: smoketest-target
tvk_s3_bucket_name: tvk-migration-demo1
tvk_secret_name: s3-access-secret

#
# Backup Plan Details
# Note this must be relevant to the test/demo app
#
tvk_create_backupplan: true
tvk_backupplan_name: smoketest-app-backupplan
tvk_backupplan_type: ns # ns|label
tvk_backup_match_label: "app: k8s-demo-app" # ignored for ns backup type

#
# Perform Backup
tvk_backup_prefix: "{{ tvk_backupplan_name }}"
tvk_backup_type: Full # Full | Incremental

# Perform Restore (to new ns)
tvk_restore_namespace: smoketest-restore

# Used for migration/DR smoketest
tvk_dr_restored_namespace: dr-restore
tvk_dr_backupplan: "{{ backupplan_name }}" # if specified, uses last known backup
tvk_location_id: # can specify which backup to restore to if known
```

# Secrets 
To create S3 based Trilio for Kubernetes Targets, a secret is used. This Play can create secrets from an encrypted vault with the following structure:

``` yaml
secrets:
  - name: s3-access-secret
    type: generic
    data:
      - key: accessKey
        value: YOURACCESSKEY
      - key: secretKey
        value: YOURSECRETKEY
```
Create this in a file called *secrets.enc* then encrypt with<br>
``` bash
ansible-vault encrypt secrets.enc
```
You would then specify this on the command line with<br>
``` bash
ansible-playbook -e @secrets.enc --vault-ask-pass tvk-utility.yaml
```

# Authentication
If using username/password authentication and not using kubeconfig specified in tvk-config.yaml<br>
Then create a seperate file, *auth.enc*, and encrypt with the following structure:<br>
``` yaml
k8s_auth_api: https://auth_endpoint_url:6443
k8s_username: username
k8s_password: password
```
Encrypt this file with the following:
``` bash
ansible-vault encrypt auth.enc
```
You would then specify this on the command line with<br>
``` bash
ansible-playbook -e @auth.enc -e @secrets.enc --vault-ask-pass tvk-utility.yaml
```
<br>
