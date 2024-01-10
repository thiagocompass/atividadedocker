# Docker Activity - PB Compass Oul

**Activity description**:

1. instalação e configuração do DOCKER ou CONTAINERD no host EC2;
    - Ponto adicional para o trabalho utilizar a instalação via script de Start Instance
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

**References and used materials**:[Amazon Web Services Documentation](https://docs.aws.amazon.com/pt_br/index.html), [Docker Documentation](https://docs.docker.com/engine/reference/run/), [EFS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html), [RDS(Using MySQL DB)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html), [Elastic Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html), [Tutorial Video](https://www.youtube.com/watch?v=jUf622GXi_E)

---

## Instances Configuration


### Properties

- 

### Using User data (Start Instance Script) to install docker

- Use the following code to automatically install docker in the instance:
  ~~~bash
    #!/bin/bash

    # Update the system
    sudo yum update -y

    # Install Docker
    sudo yum install -y docker

    # Start and enable Docker service
    sudo systemctl start docker
    sudo systemctl enable docker

    # Add the user to the "docker" group
    sudo usermod -aG docker ec2-user

    # Configure Docker to start automatically on boot
    sudo systemctl enable docker

    # Reboot the instance to apply the changes
    sudo reboot
  ~~~

- I had tested this script before and it worked, but unfortunately when I created the instance to make an AMI for the autoscaling group it didn't work, because I didn't wait the correct time for all the script finish the configuration. So I did it manually using the following commands:

    ~~~bash
        sudo yum update -y
        sudo yum install -y yum-utils device-mapper-persistent-data lvm2
        sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        sudo yum install docker-ce docker-ce-cli containerd.io
        sudo systemctl start docker
        sudo systemctl enable docker
        sudo docker --version
        sudo reboot
    ~~~
- So, I get this error:
    ~~~bash
       Status code: 404 for https://download.docker.com/linux/centos/2023.3.20231218/x86_64/stable/repodata/repomd.xml (IP: 13.32.151.28)
        Error: Failed to download metadata for repo 'docker-ce-stable': Cannot download repomd.xml: Cannot download repodata/repomd.xml: All mirrors were tried
        Ignoring repositories: docker-ce-stable
        Last metadata expiration check: 0:00:42 ago on Wed Jan  3 14:43:37 2024.
        No match for argument: docker-ce
        No match for argument: docker-ce-cli
        No match for argument: containerd.io
        Error: Unable to find a match: docker-ce docker-ce-cli containerd.io
    ~~~

- And I solved by doing this:

  ~~~bash
    sudo nano /etc/yum.repos.d/docker-ce.repo

    [docker-ce-stable]
    name=Docker CE Stable - $basearch
    baseurl=https://download.docker.com/linux/centos/7/$basearch/stable
    enabled=1
    gpgcheck=1
    gpgkey=https://download.docker.com/linux/centos/gpg
  ~~~  
### The docker-compose file
    ~~~bash
    version: '3.7'
    services:
      wordpress:
        image:wordpress
        volumes:
          -	/efs/website:/var/www/html
        ports:
          -	"80:80"
        restart: always
        environment:
          WORDPRESS_DB_HOST: db-ami-docker.cjfqd8ykwxpe.us-east-1.rds.amazonaws.com
          WORDPRESS_DB_USER: wordpress    
          WORDPRESS_DB_PASSWORD: Popcap123
          WORDPRESS_DB_NAME: db-ami-docker
          WORDPRESS_TABLE_CONFIG: wp_
    ~~~
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
