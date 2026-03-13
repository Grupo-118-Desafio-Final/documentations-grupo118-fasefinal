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

![Code Coverage - API de Usuários e Planos](final-challenge-grupo-118-users%20-%20code%20coverage.PNG)

O microsserviço de Usuários e Planos é responsavel por:
- Gerenciar cadastro de Usuários
- Gerenciar Cadastro de Planos
  - Gerenciar as configurações sobre qualidade das imagens e tamanho dos vídeos

Utilizamos SQL Server para realizar o armazenamento e o cadastro da nossa base de usuários

## 2. API de Upload de Vídeos

- Repositório: [final-challenge-grupo-118-upload-orchestrator](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-upload-orchestrator)

![Code Coverage - API de Upload de Vídeos](final-challenge-grupo-118-upload-orchestrator%20-%20code%20coverage.PNG)

Esse microsserviço é responsável por
- Executar o gerenciamento dos vídeos
- Execução do upload via chunks
- Notificar o Worker para realizar o corte das imagens

Utilizamos uma estrutura de MongoDB devido a simplicidade dos dados que precisam ser armazenados

Para interagir com essa API, criamos um frontend no repositório [final-challenge-grupo-118-frontend](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-frontend), permitindo assim uma interação mais fluída no upload dos vídeos.

OBS: Optamos por não realizar o deploy desse frontend pois não era o foco do projeto, porém em um sistema produtivo, iriamos também utiliza-lo em nuvem.

## 3. Worker de Corte de Imagens

- Repositório: [final-challenge-grupo-118-videos-consumer](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-videos-consumer)

![Code Coverage - Corte de Imagens](./final-challenge-grupo-118-videos-consumer%20-%20code%20coverage.png)

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

![Code Coverage - Worker de Notificações](./final-challenge-grupo-118-notification%20-%20code%20coverage.png)

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

OBS: A parte de gerenciamento das tabelas (DDL) utilizamos a estrutura de Code-First (onde a aplicação é responsável por criar a estrutura e aplicar as migrações), logo, não temos script de banco
- no SQL, criamos as tabelas assim
- no MongoDB, criamos o indice para facilitar o acesso das informações assim

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

# Processo de Build e Deploy

- Repositório: [final-challenge-grupo-118-template-pipeline](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-template-pipeline)

Para facilitar o deploy e garantir um processo padronizado entre os membros do grupo, criamos templates de pipeline de CI e CD utilizando Github Actions

- Templates de Build de aplicações dotnet e typescript
- Template de deploy genérico
- Criação e destruição de recursos Terraform)

## Durante o pull request
- Compilação do código: validação do build do projeto em um ambiente limpo
- Execução dos testes automatizados: executa todos os projetos de teste contidos na solução
- Análise de qualidade de código com SonarCloud: o template de pipeline cria automaticamente o projeto no SonarCloud caso ele não exista, utilizando as variáveis de ambiente configuradas no repositório
  - Caso o quality gate não atinja o percentual de 80%, é inserido no PR um comentário automático informando a falha na qualidade do código e o mesmo fica bloqueado para merge.
- Build da imagem Docker: Validação se o build em um contexto de container ocorre corretamente.

## Na main
Além de todos os passos acima, na main o pipeline realiza também:
- Push da imagem Docker para o Container Registry: a imagem é enviada para o Azure Container Registry (ACR) para ser utilizada posteriormente no deploy.

## Continous Deployment e Helm Charts
Mantendo a mesma filosofia de padronização, criamos um helm chart que também é salvo no ACR para que todas as aplicações o utilizem. Esse Helm chart é responsável por:
- Criar a estrutura classica para funcionamento em Kubernetes (deployments, services, secrets)
- Criação do Ingress interno para comunicação via APIM
- Criação de Secret de ACR para permitir pull de imagens privadas

E para o deploy temos um pipeline padrão, também via Github Actions, que realiza as seguintes etapas:
- Mapeamento de Github secrets para variáveis de ambiente do pipeline
- Login no ACR
- Pull do Helm Chart
- Proceso de substituição de variáveis no Helm Chart
- Deploy via Helm no cluster AKS

Desse modo, a configuração das variaveis que cada aplicação utiliza ficou de maneira bem flexível

No momento do deploy, precisamos apenas informar qual a versão do Helm chart e qual a versão da imagem gera

<TODO - Imagem de workflow>

Feito isso, o processo acontece de forma automática

<TODO - Imagem de pipeline executando>

Abaixo documentamos em um modelo de diagrama, todo nosso processo de CI e CD

![Panorama Geral - CI e CD](CI-CD%20Structure/Panorama%20Geral%20-%20CI%20e%20CD.png)

## Secrets

Todos os pipelines reaproveitam alguns secrets configurados no Github Actions para garantir a segurança das credenciais utilizadas durante o processo de CI/CD.
- Credenciais Azure
- Credenciais ACR
- Credenciais SonarCloud

# Configurações
Alguns passos, manuais até então, são necessários para o funcionamento correto do sistema:
1. Conexão no Kubernetes via kubectl port-foward para buscar o Swagger 
   - Como as APIs não são expostas publicamente, é preciso importar manualmente o Swagger para o APIM (efetuando o download do mesmo). Para isso, precisamos acessar as APIs via kubectl port-forward

Exemplo para o módulo de vídeos:

```bash
kubectl port-forward svc/final-challenge-grupo-118-upload-orchestrator 8080:80 -n tech-challenge
```

2. Importação do Swagger no APIM
   - Após baixar o Swagger via port-forward, é necessário importar o mesmo no APIM para que as APIs fiquem disponíveis para consumo
   - No portal do APIM, selecionar "APIs" e clicar em "+ Add API"
   - Selecionar "OpenAPI" e fazer o upload do arquivo Swagger baixado
   - Após o upload, configurar o "Web service URL" para o endpoint correto (exemplo: http://10.10.0.10/uploads-api)
     - é feito dessa maneira pois o Ingress configurado está pronto para substituir e enviar corretamente para as APIs, o path substituido.

# Observabilidade

Para enriquecer a experiência do monitoramento, criamos uma instância do Grafana Cloud e, através de um operador do [Grafana Alloy](https://grafana.com/docs/alloy/latest/), recebemos Traces, Logs e Metrics para envio para o cloud

## Trace

Abaixo, temos um exemplo de um trace capturado no nosso consumidor de notificações para busca de informações do usuário

![Observability - Traces](./Observability/Observability%20-%20Traces.png)

## Logs
Também inserimos logs (principalmente no consumidor que gera os frames) para facilitar diagnósticos

![Observability - Logs](./Observability/Observability%20-%20Logs.png)


## Metrics

EM um momento inicial, inserimos alguns dashboards pré-configurados para visualização dos recursos computacionais dentro do cluster kubernetes além de informações sobre as requisições efetuadas para as rotas das APIs

### Observability - Metrics 1
![Observability - Metrics 1](./Observability/Observability%20-%20Metrics%201.png)

### Observability - Metrics 2
![Observability - Métrics 2](./Observability/Observability%20-%20Metrics%202.png)

# Diferenciais

No desenho da nossa solução, optamos por elaborar o conceito de `Plano`, onde podemos controlar e evoluir o sistema em torno dos tipos de plano e seus benefícios.

Nessa versão, englobamos:
- quantidade de frames
- qualidade das imagens

Porém a ideia seria evoluir ainda mais de maneira que poderiamos ter 
- filas de processamento separadas por plano
- deployment dos consumidores separados, tendo mais recursos computacionais, para gerar os zips mais rapidamente.
- etc

# Diagramas

Abaixo elaboramos alguns diagramas C4 para ilustrar a arquitetura do sistema

## Diagrama C4 - Contexto do Sistema

Nessa visão mostramos em alto nível a ideia sobre o nosso sistema

![Diagrama de Contexto - Desafio Final FIAP X](C4%20Diagrams/Diagrama%20de%20Contexto%20-%20Desafio%20Final%20FIAP%20X.png)

## Diagrama C4 - Container

Nessa visão, detalhamos um pouco como as APIs e os componentes de infraestrutura interagem entre si

![Diagrama de Container - Desafio Final FIAP X](C4%20Diagrams/Diagrama%20de%20Container%20-%20Desafio%20Final%20FIAP%20X.png)

## Diagrama de Sequência

Nessa visão, mostramos como é o caminho para uso do sistema, passando pelas rotas, as áreas de contexto do sistema etc

![Diagrama de Sequência - FIAP X](C4%20Diagrams/Diagrama%20de%20Sequência%20-%20FIAP%20X.png)