# DevOps4Devs 02

## Aula 01
### O projeto conversão de temperatura se encontra no link abaixo:

[https://github.com/KubeDev/conversao-temperatura](https://github.com/KubeDev/conversao-temperatura)

## Aula 02 - Kubernetes: do zero ao deploy
### Comando de criação do cluster Kubernetes com o K3D
```bash
k3d cluster create meucluster --servers 3 --agents 3 -p "30000:30000@loadbalancer"
```
### K3D

Ferramenta para auxiliar na montagem de um cluster kubernetes local para provas de conceito e utilizar como ambiente de estudo. O processo de instalação pode ser consultado no site da ferramenta: https://k3d.io/

> [!NOTE]
> Outras ferramentas existentes para criar um cluster kubernetes local:
> - K3S
> - MicroK8S
> - Kind
> - Minikube

> [!IMPORTANT]
> É recomendado a intalação do **kubectl**, que é uma ferramenta de linha de comando para interagir com o cluster.

#### Criação de cluster com k3d

##### k3d cluster create
Cria cluster kubernetes com apenas um workernode.

Parâmetros:
- <nome>
- --servers <numero de servers>
- --agents <número de agents>

Visualizar os nodes: `kubectl get nodes`

#### k3d cluster list
Lista os clusters criados

#### k3d cluster stop

#### k3d cluster start

#### k3d cluster delete

#### Comando para criar o cluster com k3d e executar a aplicação:
```Bash
k3d cluster create meucluster -p "8080:30000@loadbalancer"
```

### Subir uma aplicação no cluster Kubernetes local

#### Principais elementos de deploy do Kubernetes
##### Pod
Menor objeto dentro de um cluster kubernetes. É nele que serão executados os containers.

Deve-se colocar cada container da nossa aplicação dentro de um pod pois é ele quem escalamos quando precisamos de melhorar o desempenho da aplicação num curto espaço de tempo (a exceção é quando utilizamos o padrão Sidecar).

##### ReplicaSet
Elemento que gerencia os pods do cluster. Será responsável pela resiliência da aplicação (quando um pod parar, ele automaticamente inicializa outro).

##### Deployment
Gerencia os replicaSets do cluster.
Cria o replicaSet especificando a versão da aplicação que está sendo executada. 

##### Labels e Selectors
Forma de conectar os objetos dentro do cluster.

Labels são elementros chave/valor que são utilizados para marcar os objetos no cluster kubernetes.
Exemplo: marcar um pod com as seguintes labels:
- app: web-color
- versao: v2
- ambiente: homolog

Selectors é a seleção de objetos baseado em algumas labels.
Exemplo: marcar o deplayment e o replicaSet com o seguinte selector:
- app: web-color
- versao: v2
- ambiente: homolog

Labels marca os objetos dentro do cluser e selectors fazem a ligação entre objetos.

#### Service Discovere
Ponto único de comunicação entre aplicações dentro do Kubernetes.
Responsável por fazer balanceamento de carga da comunicação.

##### Tipos de Service

- ClusterIP: ponto único de comunicação para serviços rodando dentro do mesmo cluster (apenas comunicação interna).
- NodePort: cria acesso externo para os pods.
- LoadBalancer: cria um load balancer no cloud provider e expões um IP público.

### Deploy de uma aplicação

#### Preparando o cluster
Criar cluster kubernetes: `k3d cluster create <nome-cluster> --servers 3 --agents 3 -p "30000:30000@loadbalancer"`

Listar os nós criados: kubectl get nodes

#### Preparando a imagem da aplicação

Criar imagem do projeto a partir do dockerfile:

docker build -t tmansur/review-filmes:v1 -f <caminho-dockerfile> <caminho-diretório-arquivos-programa> (Ex: docker build -t tmansur/review-filmes:v1 -f Review-Filmes.Web/Dockerfile .)

Verificar imagem criada: docker image ls

Enviar imagem para o Docker Hub: docker push tmansur/review-filmes:v1

#### Planejando e executando o deploy

![planejamento-deploy](https://github.com/tmansur/devops4devs-02/assets/18071398/62e3ea2e-cbf8-4c0c-9a70-af09227d1d86)

Para fazer o deploy do serviço no kubernetes temos que criar um arquivo .yaml com as informações que serão executadas.

> [!TIP]
> Algumas informações utilizadas nesse arquivo podem ser obtidas executando o seguinte comando dentro do kubernetes : `kubectl api-resources`.

~~~ YAML

~~~

Criando os objetos no Kubernetes: `kubeclt apply -f <caminho/arquivo.yaml>`

Listando objetos criados: `kubectl get pod`, ` kubectl get deploy`, `kubectl get replicaset`, ` kubectl get service`, `kubeclt get all`

Para testar o funcionamento do service (mesmo ele expondo o serviço apenas internamente no cluster) podemos usar o comando : `kubectl port-forward service/postgre 5432:5432`. Esse comando faz um **redirecionamento temporário** da porta do cluster para a porta do nosso computador.

#### Rollout de versão

Lista histórico de deploys: `kubectl rollout history deployment <serviço>` (Exemplo: kubectl rollout history deployment reviewfilmes)

Fazer rollout para uma versão anterior a que está executando: `kubectl rollout undo deployment <serviço>`

## Aula 03 - Deploy ágil e seguro na AWS

### Link para a AWS:

[https://aws.amazon.com](https://aws.amazon.com)

### Link para instalação do AWS CLI:

[https://aws.amazon.com/pt/cli](https://aws.amazon.com/pt/cli)

Para configurar o AWS CLI para um usuário temos que criar um Access Key para o usuário (no portal da AWS | IAM) e executar o comando `aws configure`.

### EKS - Elastic Kubernetes Service

Serviço de Kubernete gerenciado da AWS (PaaS).

#### Criação das roles para cluster Kubernetes e dos workers nodes
Roles são permissões para um serviço utilizar/acessar outro na AWS.

IAM | Roles | Create Role
- Criação de role para EKS Cluster
- Criação de role EC2 para Worker Node

#### Criação da estrutura de redes

Acessar a opção Cloud Formation e utilizar o link a seguir para criação da rede.

URL do template usado do Cloud Formation:
```
https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
```

#### Criação do cluster Kubernetes com AWS AKS 

- Criação do cluster
- Configurar o Kubectl para conectar ao cluster criado:
  - Configurando a conexão: aws eks update-kubeconfig --name <nome-do-cluster>
  - Testando: kubectl get pod
- Criação dos worker nodes no cluster. Acessar o cluster criado | compute | Add node group
  - Testando: kubectl get nodes

#### Fazendo o deploy do projeto

Alterar o arquivo deployment.yaml para que o service que expõe o serviço web trabalhe com XXX e não mais com nodePort:

~~~
// deployment.yaml
...
apiVersion: v1 
kind: Service   
metadata:          
  name: reviewfilmes
spec:
  selector:
    app: reviewfilmes 
  ports:
    - port: 80
      targetPort: 8080
      # nodePort: 30000 # Configuração para rodar localmente
  # type: NodePort # Configuração para rodar localmente
  type: LoadBalancer # Configuração para rodar no kubernetes na nuvem
~~~

kubectl apply -f <arquivo.yaml>

> [!CAUTION]
> Para não ser cobrado pelo ambiente utilizado, é importante excluí-lo logo após realização dos testes:
> - Deletar a aplicação: kubectl delete -f <arquivo-deploy.yaml>
> - Deletar o node group via portal do AWS
> - Deletar o cluster via portal da AWS
> - Deletar a estrutura de rede via Cloud Formation

# Aula 04 - Pipelines inteligentes com Github Actions

# Aula 05 

### Comando para obter a senha do Grafana
```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```
