# Deploying Kimai with Docker Compose and Ansible

step-by-step instructions to deploy **Kimai** (a time-tracking application) using **Docker Compose** and **Ansible** on a remote server.


## 1. Clone the Repository
```bash
git clone https://github.com/HardikBhokare/Kimai-Assignment.git
cd DockerCompose-deployment
```

## 2. Update Inventory File
Modify the `ansible/inventory.ini` file to include your server details:

```ini
[servers]
<add server IP>
```

## 3. Define Deployment Variables
change ansible user define `ansible/group_vars/all.yml` if you are using any other user:
```yaml
ansible_user: "ansible"
```

## 4. Review Docker Compose Configuration
Check `docker-compose.yml` and change the credentials if required

## 5. Run the Ansible Playbook
To deploy Kimai on the remote server, run:
```bash
cd ansible
ansible-playbook -i inventory.ini playbook.yml
```

## 6. Access Kimai
Once deployed, access Kimai in your browser at:
```
http://your_server_ip:8080
```
