# Nossas Lojas V2

API de estudo para gerenciar lojas físicas, agora implementada com TypeScript, Express e TypeORM sobre MySQL. Mantém o contexto da [trilha de microsserviços Node.js](https://github.com/myreli/c_trilhas/blob/master/microservices_node/desafio2/README.md), com consultas por identificador, estado e conjunto de cidades.

[![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white)](https://nodejs.org/)
[![MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

## 📋 Table of contents

- [✨ Features](#features)
- [🧪 Technologies](#technologies)
- [🛠️ Environment setup](#environment-setup)
- [🚀 Getting started](#getting-started)
- [🐋 Getting started with Docker](#docker)
- [🔌 API usage](#api)
- [🗂️ Project structure](#structure)
- [✅ Testing and quality](#testing)
- [🔎 Troubleshooting](#troubleshooting)
- [🤝 Contributing](#contributing)
- [📜 License](#license)

<a id="features"></a>

## ✨ Features

- Cadastro, edição, remoção e listagem de lojas.
- Busca por ID, estado e conjunto de cidades de um estado.
- Validação de dados de entrada e respostas padronizadas para alguns erros.
- Swagger UI em `/api-docs`, baseado em [Documentacao/swagger.json](Documentacao/swagger.json).

<a id="technologies"></a>

## 🧪 Technologies

Versões declaradas em [package.json](package.json); não há lockfile ou runtime fixado.

| Tecnologia | Declaração | Finalidade |
| --- | --- | --- |
| [TypeScript](https://www.typescriptlang.org/docs/) | `^2.8.1` | Compilação de `src/` para `js/` |
| [Express](https://expressjs.com/) | `^4.16.3` | API HTTP |
| [TypeORM](https://typeorm.io/) | `^0.2.0` | Entidades e persistência |
| [mysql](https://github.com/mysqljs/mysql) | `^2.15.0` | Driver MySQL |
| [swagger-ui-express](https://github.com/scottie1984/swagger-ui-express) | `^4.0.2` | Interface da documentação |
| [nodemon](https://nodemon.io/) | `1.18.4` | Reinício do servidor |
| [concurrently](https://github.com/open-cli-tools/concurrently) | `^3.4.0` | Execução conjunta dos observadores |

O campo `version` de `tsconfig.json` não fixa a versão do compilador; a dependência do manifesto é a referência.

<a id="environment-setup"></a>

## 🛠️ Environment setup

Tenha Git, Node.js e MySQL acessível. O repositório não fixa versões de Node ou MySQL. Foi escolhido Node.js 14.21.3 para testar esta base antiga de Express/TypeORM; a compilação ainda depende da compatibilidade entre os intervalos sem lockfile.

```bash
nvm install 14.21.3
nvm use 14.21.3
git clone https://github.com/paulodaluz/NossasLojasV2.git
cd NossasLojasV2
npm install
```

Use npm, como o Dockerfile; não há lockfile para `npm ci`. Não há hooks ou `prepare` e os executáveis são dependências locais.

Configure `host`, `port`, `username`, `password` e `database` em [ormconfig.json](ormconfig.json), usando valores de seu ambiente. A porta MySQL declarada é 3306. Não existe `connectionFactory.js` nesta versão nem carregamento de `.env`.

O arquivo define `synchronize: false` e não há migrações versionadas. Crie uma base de desenvolvimento e a tabela correspondente à entidade `ListaLojas`:

```sql
CREATE DATABASE lojas_exemplo;
USE lojas_exemplo;
CREATE TABLE lista_lojas (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  nome_loja VARCHAR(255) NOT NULL,
  endereco VARCHAR(255) NOT NULL,
  celular VARCHAR(255) NOT NULL,
  cnpj VARCHAR(255) NOT NULL,
  horarioDeTrabalho VARCHAR(255) NOT NULL,
  cidade VARCHAR(255) NOT NULL,
  estado VARCHAR(255) NOT NULL
);
```

Os caminhos de entidades apontam para `js/entity/*.js`; a compilação precisa ocorrer antes da conexão.

<a id="getting-started"></a>

## 🚀 Getting started

Compile usando a dependência local e, se a compilação terminar sem erros, inicie:

```bash
./node_modules/.bin/tsc
npm run server
```

Para desenvolvimento, `npm start` executa `npm run watch` (`tsc -w`) e `npm run server` (`nodemon js/index.js`) em paralelo. Não há script `build` ou `start:prod`; `node js/index.js` executa diretamente o resultado compilado.

O servidor escuta **3000**, após conseguir conectar ao MySQL. Acesse `http://localhost:3000/api-docs`; a raiz redireciona para essa página. Uma consulta de integração é:

```bash
curl http://localhost:3000/listaLojas
```

Não há endpoint de saúde independente do banco. Encerre os processos com `Ctrl+C`.

<a id="docker"></a>

## 🐋 Getting started with Docker

```bash
docker build -f node.dockerfile -t nossas-lojas-v2 .
docker run --name nossas-lojas-v2 -p 3000:3000 nossas-lojas-v2
docker stop nossas-lojas-v2
docker rm nossas-lojas-v2
```

**Inspeção somente: comandos Docker não executados nesta tarefa.** O contexto é a raiz, a base é `node:latest` e a entrada é `npm start`, com os observadores de compilação e servidor. Não há Compose, banco ou volumes definidos. Configure um MySQL alcançável pelo container; `localhost` dentro dele não é o host.

O Dockerfile copia o repositório inteiro e desabilita a verificação TLS do npm. A versão flutuante de Node pode divergir do ambiente de validação. Essas configurações, bem como a proteção de arquivos com credenciais no contexto, precisam de revisão separada.

<a id="api"></a>

## 🔌 API usage

Não há autenticação nas rotas. Use `Content-Type: application/json` nos corpos.

| Método | Caminho | Uso |
| --- | --- | --- |
| POST | `/criaLoja` | Criar |
| PUT | `/editaLoja/:id` | Editar |
| DELETE | `/deletaLoja/:id` | Remover |
| GET | `/buscaPorId/:id` | Buscar por ID |
| GET | `/buscaPorEstado/:estado` | Buscar por estado |
| POST | `/buscaPorCidades/:estado` | Buscar cidades informadas no corpo |
| GET | `/listaLojas` | Listar todas |
| GET | `/` | Redirecionar para `/api-docs` |

Exemplo de consulta por cidades, após configurar o banco:

```bash
curl -X POST http://localhost:3000/buscaPorCidades/SP   -H 'Content-Type: application/json'   -d '{"cidades":["Cidade Exemplo","Outra Cidade"]}'
```

Cadastro e edição recebem `nome_loja`, `endereco`, `celular`, `cnpj`, `horarioDeTrabalho`, `cidade` e `estado`, todos como texto. A validação está em [src/models/Validacao.ts](src/models/Validacao.ts); o CNPJ exige 18 caracteres e o estado, dois. Confira os contratos antigos contra `src/rotas.ts` antes de importar requisições.

<a id="structure"></a>

## 🗂️ Project structure

| Caminho | Responsabilidade |
| --- | --- |
| `src/index.ts` | Conexão, Swagger e inicialização HTTP |
| `src/rotas.ts` | Métodos e caminhos |
| `src/controller/Controler.ts` | Operações sobre lojas |
| `src/entity/ListaLojas.ts` | Modelo persistido |
| `src/models/` | Validação e respostas |
| `ormconfig.json` | MySQL e caminhos compilados |
| `Documentacao/` | Especificação Swagger |
| `js/` | Saída gerada pelo TypeScript |

<a id="testing"></a>

## ✅ Testing and quality

Não existem scripts de testes, lint ou formatação. `./node_modules/.bin/tsc` é a verificação de compilação disponível.

Instalação concluída com Node.js 14.21.3/npm 6.14.18. `./node_modules/.bin/tsc` falhou (código 2) em tipos de dependências atuais, incompatíveis com o compilador antigo. Mesmo com erros, foram emitidos arquivos em `js/`; tentar `node js/index.js` registrou `ECONNREFUSED 127.0.0.1:3306` e encerrou sem abrir o HTTP. A saída de processo 0 nesse caso não significa inicialização bem-sucedida. API, persistência e Swagger não foram verificados. Captura de backend pendente, pois VS Code e aparência atual do usuário não puderam ser confirmados. Docker não foi executado.

<a id="troubleshooting"></a>

## 🔎 Troubleshooting

| Situação | Verificação |
| --- | --- |
| Erros TypeScript em dependências | O projeto não trava versões; tipos atuais podem exceder a sintaxe aceita pelo TypeScript 2. A correção de dependências deve ser avaliada separadamente. |
| `js/index.js` ausente | Confira a compilação e o `outDir` em `tsconfig.json`. |
| `TypeORM connection error` | Revise o MySQL e `ormconfig.json`; as rotas só são registradas após conectar. |
| Tabela ausente | `synchronize` está desativado e não há migrações; prepare a tabela em uma base de desenvolvimento. |

<a id="contributing"></a>

## 🤝 Contributing

Crie uma branch a partir de `master`, valide a instalação e os cenários afetados em um banco de desenvolvimento próprio e abra um pull request descrevendo os resultados. Correções de código e atualizações de dependências são entregas separadas. Não publique credenciais nem dados reais.

<a id="license"></a>

## 📜 License

O projeto está sob a [licença MIT](LICENSE), com copyright de 2021 de Paulo da Luz. O manifesto conserva os créditos do exemplo TypeORM de Umed Khudoiberdiev que serviu de base; esses avisos foram preservados.

---

<p align="center">Made with 💜 by <a href="https://github.com/paulodaluz">Paulo da Luz</a></p>
