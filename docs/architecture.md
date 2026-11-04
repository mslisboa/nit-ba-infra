
# Arquitetura da Infraestrutura

## 1. Visão geral

A infraestrutura é responsável por coordenar a execução dos componentes do módulo de integração estadual dos NITs da Bahia.

O projeto utiliza uma arquitetura distribuída em quatro repositórios independentes:

| Repositório | Tecnologia | Responsabilidade |
|---|---|---|
| nit-ba-frontend | Next.js | Interface e visualização |
| nit-ba-backend | FastAPI | API e regras de negócio |
| nit-ba-etl | Python | Extração, transformação e carga |
| nit-ba-infra | Docker Compose | Orquestração |

O PostgreSQL é compartilhado pelos serviços que necessitam acessar os dados.

## 2. Arquitetura dos serviços

O fluxo principal é:

Fontes de dados
    |
    v
ETL (Python)
    |
    v
PostgreSQL
    |
    v
Backend (FastAPI)
    |
    v
API REST
    |
    v
Frontend (Next.js)

O Docker Compose é responsável por executar e conectar os serviços.

## 3. Responsabilidades dos serviços

### 3.1 PostgreSQL

Responsável pelo armazenamento persistente dos dados.

Deve possuir:

- Container próprio;
- Volume persistente;
- Configuração por variáveis de ambiente;
- Healthcheck;
- Rede interna.

O PostgreSQL não deve ser exposto publicamente em produção sem necessidade.

### 3.2 Backend

Responsável pela API e pelas consultas ao banco.

Deve:

- Acessar o PostgreSQL;
- Disponibilizar os endpoints;
- Implementar regras de negócio;
- Controlar as migrações;
- Disponibilizar documentação OpenAPI.

O container do backend deve iniciar somente quando suas dependências essenciais estiverem disponíveis.

### 3.3 Frontend

Responsável pela interface de consulta e visualização.

Deve consumir a API do backend.

Não deve acessar diretamente o PostgreSQL.

É necessário considerar a diferença entre URLs utilizadas pelo navegador do usuário e URLs utilizadas internamente pelos containers.

Não utilizar localhost como endereço interno de comunicação entre containers.

### 3.4 ETL

Responsável pelos processos de extração, transformação, validação e carga.

O ETL deverá:

- Acessar o PostgreSQL;
- Utilizar credenciais próprias;
- Executar pipelines sob demanda;
- Registrar execuções;
- Respeitar os contratos do banco definidos pelo backend.

Não deve ser executado automaticamente junto com a inicialização do backend.

## 4. Redes

Utilizar redes Docker para separar a comunicação dos serviços.

A arquitetura deve garantir:

- Backend com acesso ao PostgreSQL;
- ETL com acesso ao PostgreSQL;
- Frontend com acesso ao backend;
- PostgreSQL sem exposição pública desnecessária.

Sempre que adequado, manter o banco em uma rede interna.

A comunicação entre o navegador e a API deve utilizar um endereço acessível pelo usuário ou uma camada de proxy apropriada.

## 5. Volumes

O PostgreSQL deverá utilizar volume persistente.

Os dados não podem depender do ciclo de vida do container.

Arquivos temporários do ETL devem possuir armazenamento apropriado e não devem ser incluídos nas imagens de aplicação.

Volumes de desenvolvimento e produção devem ser tratados separadamente.

## 6. Execução dos componentes

O ambiente deve permitir:

- Inicialização conjunta da aplicação;
- Inicialização individual dos serviços;
- Execução pontual do ETL;
- Execução controlada das migrações;
- Consulta de logs;
- Reinicialização de serviços;
- Encerramento sem apagar os dados persistentes.

## 7. Desenvolvimento e produção

Manter configurações diferentes para desenvolvimento e produção.

### Desenvolvimento

Priorizar:

- Facilidade de execução;
- Volumes necessários;
- Logs acessíveis;
- Hot reload, quando aplicável;
- Configuração simplificada.

### Produção

Priorizar:

- Imagens construídas previamente;
- Tags de versão;
- Configurações seguras;
- Volumes persistentes;
- Healthchecks;
- Política de reinicialização;
- Controle de acesso;
- Backup e recuperação.

Não utilizar imagens identificadas exclusivamente por latest como estratégia de versionamento de produção.

## 8. Comunicação entre repositórios

Cada repositório mantém sua implementação independente.

A infraestrutura utiliza os Dockerfiles ou imagens produzidos pelos respectivos repositórios.

Alterações nos serviços precisam ser avaliadas quanto ao impacto na orquestração.

Exemplos:

- Mudança de porta;
- Alteração de variáveis de ambiente;
- Alteração do comando de inicialização;
- Alteração de dependências;
- Alteração de healthchecks;
- Alteração de volumes.

Essas mudanças devem ser refletidas na configuração do Compose.

## 9. Banco de dados e migrações

O backend é responsável pelo schema canônico e suas migrações.

A infraestrutura apenas organiza o ambiente necessário para executar essas operações.

As migrações devem ser executadas de maneira controlada.

Evitar a execução concorrente de migrações por múltiplas instâncias.

## 10. Persistência e recuperação

Estabelecer mecanismos de backup do PostgreSQL.

Documentar:

- Como criar backups;
- Onde armazená-los;
- Como restaurá-los;
- Como validar a restauração.

Volumes Docker não substituem uma estratégia de backup.

## 11. Evolução

A infraestrutura deverá permitir a evolução independente dos componentes.

Novos serviços poderão ser incorporados futuramente, desde que suas responsabilidades e dependências sejam documentadas.

Não adicionar serviços de infraestrutura sem necessidade demonstrada.
