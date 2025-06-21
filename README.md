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
    # evacuation delay
    delay: 0
~~~

Each test should create a results file under /home/zuul/iha-tests-results/ named after the respective playbook, for example "01_disabled.xml".

~~~
{% if success %}
    <testcase classname="{{ name }}" name="{{ name }}">
    </testcase>
{% else %}
    <testcase classname="{{ name }}" name="{{ name }}">
        <failure type="failure"> tests failed </failure>
    </testcase>
{% endif %}
~~~

Right now we use the following as first task to pre-set the test to have failed:

~~~
---
- name: 01 TEST DISABLED PARAM
  hosts: controller-0
  vars:
    name: "01_disabled"
  tasks:
    - name: 01 Create results file (failure)
      ansible.builtin.template:
        src: templates/iha-tests-results.xml.j2
        dest: /home/zuul/iha-tests-results/{{ name }}.xml
      vars:
        success: false
~~~

And last task instead sets the test to have succeeded if everything went fine:

~~~
    - name: 01 Create results file (success)
      ansible.builtin.template:
        src: templates/iha-tests-results.xml.j2
        dest: /home/zuul/iha-tests-results/{{ name }}.xml
      vars:
        success: true
~~~

The 99_gen_junitxml.yaml playbook will generate a junit.xml file out of them.
