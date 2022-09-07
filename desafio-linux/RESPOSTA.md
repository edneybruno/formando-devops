Desafio #1
Autor: EDNEY BRUNO

1. Kernel e Boot loader

*Como possuo acesso ao host físico, reiniciei o host e na tela do GRUB apertei a tecla 'e' onde encontrar algumas opções do GRUB e na linha que começa com linux adicionei o trecho no final: 'init=/bin/sh' para poder entrar na linha de comando e poder executar comandos para alterar a senha do root, primeiro remontei o diretorio '/' como 'rw' e posteriormente alterei a senha do root com o comando passwd root e reiniciei a máquina.

    `mount -o remount,rw /`
    `passwd root`

*Segue as senhas alteradas.

*Senha do root: 'a senha do root e segura'

*Senha do usuário vagrant: 'formandodevops'

*Assim editei o arquivo sudoers através do comando 'visudo' e editei algumas linhas como os alias e tambem o grupo 'sys' e adicionei o usuário vagrant no grupo 'sys'.

*Podendo assim executar alguns comandos de root.


2. Usuários
2.1. Criação de Usuários

*Com o usuário root logado vamos dar os comandos.

*Criando o grupo getup.

    `groupadd -g 2222 getup`
    
*Criando o usuario getup

    `adduser --uid 1111 --gid 2222 getup`
    
*Definindo a senha e quando o usuario logar irá pedir para ele definir a senha.

    `passwd -e getup`
    
*Adicionando ao grupo 'bin'

    `gpasswd -a getup bin`
    
*Obs: Usei o comando 'adduser' para poder ja ir adicionando o bash, o diretorio home, mas poderia ter usado useradd.

*Editei com o visudo o arquivo sudoers e adicionei a seguinte linha.

    `getup   ALL=(ALL)   NOPASSWD: ALL`


3. SSH
3.1. Autenticação Confiável

*Com o usuário root logado fazemos a alteração no arquivo de configuração do SSH.

*Desabilitando a autenticação por senha e deixando somente o uso de chaves.

    `PasswordAuthentication no`


3.2. Criação de Chaves
*Gerando a chave.

    `ssh-keygen -t ecdsa`
    
*Deixei o nome padrão da chave

*Definir ou não a senha para a chave
    
*Copiando a chave pro host remoto.

    `ssh-copy-id vagrant@192.168.0.116 #IP que foi dado ao host na rede.`
    
    Informa a senha do usuário vagrant.
    
*Conectando no host como usuário vagrant

    `ssh -p [port] vagrant@192.168.0.116 #Porta especificada quando não é a padrão [22].`
    
    Informa a senha da chave ecdsa caso tenha colocado e pronto.
    
*OBS: Deve-se configurar as chaves e enviá-las pro host remoto antes de alterar a config 'PasswordAuthentication' para 'no', pois vc pode não conseguir enviar a chave pub.


3.3. Análise de logs e configurações ssh

*Decodificando com o base64 e jogando o resultado para descompactar com o gunzip e jogando a saída padrão para o arquivo.

    `base64 -d id_rsa-desafio-linux-devel.gz.b64 | gunzip > id_rsa-desafio-linux-devel`
    
*Tentei conectar ao host remoto com a chave que foi obtida anteriormente mas estava dando um erro.

*Primeiro verifiquei qual log teria a informação sobre o erro do usuário devel com o comando:

    `grep -Rl 'devel' * #lista os arquivos que contem o padrão 'devel'`
    
*Olhei todos os arquivos que retornaram e verifique no log secure o seguinte erro:

    `Authentication refused: bad ownership or modes for file /home/devel/.ssh/authorized_keys`
    
*Verifiquei as permissões do arquivo e com a dica vi que o erro tratava de permissões, alterei as permissões e conectei normalmente.

    `chmod 700 ~devel/.ssh/authorized_keys`
    
*Conectando no host remoto.

    `ssh -i id_rsa-desafio-linux-devel devel@192.168.0.116`


4. Systemd
    
*Encontrado erro no arquivo de configuração do NGINX onde faltava um ';' no final da linha.

    `nginx -t        #Verificando a sintaxe do arquivo nginx.conf`
    
*Depois o erro foi no arquivo da Unit do NGINX.

*Editei o arquivo '/lib/systemd/system/nginx.service' e alterei a devida linha:

    `ExecStart=/usr/sbin/nginx -BROKEN`
    
*E coloquei assim no lugar da linha acima:

    `ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'`
    
*Recarreguei o daemon

    `systemctl daemon-reload`
    
*Restartei o serviço

    `systemctl restart nginx`
    
*E rodei o comando

    `curl http://127.0.0.1`


5. SSL
5.1. Criação de Certificados
    
*Com o usuário root logado damos os seguintes comandos.

*Editar o arquivo /etc/hosts

    `echo "192.168.0.116     www.desafio.local"`
    
*Fiz a instalação do modulo mod_ssl

    `yum install mod_ssl`
    
*Diretorio para chaves e certificados do CA

    `cd /etc/ssl/certs`
    
*Gerando primeiro a chave.

    `openssl genrsa -des3 -out desafio.local.key 2048`
    
    -> Informa a senha para a chave

*Gerando o certificado.

    `openssl req -x509 -new -key desafio.local.key -sha256 -days 365 -out desafio.local.pem`
    
    -> Informa a senha da chave criada anteriormente.
    
    -> Responder as informações.
    
    -> Assim temos a chave e o certificado CA.
    

5.2. Uso de certificados

*Diretório para a chave e certificados.

    `cd /etc/pki/nginx/`
    
    `mkdir private       #Diretorio para guardar a chave.`
    
*Criando uma chave privada para o NGINX.

    `openssl genrsa -out ./private/server.key 2048`
    
*Agora criaremos o certificado de requisição 'csr' com a chave criada.

    `openssl req -new -key server.key -out desafio.local.csr`
    
    -> Responder as informações.
    
*Agora vamos assinar o certificado do NGINX.

    `openssl x509 -req -in desafio.local.csr -CA /etc/ssl/certs/desafio.local.pem -CAkey /etc/ssl/certs/desafio.local.key -CAcreateserial -out server.crt -days 365 -sha256`

*Agora podemos solicitar a pagina HTTPS

    `curl https://www.desafio.local --cacert /etc/pki/nginx/server.crt`


6. Rede
6.1. Firewall
    
*Quando dei o comando 'ping 8.8.8.8' já estava funcionando. Não sei se foi erro na hora de criar o desafio.

*Mas caso tenha sido posso supor que o firewall estava bloqueando o protocolo ICMP, bastando então liberá-lo.


6.2. HTTP
*Inclui cabeçalhos de resposta de protocolo na saída

    `curl -i https://httpbin.org/response-headers?hello=world`

LOGS

*Criei arquivo /etc/logrotate.d/nginx e adicionei essa configuração para rotacionar os logs.

    `/var/log/nginx/*log{
        create          #cria o arquivo de log.
        daily           #rotaciona o arquivo diariamente.
        rotate 10       #salva até 10 copias dos logs.
        missingok       #não gera erro se o arquivo de log estiver ausente.
        notifempty      #não rotaciona se o log estiver vazio.
        compress        #compacta as cópias com gzip(padrão)
    }`


7. Filesystem
7.1. Expandir a partição LVM

*Listei os discos.

    `fdisk -l`
    
*Editei o disco e criei outra outra partição do tamanho restante do disco e do tipo Linux LVM.

    `fdisk /dev/sdb`
    
*Atualizei a tabela de particionamento.

    `partprobe`
    
*Adicionei a partição 2 no LVM como disco físico.

    `pvcreate /dev/sdb2`
    
*Extendi o grupo de volume.

    `vgextend data_vg /dev/sdb2`
    
*E depois extendi o volume lógico para o tamanho solicitado.

    `lvextend -L +4G /dev/data_vg/data_lv`
    
*Como o LV estava formatado atualizei o tamanho tambem no filesystem.

    `resize2fs /dev/data_vg/data_lv`


7.2. Criar partição LVM

*Criando um volume lógico data_lv2 a partir do grupo de volume data_vg.

    `lvcreate -L 5G -n data_lv2 data_vg`
    
*Formatando a partição LVM com o filesystem ext4.

    `mkfs.ext4 /dev/data_vg/data_lv2`


7.3. Criar partição XFS

*Fiz a instalação do pacote de ferramentas do xfs.

    `yum install xfsprogs.x86_64`
    
*Formatando o disco inteiro com o filesystem xfs.

    `mkfs.xfs -f /dev/sdc`
