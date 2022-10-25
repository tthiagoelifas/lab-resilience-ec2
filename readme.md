# CloudFaster Academy: Laboratório de resliência com EC2 + Load Balance + Auto Scaling

> **Autor:** [CloudFaster Tecnologia](https://cloudfaster.com.br), **Última revisão:** 25/10/2022

## Pré-requisitos

1) Uma conta na AWS.
2) Um usuário com permissões suficientes para acessar os recursos necessários (EC2, VPC, ELB, Auto Scaling).

## Passo 1: Subir uma EC2 com script de inicialização para aplicação web

Após acessar sua conta AWS, navegue até o serviço "EC2" ou acesse diretamente por esse link:
https://console.aws.amazon.com/ec2.

Clique em execultar instâncias

![EC2_01](./assets/tela_01.png)

Agora vamos configurar os parâmetros da EC2.

- Nome: LAB_EC2

![EC2_02](./assets/tela_02.png)

- Sistema Operacional: Vamos escolher o Amazon Linux com arquiterua 64bits

![EC2_03](./assets/tela_03.png)

- Tipo de instância: Vamos selecionar a t2.micro a fim de utilizar o free tier da AWS

![EC2_04](./assets/tela_04.png)

- Clicar em Criar nova chave de acesso

![EC2_05](./assets/tela_05.png)

- Par de chaves de acesso:
    - Nome: key_lab_ec2
    - Tipo de par de chaves: Vamos deixar o padrão RSA
    - Formato:
        - .pem (recomendada para quem utiliza sistema operacional Linux ou Mac)
        - .ppk (recomendada para que utiliza sistema operacional Windows com o Putty instalado)

![EC2_06](./assets/tela_06.png)

- Configurações de rede:
    - Criar grupo de segurança:
        - Permitir trafego SSH de:
            - Anywhere (é feita uma liberacao de acesso para qualquer destino)
            - Custom (é possível colocar um endereço de IP de onde deve ser feito o acesso)
            - MyIp (á AWS identifica seu IP de origem e faz a liberação apenas para ele)

![EC2_07](./assets/tela_07.png)

- Configurações avançadas vamos colocar nosso script para que a EC2 execute o mesmo durante sua inicialização.
- Após essa etapa, clique em *"Execultar instância"*, para que a mesma seja provisionada e inicializada


![EC2_08](./assets/tela_08.png)

## Passo 2: Testar o acesso web na aplicação

No painel do serviço de EC2, vamos verificar o estato da nossa instanância:

- Executando (EC2 ativa, provisionada e pronta para utilização)
- Prosionando (EC2 ainda está sendo disponiblizada pela AWS)

Vamos copiar o IP público da nossa instânia e testar em um navegado para validar a aplicação. <br /> <br />

> **Importante:** O IP externo utilizado no LAB é diferente do que vai aparecer na sua Console da AWS.

<br /> <br />
![EC2_09](./assets/tela_09.png)

Observamos que foi retornado um erro de timeout de endereço de IP, isso ocorreu pois para validar a aplicação vamos precisar liberar a porta HTTP (TCP 80) no nosso Security Group.

![EC2_10](./assets/tela_10.png)

Clicando na aba *"Segurança"* vamos clicar no nosso Security Group para editar as regas de acesso.

![EC2_11](./assets/tela_11.png)

Selecionar a aba *"Regras de entrada"* e *"Editar regras de entrada"*.

![EC2_12](./assets/tela_12.png)

Vamos *"Adicionar regras"*, selecionar o protocolo HTTP, Colocar para *"Qualquer origem"* e vamos *"Salvar regras"*.

![EC2_13](./assets/tela_13.png)

Realizando um novo teste de acesso ao IP externo, vamos ter sucesso em acessar nossa aplicação.

![EC2_14](./assets/tela_14.png)

## Passo 3: Criar um ELB (Elastic Load Balance)

Ainda dentro do serviço de [EC2](https://console.aws.amazon.com/ec2), vamos realizar a configuração do Elastic Load Balance.

Vamos clicar em *"Balanceamento de cargas - Load Balancers*

![EC2_15](./assets/tela_15.png)

Clique em *"Criar Load Balancer"*

![EC2_16](./assets/tela_16.png)

Temos três opções de Elastic Load Balancer:
- Application Load Balancer (Balanceador que atua na camada de aplicação nos protocolos HTTP e HTTPS)

- Network Load Balancer (Balanceador que atua na camada de rede fazendo chamadas nos protocolos TCP, UDP e TSL)

- Gateway Load Balancer (Distribuir tráfego entre dispositivos virtuais)

Como queremos balancer uma aplicação web, vamos utilizar o *"Applicaton Load Balance"*.


![EC2_17](./assets/tela_17.png)

Preencher o nome (*ELB-LAB-EC2*) do nosso ELB, deixar ele com interface para internet e utilizar o padrão de IPV4.

![EC2_18](./assets/tela_18.png)

No Painel de EC2, temos a coluna onde temos a *Zona de Disponibilidade* que ela está. no nosso exemploe ela fica em *`us-east-1`*

![EC2_19](./assets/tela_19.png)

Na configuração do Load Balance, vemos que ele mostra todas as *Zonas de Disponilidade* que pertecem a VPC

![EC2_20](./assets/tela_20.png)

Vamos selecionar *Duas*, uma é a que a nossa EC2 está (`us-east-1`) e podemos escolher outra onde futuramente podemos configurar outra EC2.

![EC2_21](./assets/tela_21.png)

Configurar um novo *Security Group* para o nosso Load Balance

![EC2_22](./assets/tela_22.png)

Atribuições:
- *Nome* para o *Security Group* 
- Adicionar o protocolo HTTP para ser acessado que qualquer origem

![EC2_23](./assets/tela_23.png)

![EC2_24](./assets/tela_24.png)

Vamos selecionar o *Security Group* criado para o nosso ELB

![EC2_25](./assets/tela_25.png)

## Passo 4: Registrar a EC2 no ELB (Elistic Load Balance)

Clique em  *Create Target Group*

![EC2_26](./assets/tela_26.png)

Na configuração do *Target Group*, temos alguns tipos de *Target*, pro nosso ALB vamos utilizar o Type `Instância`

![EC2_27](./assets/tela_27.png)

Vamos colocar um *Nome* e a porta que o nosso *Target* vai trabalhar.
- Porta 80 (protocolo HTTP)

![EC2_28](./assets/tela_28.png)

Seleciona a *Instância* que foi criada no inicio do LAB e clique em *Include as pending below*.

![EC2_29](./assets/tela_29.png)

Criando nosso Listener
Vamos selecionar o *Target Group* que acabamos de criar

Clicar quem *Criar Load Balancer*

![EC2_30](./assets/tela_30.png)

## Passo 5: Testar o acesso pelo ELB (Elastic Load Balance)

Navegando na tela da Load Balance vamos copiar o nome de DNS criado para testar o acesso em um navegado

> **Importante:** Para o ELB ficar como *Ativo* pode demorar alguns minutos.

![EC2_31](./assets/tela_31.png)

Cole o link do DNS do Load Balancer e acesse sua aplicação

![EC2_32](./assets/tela_32.png)


## Passo 6: Criar uma AMI e guardar o ID

## Passo 7: Terminar a EC2 criada

