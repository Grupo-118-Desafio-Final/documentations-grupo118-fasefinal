# documentations-grupo118-fasefinal

Criamos um repositório separado para armazenar diversos detalhes acerca da implementação do projeto na Fase Final

# Membros do Grupo
- Sabrina Cardoso de Oliveira
    - **Matrícula**: RM363507
    - **Usuário Discord**: sah.mdo
- Tiago Cristiano Koch
    - **Matrícula**: RM361415
    - **Usuário Discord**: tiagokoch0076
- Tiago Victor de Oliveira
    - **Matrícula**: RM364588
    - **Usuário Discord**: oliveirad.tiago
- Túlio Henrique de Paula Rezende
    - **Matrícula**: RM360982
    - **Usuário Discord**: tuliomamute
- Vinícius Rossmann Nunes
    - **Matrícula**: RM362963
    - **Usuário Discord**: _viniciusnunes

# Vídeo de Apresentação do Projeto

Para assistir ao vídeo de apresentação do projeto, clique na imagem abaixo:

[![Watch the video](11SOAT%20-%20Fase%20Final%20-%20Grupo%20118.PNG)]()


# Collection do Postman
Para facilitar o teste dos endpoints da API, foi criada uma que contém os endpoints utilizados no video.

[![Run in Postman](https://run.pstmn.io/button.svg)](./Testes/postman_collection.json)

# Módulos do Sistema
Dividimos a aplicação em alguns serviços (entre APIs e Workers) para permitir uma maior escalabilidade da solução de acordo com a demanda

Utilizamos mecanismos de Mensageria para garantir que não iremos perder elementos no fluxo de processamento, Incluindo a maneira como realizamos o aviso para o sistema de mensageria, informando sucesso apenas se tudo que é proposto no fluxo foi feito corretamente (ACK e NACK)

## 1. API de Usuários e Planos

- Repositório: [final-challenge-grupo-118-users](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-users)
- Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

O microsserviço de Usuários e Planos é responsavel por:
- Gerenciar cadastro de Usuários
- Gerenciar Cadastro de Planos
  - Gerenciar as configurações sobre qualidade das imagens e tamanho dos vídeos

Utilizamos SQL Server para realizar o armazenamento e o cadastro da nossa base de usuários

## 2. API de Upload de Vídeos

- Repositório: [final-challenge-grupo-118-upload-orchestrator](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-upload-orchestrator)
- Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

Esse microsserviço é responsável por
- Executar o gerenciamento dos vídeos
- Execução do upload via chunks
- Notificar o Worker para realizar o corte das imagens

Utilizamos uma estrutura de MongoDB devido a simplicidade dos dados que precisam ser armazenados

Para interagir com essa API, criamos um frontend no repositório [final-challenge-grupo-118-frontend](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-frontend), permitindo assim uma interação mais fluída no upload dos vídeos

## 3. Worker de Corte de Imagens

- Repositório: [final-challenge-grupo-118-videos-consumer](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-videos-consumer)
- Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

Esse consumidor é ativado a partir de uma mensagem via serviço de mensageria para:
- Realizar o download do vídeo publicado no Blob Storage
- Realizar os cortes das imagens com a qualidade de acordo com o plano
- Geração do ZIP
- Publicação no Blob Storage
- Atualização do registro no MongoDB com a URL do ZIP gerado

Decidimos pelo worker também alterar o registro no MongoDB diretamente por fazer parte do mesmo "contexto" da API de vídeos e tratarem do mesmo assunto (gerenciamento de arquivos)

## 4. Worker de Notificações

- Repositório: (final-challenge-grupo-118-notification)[https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-notification]
- Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

Esse consumidor é ativado a partir de uma mensagem via serviço de mensageria para
- Avisar caso de sucesso no corte das imagens
- Avisar caso de erro no corte das imagens
- Qualquer notificação geral necessária no sistema

Esse serviço também consulta a API de planos e usuários para validar quais os meios possíveis para notificar o usuário (ex: email)

# Infraestrutura e Padronização
Para a parte de infraestrutura e padronização do projeto, criamos repositórios adicionais para permitir a separação e paralelização das atividades

## 1. Infraestrutura

- Repositório: [final-challenge-grupo-118-terraform-infra](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-terraform-infra)

Para a infraestrutura, utilizamos uma abordagem semelhante aos trabalhos anteriores com

- Azure Container Registry
- Kubernetes
- API Gateway
- Ingress Interno para comunicação do API Gateway -> Kubernetes dentro da rede da Azure

Além dos recursos descritos, foi necessário adicionar

- Storage Account: para armazenamento das imagens e vídeos
- Messaging: utilização da infraestrutura da CloudAMQP para o serviço de mensageria (RabbitMQ ou LavinMQ, de acordo com a necessidade)

## 2. Bases de Dados

- Repositório: [final-challenge-grupo-118-terraform-database](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-terraform-database)

Na parte de armazenamento, mantivemos o mesmo modelo dos trabalhos anteriores com

- SQL Server
- MongoDB

## 3. Observabilidade

- Repositórios: 
  - [final-challenge-grupo-118-lgtm-keycloak](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-lgtm-keycloak)
  - [final-challenge-grupo-118-standard-dependencies](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-standard-dependencies)

Para a observabilide, criamos um repositório que irá instalar o Grafana Alloy dentro do cluster e realizar o envio de Métricas, Traces e Logs para uma instância de Grafana Cloud

E para manter o modelo padronizado criamos um Nuget Package, para facilitar a adoção e configuração dos serviços

## Estrutura de Deploy

- Repositório: [final-challenge-grupo-118-helm-chart](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-helm-chart)

Criamos um Helm Chart padronização, que é armazenado no ACR criado na infraestrutura, para manter unificada as configurações necessárias para o deploy no cluster Kubernetes

## Ambiente de Desenvolvimento

- Repositório: [final-challenge-grupo-118-compose-to-develop](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-compose-to-develop)

Para agilizar o processo de desenvolvimento, criamos esse repositório contendo os serviços cloud que iriamos entregar, de maneira local. Dessa maneira não precisariamos provisionar cedo a infraestrutura