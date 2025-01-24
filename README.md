# Desafio Técnico - Winover

# Fonte de dados

## Football API

Utilizaremos a [Football API](https://footballapi.com/) como fonte de dados para nosso serviço. Ela fornece uma base de dados rica e abrangente, com informações detalhadas sobre os times, jogadores, jogos e estatísticas históricas. Além disso, a Football API suporta atualizações em tempo real através de webhooks, o que garante que os dados estejam sempre atualizados com os eventos mais recentes.

O processo de extração de dados será feito por meio de requisições HTTPS para os endpoints RESTful fornecidos pela API. No caso de webhooks, o pipeline será automaticamente notificado sobre eventos relevantes, como gols ou início/fim de partidas.

## Pré-população
Antes de iniciar o serviço, utilizaremos a Football API para pré-popular nosso banco de dados com informações completas de todos os times da Premier League. Esse processo contemplará:

- Extração inicial de dados de todos os times, com detalhes como nome, logo, partidas, e estádio.

Esses dados servirão como base para que o aplicativo tenha conteúdo mesmo antes de eventos em tempo real serem consumidos.

# Processamento de dados

## Atualizações em Tempo Real
Nossa API será atualizada em tempo real utilizando do mecanismo Polling:

### Chamadas agendadas (Polling)
Utilizando o Firebase Cloud Scheduler, faremos requisições periódicas à API para buscar informações de partidas em andamento. Essas requisições serão programadas para ocorrer a cada hora, garantindo que o banco de dados esteja sincronizado com as atualizações mais recentes acerca das partidas realizadas e da classificação do campeonato.

### Webhooks
Como alternativa ao polling, podemos utilizar webhooks fornecidos pela Football API para tratar eventos em tempo real.

Para isso, será necessária a criação de um endpoint dedicado que escute e processe as notificações enviadas pela Football API. Esse endpoint validará as mensagens recebidas (por exemplo, verificando assinaturas ou tokens de autenticação) e fará o processamento e atualização no banco de dados.

# Armazenamento

Nossa API fará uso do Firebase Firestore, que é um banco de dados não relacional (NoSQL) altamente escalável. Ele organiza os dados em collections e documents, o que se alinha bem à estrutura do nosso projeto, pois:

- Cada time será armazenado como um document dentro de uma collection `teams`.
- Dados de partidas serão organizados em uma collection `matches`.
- Haverá, ainda, uma tabela `standings`, populada por uma Cloud Function disparada a partir de alterações na tabela `matches`.

# Monitoramento e Escalabilidade

## Monitoramento

Para garantir que o pipeline funcione corretamente, utilizaremos os serviços de monitoramento do Google Cloud, como:

- Cloud Monitoring para gerar métricas de uso, desempenho e latência das funções.
- Cloud Logging para capturar logs detalhados sobre cada requisição, permitindo identificar rapidamente erros e gargalos.

## Escalabilidade

As Google Cloud Functions oferecem escalabilidade automática, ajustando dinamicamente os recursos alocados com base no número de requisições simultâneas. Isso elimina a necessidade de provisionamento manual de servidores e garante que o sistema suporte picos de tráfego, como durante grandes jogos ou eventos importantes da Premier League.

Adicionalmente, a Firestore também escala automaticamente para grandes volumes de dados e acessos, mantendo baixa latência.

# Integração com o aplicativo

O aplicativo usará o SDK oficial do Firebase para acessar os dados diretamente do Firestore. O uso do SDK traz as seguintes vantagens:

## Sincronização em Tempo Real
O Firestore possui suporte nativo para sincronização em tempo real. Isso significa que qualquer alteração nos dados armazenados (por exemplo, a atualização do placar de uma partida) será refletida automaticamente no frontend do aplicativo, sem a necessidade de polling ou atualizações manuais.

## Acesso Seguro aos Dados

Utilizando as regras de segurança do Firebase, podemos garantir que cada usuário do app terá acesso somente aos dados que estão autorizados a visualizar. Isso inclui:

- Garantir que o aplicativo apenas leia os dados, enquanto todas as operações de escrita e transformação são gerenciadas pela API backend.
- Permitir somente a usuários autorizados/autenticados a ler os dados.

## Estrutura e Endpoints

No Firestore, os dados serão organizados em coleções e documentos, facilitando o consumo pelo SDK:

- `teams`: Contém informações sobre os times (nome, logo, estádio).
- `matches`: Atualizações em tempo real de partidas em andamento.
- `standings`: Tabela de classificação atualizada do torneio (derivada da tabela `matches`).

## Fluxo de Comunicação Backend-Frontend

O backend é responsável por gerenciar atualizações de dados, seja por polling ou webhooks, e salvar as informações processadas no Firestore.

O app consome essas informações diretamente, utilizando o SDK do Firebase, o que reduz a carga sobre o backend e melhora a performance.

### Relacionamento com as Telas do App

#### Tela 1: Standings (Listagem da Tabela)
- **Dados Consumidos:**
  - Posição do time
  - Nome, escudo
  - Pontuação total
  - Partidas jogadas
  - Vitórias
  - Saldo de gols

- **Origem dos Dados:**
  - Collection: `teams`
  - Collection: `standings` (ordenada pela pontuação total)

#### Tela 2: Club Details (Detalhes do Time)
- **Dados Consumidos:**
  - Nome, escudo e estádio do time
  - Posição e pontuação atual
  - Lista de próximas partidas, incluindo adversário, local (Home/Away) e escudo

- **Origem dos Dados:**
  - Collection: `teams`
  - Subdocumento: `matches` (filtro por partidas não finalizadas, ordenado por data de forma ascendente)
  - Documento: `standings` (pontos e posição)

Esse fluxo garante que o aplicativo mantenha consistência e alta performance com base nos dados gerenciados pelo backend e armazenados no Firestore.

# Fluxo de dados
![Fluxograma de dados](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Fluxograma.png?raw=true)

# Protótipo do client mobile
[Figma](https://www.figma.com/proto/gPiRKiaCbCd57jEmOtWqrw/Projeto-Eduardo?node-id=0-1&p=f&t=N0gIss1BBlX1sBsn-0&scaling=scale-down&content-scaling=fixed&starting-point-node-id=1%3A4620&show-proto-sidebar=1)

#### Screenshots
![Phone - Welcome](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Welcome.png?raw=true)
![Phone - Login](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Login.png?raw=true)
![Phone - Standings](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Standings.png?raw=true)
![Phone - Club Details](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Club%20Details.png?raw=true)
![Tablet - Welcome](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Tablet%20Welcome.png?raw=true)
![Tablet - Login](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Tablet%20Login.png?raw=true)
![Tablet - Standings](https://github.com/duhdoesk/Desafio-T-cnico---Winover/blob/main/Tablet%20Standings.png?raw=true)
