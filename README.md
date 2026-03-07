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

Repositório: [final-challenge-grupo-118-users](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-users)
Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

O microsserviço de Usuários e Planos é responsavel por:
- Gerenciar cadastro de Usuários
- Gerenciar Cadastro de Planos
  - Gerenciar as configurações sobre qualidade das imagens e tamanho dos vídeos

Utilizamos SQL Server para realizar o armazenamento e o cadastro da nossa base de usuários

## 2. API de Upload de Vídeos

Repositório: [final-challenge-grupo-118-upload-orchestrator](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-upload-orchestrator)
Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

Esse microsserviço é responsável por
- Executar o gerenciamento dos vídeos
- Execução do upload via chunks
- Notificar o Worker para realizar o corte das imagens

Utilizamos uma estrutura de MongoDB devido a simplicidade dos dados que precisam ser armazenados

## 3. Worker de Corte de Imagens

Repositório: [final-challenge-grupo-118-videos-consumer](https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-videos-consumer)
Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

Esse consumidor é ativado a partir de uma mensagem via serviço de mensageria para:
- Realizar o download do vídeo publicado no Blob Storage
- Realizar os cortes das imagens com a qualidade de acordo com o plano
- Geração do ZIP
- Publicação no Blob Storage
- Atualização do registro no MongoDB com a URL do ZIP gerado

Decidimos pelo worker também alterar o registro no MongoDB diretamente por fazer parte do mesmo "contexto" da API de vídeos e tratarem do mesmo assunto (gerenciamento de arquivos)

## 4. Worker de Notificações

Repositório: (final-challenge-grupo-118-notification)[https://github.com/Grupo-118-Desafio-Final/final-challenge-grupo-118-notification]
Abordagem da API: <TODO - Colocar modelo escolhido>

<TODO - Code Coverage>

Esse consumidor é ativado a partir de uma mensagem via serviço de mensageria para
- Avisar caso de sucesso no corte das imagens
- Avisar caso de erro no corte das imagens
- Qualquer notificação geral necessária no sistema

Esse serviço também consulta a API de planos e usuários para validar quais os meios possíveis para notificar o usuário (ex: email)