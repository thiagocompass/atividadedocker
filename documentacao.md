# Docker Activity - PB Compass Oul

**Activity description**:

1. instalação e configuração do DOCKER ou CONTAINERD no host EC2;
    -Ponto adicional para o trabalho utilizar a instalação via script de Start Instance
    (user_data.sh);
2. Efetuar Deploy de uma aplicação Wordpress com:
    container de aplicação
    RDS database Mysql;
3. configuração da utilização do serviço EFS AWS para estáticos do container de aplicação Wordpress;
4. configuração do serviço de Load Balancer AWS para a aplicação Wordpress;
- Pontos de atenção:
não utilizar ip público para saída do serviços WP (Evitem publicar o serviço WP via IP Público)
sugestão para o tráfego de internet sair pelo LB (Load Balancer Classic)
pastas públicas e estáticos do wordpress sugestão de utilizar o EFS (Elastic File Sistem)
Fica a critério de cada integrante (ou dupla) usar Dockerfile ou Dockercompose;
Necessário demonstrar a aplicação wordpress funcionando (tela de login)
Aplicação Wordpress precisa estar rodando na porta 80 ou 8080;
Utilizar repositório git para versionamento;
Criar documentação.

**References and used materials**:[Amazon Web Services Documentation](https://docs.aws.amazon.com/pt_br/index.html), [Docker Documentation](https://docs.docker.com/engine/reference/run/), [EFS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html), [RDS(Using MySQL DB)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html), [Elastic Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)

---

## Linux Configuration Steps


### To set up the NFS (only the server side

- Install the firewall package `sudo yum install firewall`;
- To keep the NFS service working through the firewall `firewall-cmd —add-service-nfs —permanent` and `firewall-cmd —reload`;
- Start the NFS service `sudo systemctl start nfs-server`;
- Enable the NFS service `sudo systemctl enable nfs-server`;
- Create a new directory to NFS, ex. `sudo mkdir /srv/yourname`;
- Give permissions to nfsnobody user write in /srv/yourname, `chown nfsnobody:nfsnobody share/`;
- Set up the exports file, access `nano /etc/exports`. Inside, write `/srv/share 0.0.0.0/0(rw:all_squash)`, the IP is from the client instance. To finish, export the file `exportfs -rva`;

### To set up Apache.

- Write the command  `sudo yum update -y` to update the system;
- Install apache `sudo yum install httpd -y`;
- Start apache `sudo systemctl start httpd`;
- Enable apache to start automatically `sudo systemctl enable httpd`;
- To verify the service status `sudo systemctl status httpd`;
- If you want to stop apache use `sudo systemctl stop httpd`.

### Creating the validation script

- Create a new file named `nano script.sh`, you may put it inside of /yourname/.
- The script: 
~~~php   
#!/bin/bash

# Script to verify the Apache's state and save this data
if systemctl is-active --quiet httpd; then
    STATUS="Online"
    MESSAGE="The Apache service is working!"
    OUTPUT_FILE="apache_online_status.txt"
else
    STATUS="Offline"
    MESSAGE="The Apache service isn't working :/"
    OUTPUT_FILE="apache_offline_status.txt"
fi

# Take the curret date and hour
CURRENT_DATE=$(date +"%d/%m/%Y %H:%M:%S")

# To salve the data
echo "Date and hour: $CURRENT_DATE" > "/srv/thiago/$OUTPUT_FILE"
echo "Service name: Apache" >> "/srv/thiago/$OUTPUT_FILE"
echo "Status: $MESSAGE" >> "/srv/thiago/$OUTPUT_FILE"
~~~
- Salve the file using CONTROL + O;
- To turn the file an executable file, write `chmod +x script.sh`; 
- To run the script`./script.sh`. 

### To set up the automatic execution to 5 in 5 minutes.

#### There are two different ways
<details>
<summary>Contrab(more easy)</summary>

### To configure the crontab

- Edit the file `cronjob`.
- Write in crontab:
    ```bash
    */5 * * * * /your/script/path/script.sh
    ```
- Salve the file.
- To verify if it’s working, write `crontab -l`.
</details>
<details>
<summary>By Systemd (more complex)</summary>

### To configure the systemd service.
- Create a new file `sudo nano /etc/systemd/system/validate_apache.service`.
- Add this code in validate_apache.service:
    ```bash
    [Unit]
    Description=Validate apache service
    
    [Service]
    Type=simple
    ExecStart=/home/ec2-user/script.sh
    Restart=on-failure
    RestartSec=5
    
    [Install]
    WantedBy=multi-user.target
    ```
- Save the file;
- Reload systemd, write `sudo systemctl daemon-reload`;
- Start the service `sudo systemctl start validate_apache`;
- Enable it to start automatically  `sudo systemctl enable validate_apache`;
- Verify the service status using `sudo systemctl status validate_apache`.

### Now add the timer to systemd.
- Create a new file `sudo nano /etc/systemd/system/validate_apache.timer`.
- Add this code in validate_apache.timer:
    ```bash
    [Unit]
    Description=Validate apache timer
    
    [Timer]
    OnBootSec=5min
    OnUnitActiveSec=5min
    Unit=validate_apache.service

    [Install]
    WantedBy=multi-user.target
    ```
- Salve the file;
- Reload systemd again `sudo systemctl daemon-reload`;
- To start the timer enter `sudo systemctl start validate_apache.timer`;
- Enable this server to start automatically `sudo systemctl enable validate_apache.timer`;
- To verify the service status, write `sudo systemctl status validate_apache.timer`.

</details>
