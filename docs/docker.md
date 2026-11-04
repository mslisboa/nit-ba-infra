
# Docker e Docker Compose

## 1. Objetivo

Este documento descreve as orientações para containerização e execução integrada dos componentes do módulo de integração estadual dos NITs da Bahia.

O projeto utiliza Docker e Docker Compose.

Cada aplicação possui seu próprio Dockerfile, enquanto este repositório é responsável pela orquestração geral.

## 2. Organização dos arquivos

Estrutura sugerida:

nit-ba-infra/
|
├── AGENTS.md
├── README.md
├── opencode.json
|
├── compose.yaml
├── compose.dev.yaml
├── compose.prod.yaml
|
├── .env.example
├── .gitignore
|
└── docs/
    ├── architecture.md
    └── docker.md

Os arquivos de configuração devem ser ajustados conforme a estrutura real dos repositórios.

## 3. Repositórios das aplicações

Durante o desenvolvimento local, os quatro repositórios podem estar organizados como diretórios irmãos:

nit-ba-workspace/
|
├── nit-ba-frontend/
├── nit-ba-backend/
├── nit-ba-etl/
└── nit-ba-infra/

A configuração de desenvolvimento pode utilizar os Dockerfiles dos diretórios irmãos como contexto de build.

Os caminhos devem ser relativos e documentados.

Não assumir que os repositórios estarão sempre nessa localização.

## 4. Serviços Docker

A aplicação deverá contemplar os seguintes serviços:

| Serviço | Origem | Função |
|---|---|---|
| postgres | Imagem PostgreSQL | Banco de dados |
| backend | nit-ba-backend | API FastAPI |
| frontend | nit-ba-frontend | Aplicação Next.js |
| etl | nit-ba-etl | Processamento dos dados |

Cada serviço deve possuir configuração própria.

## 5. Dockerfiles

Cada repositório de aplicação deverá manter seu próprio Dockerfile.

### Frontend

O Dockerfile deve:

- Instalar dependências;
- Construir a aplicação Next.js;
- Executar a aplicação conforme o ambiente;
- Utilizar variáveis de ambiente necessárias;
- Evitar incluir arquivos desnecessários.

### Backend

O Dockerfile deve:

- Instalar dependências Python;
- Preparar o ambiente FastAPI;
- Configurar o comando de inicialização;
- Permitir execução controlada de migrações;
- Utilizar variáveis de ambiente.

### ETL

O Dockerfile deve:

- Instalar dependências de processamento;
- Preparar os pipelines;
- Permitir execução pontual;
- Não executar todos os pipelines automaticamente durante a inicialização de outros serviços.

Não duplicar os Dockerfiles neste repositório.

## 6. Docker Compose

Utilizar Docker Compose para coordenar os serviços.

A configuração deverá permitir:

- Subir toda a aplicação;
- Executar serviços individualmente;
- Executar o ETL sob demanda;
- Executar migrações de maneira controlada;
- Consultar logs;
- Encerrar serviços.

Utilizar arquivos complementares para configurações específicas de desenvolvimento e produção, evitando duplicações desnecessárias.

## 7. Healthchecks e dependências

O PostgreSQL deve possuir healthcheck.

O backend deve aguardar o banco estar disponível antes de executar operações que dependam dele.

O frontend deve possuir uma verificação de disponibilidade apropriada.

O ETL deve ser executado somente quando necessário.

Não confundir ordem de inicialização de containers com disponibilidade real dos serviços.

Utilizar depends_on com condições apropriadas quando suportado pela configuração adotada.

## 8. Rede

Utilizar redes Docker para a comunicação interna.

O backend e o ETL devem acessar o PostgreSQL através do nome do serviço Docker, e não por localhost.

O frontend deve acessar o backend através de uma URL adequada ao ambiente.

Não expor o PostgreSQL publicamente em produção sem necessidade.

## 9. Volumes

O PostgreSQL deve possuir volume persistente.

Exemplo conceitual:

postgres_data:/var/lib/postgresql/data

O volume não deve ser removido durante operações normais de reinicialização.

Criar volumes separados para desenvolvimento e produção quando necessário.

Não versionar dados persistidos no Git.

## 10. Variáveis de ambiente

Manter um arquivo .env.example contendo somente nomes e valores demonstrativos.

Não versionar o arquivo .env real.

Variáveis esperadas incluem:

POSTGRES_DB
POSTGRES_USER
POSTGRES_PASSWORD
DATABASE_URL
API_PORT
FRONTEND_PORT
NEXT_PUBLIC_API_URL

Os nomes definitivos devem respeitar a configuração de cada aplicação.

### 10.1 Segurança

- Não armazenar senhas no Compose.
- Não incluir tokens no código.
- Não versionar credenciais.
- Não utilizar valores secretos de produção nos ambientes de desenvolvimento.
- Avaliar mecanismos de gerenciamento de segredos para produção.

Não utilizar NEXT_PUBLIC_ para segredos.

## 11. Execução do ambiente

Os comandos devem ser documentados no README após a implementação efetiva do Compose.

Comandos operacionais esperados:

### Inicializar

docker compose up -d

### Consultar containers

docker compose ps

### Consultar logs

docker compose logs -f

### Parar serviços

docker compose down

### Executar o ETL sob demanda

docker compose run --rm etl

A execução real dependerá do comando definido no Dockerfile e da configuração do serviço.

### Consultar apenas um serviço

docker compose logs -f backend

## 12. Migrações do banco

As migrações são responsabilidade do backend.

A infraestrutura deve permitir sua execução controlada.

Não executar migrações automaticamente por múltiplas instâncias.

Antes de subir uma nova versão que altere o schema:

1. Verificar a versão do banco;
2. Verificar as migrações pendentes;
3. Executar a migração;
4. Verificar o resultado;
5. Inicializar os serviços dependentes.

Não executar operações destrutivas sem autorização.

## 13. Execução do ETL

O ETL deve possuir perfil ou mecanismo equivalente para execução sob demanda.

Não manter um container de ETL executando continuamente sem necessidade.

Uma execução deve:

1. Utilizar as variáveis de ambiente;
2. Conectar ao PostgreSQL;
3. Executar o pipeline solicitado;
4. Registrar logs;
5. Encerrar após concluir.

Futuras execuções agendadas poderão utilizar um mecanismo adicional, conforme necessidade.

## 14. Desenvolvimento local

O ambiente de desenvolvimento pode utilizar:

- Volumes de código;
- Hot reload;
- Logs detalhados;
- Configurações simplificadas;
- Banco de desenvolvimento persistente.

Não utilizar essas configurações em produção sem avaliação.

## 15. Produção

O ambiente de produção deve priorizar:

- Imagens previamente construídas;
- Tags de versão;
- Política de reinicialização;
- Healthchecks;
- Configurações seguras;
- Volumes persistentes;
- Backups;
- Redes adequadas;
- Controle de acesso.

Não utilizar bind mounts de código-fonte em produção.

Não utilizar imagens sem versão definida.

## 16. Backup e restauração

Manter procedimentos de backup do PostgreSQL.

Documentar:

- Frequência;
- Armazenamento;
- Retenção;
- Restauração;
- Validação dos backups.

Os backups devem ser armazenados fora do volume principal do banco.

Testar periodicamente a restauração.

## 17. Cuidados operacionais

Não utilizar docker compose down -v sem autorização, pois isso pode remover volumes persistentes.

Não apagar imagens, volumes ou redes sem necessidade.

Antes de executar operações destrutivas:

1. Identificar o recurso afetado;
2. Avaliar os dados armazenados;
3. Verificar a existência de backup;
4. Confirmar a necessidade da operação.

Não armazenar arquivos brutos do ETL nas imagens Docker.

## 18. Validação

Antes de considerar uma alteração concluída:

- Validar a sintaxe do Compose;
- Verificar referências aos Dockerfiles;
- Verificar as variáveis de ambiente;
- Verificar as redes;
- Verificar os volumes;
- Verificar healthchecks;
- Testar inicialização;
- Testar comunicação entre serviços;
- Testar persistência do banco;
- Testar execução do ETL.

## 19. Evolução da infraestrutura

A infraestrutura deve permanecer simples e proporcional às necessidades do projeto.

Não adicionar Kubernetes, orquestradores complexos, brokers de mensagens ou serviços adicionais sem justificativa.

Priorizar Docker Compose no MVP.

Novos componentes devem possuir responsabilidades claramente definidas e documentação correspondente.
