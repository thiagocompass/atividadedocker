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

**References and used materials**:[Amazon Web Services Documentation](https://docs.aws.amazon.com/pt_br/index.html), [Docker Documentation](https://docs.docker.com/engine/reference/run/), [EFS Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html), [RDS(Using MySQL DB)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.MySQL.html), [Elastic Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html), [Tutorial Video](https://www.youtube.com/watch?v=jUf622GXi_E), [Docker Compose Instalation](https://www.digitalocean.com/community/tutorials/como-instalar-e-usar-o-docker-compose-no-centos-7-pt)

---

## Instances Configuration


### Instance properties

- t3.small, 8gb

### 1.1 Using User data (Start Instance Script) to install docker

- Use the following code to automatically install docker in the instance:
- Wait 5 minutes before connecting to the instance, it needs to finish all the installation process
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

### 1.2 Install Docker Compose
 - To install the docker compose, use the following commands:
  ~~~bash
    sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ~~~

 - Give the file permission 
  ~~~bash
    sudo chmod +x /usr/local/bin/docker-compose
  ~~~

 - Check if it was installed
  ~~~bash
    docker-compose --version
  ~~~

### 1.3 Create the RDS
- Open the console, create your RDS in the same VPC as the subnet.

- Select the option to link to a compute resource, and select your EC2 instance.

- Set up the DB inicial name and password.

- Don't forget to check if the SGs across the instance and RDS are corretcly setted up.

### 1.4 Create the EFS 
- Open the console, create your EFS.

- Remember to set the correct subnets, your instance subnet needs to be in.

- Check the SGs, with there are some hitch.
  
- Create a EFS directory

- Mount the EFS system in your instance, use the code:
  ~~~bash
    sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport xxx.efs.us-east-1.amazonaws.com:/ efs
  ~~~
### 1.5 Set up the compose file
- Create a new folder inside of EFS directory.
- Create a new file, named docker-compose.yml:
  ~~~bash
  sudo nano docker-compose.yml
  ~~~
- Paste the following code inside of it:
  ~~~bash
  version: '3.1'
  services:
  wordpress:
    image: wordpress
    volumes:
      - /home/ec2-user/efs/wordpress4:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: DB ENDPOINT
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
   ~~~
- Pay attention to details like: the DB endpoint end credentials, the wordpress directory path and the ports.
- Up your container, if all is right it might work, use the code:
  ~~~bash
  docker-compose.yml
  ~~~
  
### This part I didn't achive, due to an error that don't let me to go ahead

#### Configure the Load Balancer and Auto Scaling group
<details>
<summary>Set up the Load Balancer</summary>

### Create the Laod Balancer

- Select the Application Load Balancer
- Set a name, a VPC(it needs to be the same as the instance)
- Create a new target group and select your instance
- If you did all right, try to acess your instance using the load balancer DNS, it might work
</details>
<details>
<summary> Set up the Auto Scaling Group </summary>

### To create the auto scaling group
- Create an AMI of your instance
- Add this AMI to a model for auto scaling group
- Select the wished working method
- Set the load balancer and the target groups
- To finish, choose at minimum 2 instances and two different availability zones
- If you did all right now you may acess the WP index screen by your load balancer DNS.
</details>
