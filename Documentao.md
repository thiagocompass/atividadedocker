# Linux Activity - PB Compass Oul

**Activity instructions**:

- Configurar o NFS entregue;
- Criar um diretorio dentro do filesystem do
NFS com seu nome;
- Subir um apache no servidor, o apache deve
estar online e rodando;
- Criar um script que valide se o serviço esta
online e envie o resultado da validação para
o seu diretorio no nfs;
- O script deve conter, Data HORA + nome
do serviço + Status + mensagem
personalizada de ONLINE ou offline;
- O script deve gerar 2 arquivos de saida: 1
para o serviço online e 1 para o serviço
OFFLINE;
- Preparar a execução automatizada do script a
cada 5 minutos.
- Fazer o versionamento da atividade;
- Fazer a documentação explicando o processo
de instalação do Linux.

**References and used materials**: [Similar Git Project](https://github.com/alexlsilva7/atividade_aws_linux/blob/main/README.md), [Amazon Web Services Documentation](https://docs.aws.amazon.com/pt_br/index.html), [Amazon Linux 2 Documentation](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/amazon-linux-2-virtual-machine.html), [NFS Documentation](http://l.github.io/debian-handbook/html/pt-BR/sect.nfs-file-server.html).
---

## Linux Configuration Steps


### Setup the NFS (only the server side)

- Install the firewall package `sudo yum install firewall`;
- To keep the NFS service working through the firewall `firewall-cmd —add-service-nfs —permanent` and `firewall-cmd —reload`;
- Start the NFS service `sudo systemctl start nfs-server`;
- Enable the NFS service `sudo systemctl enable nfs-server`;
- Create a new directory to NFS, ex. `sudo mkdir /srv/yourname`;
- Give permissions to nfsnobody user write the in /srv/yourname, `chown nfsnobody:nfsnobody share/`;
- Set up the exports file, access `nano /etc/exports`. Inside write `/srv/share 0.0.0.0/0(rw:all_squash)`, the IP is the client instance. To finish, export the file `exportfs -rva`;

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
   `#!/bin/bash`

`# Script to verify the Apache's state and save this data`
if systemctl is-active --quiet httpd; then
    STATUS="Online"
    MESSAGE="The Apache service is working!"
    OUTPUT_FILE="apache_online_status.txt"
else
    STATUS="Offline"
    MESSAGE="The Apache service isn't working :/"
    OUTPUT_FILE="apache_offline_status.txt"
fi

`# Take the curret date and hour`
CURRENT_DATE=$(date +"%d/%m/%Y %H:%M:%S")

# To salve the data
echo "Date and hour: $CURRENT_DATE" > "/srv/thiago/$OUTPUT_FILE"
echo "Service name: Apache" >> "/srv/thiago/$OUTPUT_FILE"
echo "Status: $MESSAGE" >> "/srv/thiago/$OUTPUT_FILE"

- Salve the file using CONTROL + O
- To turn the file an executable file, write `chmod +x script.sh` 
- To run the script`./script.sh`. 
