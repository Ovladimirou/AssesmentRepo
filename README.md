Requirments:
------------
You are required to create a project for automating the deployment of the iconserver service - https://hub.docker.com/r/matthiasluedtke/iconserver
The configuration of the containers should be done using docker-compose, and the server's autoconfiguration should be handled with Ansible.
The service must respond on HTTPS port 443, using self-signed certificates for this purpose.
The project should work at least on CentOS and Ubuntu. The better the compatibility with other distributions, the better.
You need to prepare the full project code along with a README file containing detailed installation instructions.
The project must be published on GitHub or another cloud-based code repository.

Pre-requisities used by me to create the assesment/project:
-----------------------------------------------------------
- Created ec2 instance (Amazon Linux - Fedora), configured a keypair and security group to acees the ec2
- Created a public repo on my personal github account 
- Installed on the ec2 machine the below:
   - docker
   - docker-compose
   - git
   - python 3.8
   - ansible
- Generated ssh key in order to add the .pub key to github so it can pull/push and make changes throught terminal
- Created ansible.cfg (code below)
```
[defaults]
roles_path = roles
log_path = /var/log/ansible.log
collections_paths = collections

```
- Updated .gitignore beacuse it had "/*" that was ignoring all files and directories to be pushed to repository

Installation instructions:
--------------------------
- Clone the repository using "git clone"
- cd to ansible folder
- Run the below command without '#'
#ansible-playbook playbook/playbook.yml -v
- Run "docker ps" after playbok is finished to verify that container is running

!!WARNING!! 
- You need at least git and ansible installed on the machine
- You need to change the absolute paths of the certificate and key (due to permission errors it required absolute paths) in the 'roles/iconserver/tasks/deploy_iconserver.yml' (you will see a "#" comment where you need to change it)
- Of course I did not uploaded certs folder with the cert and private key, however they will be generated once you run the command. folder of certs will be created in the repository
- In order to copy certificates it required to change the owner and group. You will need to modify the roles/certificates/tasks/create_certificate_keys.yml. See '#' comments

Explanation of deployment/playbook:
-----------------------------------
First of playbook starts with server configuration; it checks what OS it is and depending on the os it install docker and docker-compose and starts docker.
Then it created the cert and private key in the certs/ folder and then it deploys locally the iconserver using docker-compose.yml template. 

Repository Structure:
---------------------
|AssessmentRepo
|--ansible/
|  --ansible.cfg
|  --playbook/
|    --playbook.yml
|  --roles/
|    --certificates/
|      --tasks/
|        --create_certificate_keys.yml 
|    --docker/
|      --tasks/
|        --docker_configuration.yml
|      --vars/
|        --main.yml
|    --iconserver/
|      --tasks/
|        --deploy_iconserver.yml.yml
|  --templates/
|    --docker-compose.yml
|--certs/
| --certificate.crt
| --PrivateKey.key

Supported OS:
-------------
Ubuntu and redhat OS

Improvements:
---------------
- If you want to run the ansible playbook for a specific instance, you need to create an inventory, specify specific servers inside and then modify the hosts in the main playbook with variable (See below)
# - hosts: "{{ server }}"
Then run the below command with the server variable
#ansible-playbook playbook/playbook.yml -v -i inventories/hosts -e "server=<ServerIp/Name>"

- Another improvement you can do/idea, if you want specific docker tag of the service you can create a vars/main.yml in the iconserver folder and add below line
docker_tag_version: "latest"

With this, you can set specific version of the service to be deployed. Additionally you will need to modify the docker-compose.yml and rename it to docker-compose.yml.j2
```
    image: "matthiasluedtke/iconserver:{{ docker_tag_version }}"
```
and also modify the task name "Copy Docker Compose template file" in the roles/iconserver/tasks/deploy_iconserver.yml to the below code
```
- name: Copy Docker Compose template file
  copy:
    src: ../../../templates/docker-compose.yml.j2
    dest: /opt/iconserver/docker-compose.yml
```
