Playbooks meant to run via test-operator.

Example AnsibleTest cr:


~~~
---
apiVersion: test.openstack.org/v1beta1
kind: AnsibleTest
metadata:
  name: iha-tests
  namespace: openstack
spec:
  containerImage: quay.io/podified-master-centos9/openstack-ansible-tests:current-podified
  debug: true
  storageClass: "lvms-local-storage"
  workloadSSHKeySecretName: 'test-operator-controller-priv-key'
  ansiblePlaybookPath: playbooks/main.yaml
  ansibleGitRepo: https://github.com/openstack-k8s-operators/iha-tests
  ansibleInventory: |
    localhost ansible_connection=local ansible_python_interpreter=python3
    controller-0 ansible_host=192.168.111.9 ansible_user=zuul ansible_ssh_private_key_file=~/test_keypair.key ansible_host_key_checking=false
    hypervisor ansible_host=192.168.111.1 ansible_user=zuul ansible_ssh_private_key_file=~/test_keypair.key ansible_host_key_checking=false
  ansibleVarFiles: |
    ---
    # example
    somevar: somevalue
~~~
