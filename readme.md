# CloudFaster Academy: Laboratório de resliência com EC2 + Load Balance + Auto Scaling

> **Autor:** [CloudFaster Tecnologia](https://cloudfaster.com.br), **Última revisão:** 31/10/2022

## Introdução

Neste laboratório iremos provisionar um ambiente resiliente e escalável, vamos subir uma aplicação em uma Amazon EC2, configurar um balanceador de carga e o serviço de Auto Scaling.
Para este cenário estaremos utilizando os seguintes serviços da AWS:

- Amazon EC2 (Amazon Elastic Compute Cloud):
    - Serviço de de capacidade computacional da AWS.

- ELB (Elastic Load Balancing):
    - Serviço de balanceadro de cargas da AWS.
    - Distribui o tráfego de rede para melhorar a escalabilidade da aplicação.

- AWS Auto Scaling:
    - Serviço que monitora os aplicativos e ajusta automaticamente a capacidade.
    - Serviço de escalabildiade da AWS.


![ARQ](./assets/ARQ-ECS-HA-SCALING.png)

## Pré-requisitos

1) Uma conta na AWS.
2) Um usuário com permissões suficientes para acessar os recursos necessários (EC2, VPC, ELB, Auto Scaling).

## Hands-on

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
- Nome: `TARGET_LAB_EC2`
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

No serviço de EC2, vamos selecionar a *Instância* que criamos:
- Clicar em *Açoes*
    - *Imagem e modelos*
        - *Criar Imagem* 

![EC2_33](./assets/tela_33.png)
<br /> <br />
<br /> <br />
Vamos colocar um Nome e uma descrição na nossa AMI (`AMI_EC2_LAB`
)
O restante das configurações pode deixar de forma padrão (não precisa modificar)


<br /> <br />
<br /> <br />

![EC2_34](./assets/tela_34.png)

Acesse o menu de AMI no serviço de EC2 e guarde o valor do ID da AMI gerada
`ami-08669891ee3aa5427`

> **Importante:** Observe tambem a coluna *Status* da AMI, a mesma deve estar como `Disponível`, caso esteja sendo provisionada, aguarde até a conclusão (este processo pode levar alguns minutos).

![EC2_37](./assets/tela_37.png)

## Passo 7: Terminar a EC2 criada

Após a conclusão da criação da AMI e a mesma estiver como `Disponivel`, vamos iniciar o processo de encerramento da EC2.

Clique com o botão direto no mouse em cima do nome da *instância* e seleciona a opção *Encerrar Intância*

![EC2_35](./assets/tela_35.png)

Ao abrir o popup de alerta, clique em *Encerrar*

> **Importante:** Após encerrar a *Instância* o acesso a aplicação nao estará mais funcional

![EC2_36](./assets/tela_36.png)

## Passo 7: Criar um Launch template

No Painel da EC2, vamos entrar em *Modelos de Execução*

Clicar em *Criar modelo de execução*

![EC2_38](./assets/tela_38.png)

Colocar um *Nome* e uma *Descrição* para nosso template `TEMPLATE_LAB_EC2`

- Nome: `TEMPLATE_LAB_EC2`


![EC2_39](./assets/tela_39.png)

Selecionar a *AMI* que criamos

![EC2_40](./assets/tela_40.png)

Colocar o *Tipo de instância* como `t2.micro` 

![EC2_41](./assets/tela_41.png)

Selecionar o *par de chaves* que criamos para a nossa *Instância*

![EC2_43](./assets/tela_43.png)

Vamos selecionar uma *subnet* (Na mesa Zona de Disponibilidade do nosso *Elastic Load Balancer*) e apontar o *Security Group* que criamos

![EC2_42](./assets/tela_42.png)


Podemos ver agora nosso *modelo de execução* criado.

![EC2_44](./assets/tela_44.png)

## Passo 8: Criar o Auto Scaling

No painel da EC2, vamos navegar no item *Auto Scaling* e *Grupos de Auto Scaling*

Clicar em *Criar grupo do Auto Scaling*

![EC2_45](./assets/tela_45.png)

Colocar um *Nome* (`AS_LAB_EC2`) no nosso *Grupo do Auto Scaling*

Escolher o *Modelo de execução* (`TEMPLATE_LAB_EC2`) que criamos anteriormente

Clicar em *Próximo*.

![EC2_46](./assets/tela_46.png)

Selecionar a *VPC* e a *Zona de Disponibilidade* que criou o template(`us-east-1b`).

Clicar em *Próximo*.

![EC2_47](./assets/tela_47.png)

Vamos anexar agora nosso *Load Balancer* e escolher o *Grupo de Destino* que criamos pra ele.

Clicar em *Próximo*.


![EC2_48](./assets/tela_48.png)

Configurar a capacidade do nosso *Grupo de Auto Scaling*
Colocar o valor `2` nas 3 opçoes de capacidade

Clicar em *Próximo*.

![EC2_49](./assets/tela_49.png)

Na etapa de *Adicionar noficações*: Clicar em *Próximo*.

Na etapa de *Adicionar etiquetas*: Clicar em *Próximo*.

Clicar em *Criar grupo de Auto Scaling*

## Passo 9: Verificar funcionamento do Auto Scaling

Podemos ver que o *Auto Scaling* subiu duas *instâncias EC2* e a *Verificação de status* foi concluida.

![EC2_50](./assets/tela_50.png)

Podemos acessar o link do *Elastic Load Balancer* que criamos e validar o acesso

![EC2_32](./assets/tela_32.png)

Vamos agora deletar uma *instância* e validar se o *Auto Scaling* vai subir uma nova *EC2*.

Selecionar a *instância*, clique com o botão direito do mouse e clique em *Encerrar instância*.

![EC2_51](./assets/tela_51.png)

Podemos ver após terminar uma *instância* o Auto Scaling iniciou o provisionamento de outra.

![EC2_52](./assets/tela_52.png)

Assim que a *instância* estiver em execução, ja teremos as duas funcionando normalmente com nosso *Load Balancer*.

![EC2_53](./assets/tela_53.png)

Finalizando o LAB temos uma aplicação escalavél na AWS utilizando o serviço de EC2 + ELB + Auto Scaling

That's all folks!