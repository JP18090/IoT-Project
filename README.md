# IoT-Project


Tecnologias utilizadas 

 node-red 
 mqtt 
 Wokwi ESP32 
InfluxDB 
Grafana 
Telegram para notificações 


fluxo: ESP32 (consulta api do tempo) --> node-red ( dentro de uma EC2 na aws) --> execução do fluxo com a logica do projeto --> Salva informações no InfluxDB --> Grafana --> Dashboard
fluxo: ESP32 (consulta api do tempo) --> node-red ( dentro de uma EC2 na aws) --> execução do fluxo com a logica do projeto --> mandar de volta a informação da abertura/fechamneto da bomba hidrica ao mqtt --> ESP32 (executa)
fluxo: ESP32 (consulta api do tempo) --> node-red ( dentro de uma EC2 na aws) --> execução do fluxo com a logica do projeto --> telegram (manda as informações do status atual par ao cliente via bot)


link dos wokiwi: 
wokwi que conuslta tempo --> https://wokwi.com/projects/463754552975480833 
wokwi que executa abertura e fechamneto da bomba hidrica --> https://wokwi.com/projects/464740954234441729 

Foto do fluxo do node-red 
<img width="1566" height="239" alt="image" src="https://github.com/user-attachments/assets/e0774d41-f0fd-49f5-9ed8-41e9ff9d01c7" />

Passo a passo node-red na aws: 

Criar instância EC2:

- Criar instância do tipo Ubuntu 24.04 LTS (HVM) - esta é a versão que foi testada e funcionou!;

- Usar o tipo de instância t2.medium (2 vCPU e 4 GiB RAM);

- Criar novo par de chaves do tipo .pem;

- Usar PuTTYgen para exportar chave do tipo .pem para chave do tipo .ppk (para que a instância possa ser acessada via PuTTY);

- Permitir tráfego SSH, HTTP e HTTPS;

- Criar instância.

Conectar na instância EC2:

- Abrir AWS CloudShell;

- Subir o arquivo .pem;

- Para alterar as permissões do arquivo da chave privada, executar o comando:

chmod 400 <arquivo.pem>

- Para entrar no console da máquina virtual, executar o comando:

ssh -i "<arquivo.pem>" <nome FQDN ou IP da instância>

Atualizar instância EC2:

- Para atualizar o repositório, executar o comando:

sudo apt update


- Para atualizar os pacotes, executar o comando:

sudo apt upgrade

Instalar o Node.js e o Node--RED:

- Baixando o script de instalação:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash 

- Executando o script:

\. "$HOME/.nvm/nvm.sh"

- Instalando o gerenciador de versões:

nvm install 25


- Instalar dependência

sudo apt install libatomic1

- Instalando o Node-RED:

npm install -g node-red

npm install -g npm@11.12.1

- Executando o Node-RED:

node-red

Resolução de problemas (versão 3)
Node-RED não inicializa após a máquina reiniciar:

- Caso o Node-RED não inicialize automaticamente após a máquina reiniciar, execute a seguinte sequencia de comandos:

npm install -g pm2

pm2 start node-red

pm2 startup

sudo env PATH=$PATH:/home/ubuntu/.nvm/versions/node/v25.8.2/bin /home/ubuntu/.nvm/versions/node/v25.8.2/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu

pm2 save

Redirecionar porta 1880 para porta 80:
- A fim de evitar ter que digitar a porta ao final da URL para acesso ao Node-RED, é possível criar um redirecionamento do tráfego adicionando uma regra no firewall do sistema operacional. Para isso, executar o comando:

sudo iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 1880

Configurar regra de entrada para porta 1880:

- Alternativamente ao redirecionamento da porta 1880 para porta 80, pode-se criar uma regra de entrada (inbound) no grupo de segurança da instância para permitir o tráfego na porta 1880.

Erro de conexão (Lost connection to server):

- O Node-RED utiliza websockets, que é uma forma de conexão bidirecional sobre HTTP e que só funciona quando a conexão é estável. Este mecanismo pode não funcionar adequadamente em redes que tem um proxy de saída. Para contornar este problema, deve-se utilizar uma conexão que contorne o proxy. No nosso caso, o mais conveniente é criar uma máquina virtual Windows no AWS para conectar ao servidor Node-RED, e assim, evitar o proxy da rede da instituição.
