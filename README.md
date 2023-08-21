# Short description.

This project uses Ansible to deploy infrastructure of project on VM (VirtualBox, Cloudserver, Dedicated Server) and web-project on Python (in docker-container).

<image src="graduatework.png">

# Applicable Ansible roles.

* *Docker* - install the docker.
* *Nginx* - install the docker-container with web-server Nginx (as systemd service) and with config file for project host.
* *Jenkins* - install the Jenkins on VM, throw up Pipeline web-project.
* *Prometheus* - install the docker-container with Prometheus (as systemd service), also Node and Blackbox Exporter on VM for monitoring.
* *Grafana* - install the docker-container with Grafana (as systemd service) and connects dashboards.
* *Alertmanager* - install the docker-container with Alertmanager (as systemd service) for sending notifications.

# Preparation before launch.

* Add your public ssh-key on VM.
* In files *group_vars/prod* and *hosts.inv* change ip-addresses of VM to the required ones.
* In file *group_vars/prod* you can change the ip-addresses (and ports) of containers and their names, the name of the site (optional). 

# Starting a project deployment.

```
ansible-playbook myproject.yml -i host.inv --ask-vault-pass
```

To remove project components (not all):
```
ansible-playbook myproject.yml -e flush_all_env=true -i host.inv --ask-vault-pass
```

# Checking the correct operation.

After the playbook Ansible is complete, edit your file *hosts*, by adding lines with the name of the site.
Example,
```
your_ip mzhdanko.by prometheus.mzhdanko.by alertmanager.mzhdanko.by grafana.mzhdanko.by jenkins.mzhdanko.by
```
Open the site *mzhdanko.by* in browser after finishing Pipeline *myapp-mz* in Jenkins.

# Compatibility.
The work of the playbook has been tested on OS:
* Ubuntu 18.04
* Ubuntu 20.04
* Ubuntu 22.04

# Detailed description of roles Ansible.

### Role docker.

Installing docker on a VM.\
Create a docker network for the project. You can change the ip-addressing of the project in the file *group_vars/prod*.

### Role nginx.

Pull the latest *docker-image Nginx*, adding the host config file.\
Uses by Reverse Proxy for sites.\
The working directory with the host configuration file on the VM is */opt/docker/nginx*.\
The container runs as a service *docker-nginx.service*, default container name is *nginx-mz*.\
You can change the site name, container name and its ip in the file *group_vars/prod*.\
*/Etc/hosts* is added to the VM to handle requests locally.

### Role Jenkins.

Installation of *Java* and *Jenkins*.\
Configuration files of Jenkins (plugins, jobs, settings) downloading from a third party resource (encrypted in vault) .\
Default data for accessing the web interface:\
http://jenkins.mzhdanko.by/ \
admin / jenkins

Pipeline myapp runs a Jenkinsfile from a repo project that steps through:
* Checks the repository git@github.com:maksimzhdanko/myproject.git
* Building a docker image
* Upload the image to the dockerhub repository: https://hub.docker.com/repository/docker/drowblr/myapp-mz/
* Stops the old container (if it does not exist, for example, at the first start, it fails, but the pipeline continues to run)
* Launches a container with a web application (according to default settings - on port 5000) available at *http://mzhdanko.by*.
* Sends a curl request to check the site.
* Configured sending notifications to the Telegram channel on success or failed build.

### Role Prometheus.

Pull the latest *docker-образа Prometheus*, adding configuration files.\
The working directory with the host configuration file on the VM is */opt/docker/prometheus*.\
The container runs as a service *docker-prometheus.service*, default container name is - *prometheus-mz*.\
Installing *Node* and *Blackbox* Exporters on VM to collect metrics on the state of the server and the availability of the project.\
Available via web interface: http://prometheus.mzhdanko.by/ \
Customized Alert Rules:
* When the web project (mzhdanko.by) is unavailable for more than 1 minute.
* If the CPU utilization is more than 80% in the last 5 minutes.
* If RAM utilization is more than 90% in the last 3 minutes.
* If the free disk space is less than 10% in the last 3 minutes.
You can change the site name, container name and its ip in the file *group_vars/prod*.

### Role Grafana.
Pull the latest *docker-образа Grafana*, adding configuration files and dashboards.\
The working directory with the host configuration file on the VM is - */opt/docker/grafana*.\
The container runs as a service  *docker-grafana.service*, default container name is  - *grafana-mz*.\
Available via web interface: *http://grafana.mzhdanko.by/* \
Login / password:\
admin / grafana \
Datasource - *prometheus*, uses 2 dashboards: VM status and project availability.\
You can change the site name, container name and its ip in the file *group_vars/prod*.

### Role Alertmanager.
Pull the latest *docker-образа Alertmanager*, adding configuration files.\
The working directory with the host configuration file on the VM is - */opt/docker/alertmanager*.\
The container runs as a service *docker-alertmanager.service*, default container name is - *alertmanager-mz*.\
Available via web interface: http://alertmanager.mzhdanko.by/ \
Notifications configured in Telegram group.\
You can change the site name, container name and its ip in the file *group_vars/prod*.

# Planned improvements.

* Rendering jenkins in docker-container.
* Deleting old docker-images from builds.
* Adding job to launch docker image running a specific version.

# For improvement and "Give ansible-vault-pass".

Write to e-mail: drow35@gmail.com or https://t.me/drowblr .

And may the force be with you.
