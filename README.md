<div align="center">

# Lamarcks IA

### Assistente de IA com roteamento híbrido e RAG corporativo

Aplicação desenvolvida com **Python, FastAPI, LLMs, busca semântica e banco vetorial**, capaz de diferenciar perguntas gerais de tecnologia de perguntas que dependem de uma base de conhecimento corporativa.

<br>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge\&logo=python\&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge\&logo=fastapi\&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-LLM-F55036?style=for-the-badge)
![ChromaDB](https://img.shields.io/badge/ChromaDB-Vector%20DB-orange?style=for-the-badge)
![RAG](https://img.shields.io/badge/RAG-Generative%20AI-blueviolet?style=for-the-badge)
![OCI](https://img.shields.io/badge/Oracle%20Cloud-OCI-F80000?style=for-the-badge\&logo=oracle\&logoColor=white)

</div>

---

## Sobre o projeto

O **Lamarcks IA** é um assistente de Inteligência Artificial desenvolvido durante minha formação no **Oracle Next Education (ONE) + Alura**.

A aplicação combina um modelo de linguagem com uma arquitetura de **Retrieval-Augmented Generation (RAG)** para responder perguntas de duas formas diferentes:

* **Conhecimento geral:** perguntas conceituais sobre tecnologia são respondidas diretamente pelo modelo de linguagem.
* **Conhecimento corporativo:** perguntas específicas sobre a empresa fictícia Pegasus são encaminhadas para uma base documental privada, onde ocorre uma busca semântica antes da geração da resposta.

O objetivo do projeto foi ir além de um chatbot tradicional e implementar um fluxo completo envolvendo **API, classificação de perguntas, embeddings, recuperação de contexto, geração de respostas e proteção da base documental**.

---

## Objetivo

O projeto busca demonstrar como uma aplicação pode combinar um LLM com informações próprias de uma organização sem simplesmente disponibilizar os documentos internos ao usuário.

Entre os principais objetivos estão:

* Implementar uma API utilizando FastAPI;
* Integrar um modelo de linguagem através da Groq;
* Classificar perguntas de acordo com sua finalidade;
* Implementar um fluxo de RAG;
* Gerar embeddings dos documentos;
* Armazenar embeddings em um banco vetorial;
* Recuperar informações semanticamente relacionadas;
* Utilizar os documentos recuperados como contexto para o LLM;
* Evitar respostas inventadas sobre informações corporativas;
* Impedir solicitações que tentem expor documentos, prompts ou dados internos;
* Disponibilizar informações sobre o estado da aplicação por meio de uma API de status.

---

## Como funciona

O fluxo principal da aplicação pode ser representado da seguinte forma:

```text
                    Usuário
                       │
                       ▼
                Interface Web
                       │
                       ▼
                    FastAPI
                       │
                       ▼
              Classificação da pergunta
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
           GERAL           CORPORATIVO
              │                 │
              │                 ▼
              │           Busca semântica
              │                 │
              │                 ▼
              │             ChromaDB
              │                 │
              │                 ▼
              │        Contexto recuperado
              │                 │
              └────────┬────────┘
                       ▼
                     LLM
                       │
                       ▼
                    Resposta
```

A classificação é feita pelo próprio modelo de linguagem antes da etapa de recuperação.

Quando a pergunta depende de informações específicas da Pegasus, o sistema consulta a base vetorial.

---

## RAG — Retrieval-Augmented Generation

A base corporativa utiliza documentos organizados por áreas de conhecimento.

Atualmente o projeto possui documentos relacionados a:

```text
documentos/
├── arquitetura/
├── backend/
├── frontend/
├── incidentes/
└── onboarding/
```

O processo de indexação segue estas etapas:

```text
Documentos
    ↓
Extração do conteúdo
    ↓
Divisão em chunks
    ↓
Geração de embeddings
    ↓
ChromaDB
```

Quando uma pergunta corporativa é realizada:

```text
Pergunta
    ↓
Embedding da pergunta
    ↓
Busca por similaridade
    ↓
Trechos relevantes
    ↓
Contexto enviado ao LLM
    ↓
Resposta contextualizada
```

A aplicação utiliza **Sentence Transformers** com o modelo:

```text
sentence-transformers/all-MiniLM-L6-v2
```

Os embeddings são normalizados e armazenados no **ChromaDB utilizando distância cosseno**.

---

## Roteamento de perguntas

Uma das principais características do projeto é o roteamento entre conhecimento geral e conhecimento corporativo.

O classificador recebe a pergunta e retorna uma das categorias:

```text
GERAL
```

ou

```text
CORPORATIVO
```

### Pergunta geral

Exemplo:

```text
O que é uma API REST?
```

Nesse caso, a aplicação utiliza o conhecimento geral do modelo.

### Pergunta corporativa

Exemplo:

```text
Como funciona o onboarding da Pegasus?
```

Nesse caso, a aplicação realiza uma busca na base documental antes de gerar a resposta.

Essa separação evita consultar a base corporativa desnecessariamente em perguntas conceituais.

---

## Controle de informações corporativas

O sistema possui regras específicas para evitar que informações internas sejam expostas diretamente.

Entre os controles implementados estão:

* Bloqueio de solicitações para obter documentos completos;
* Bloqueio de solicitações para revelar o prompt do sistema;
* Bloqueio de solicitações para expor caminhos internos;
* Bloqueio de solicitações para revelar conteúdo bruto do ChromaDB;
* Não exposição dos embeddings;
* Não disponibilização direta dos documentos ao usuário;
* Respostas baseadas em síntese do conteúdo recuperado;
* Tratamento explícito quando uma informação corporativa não é encontrada.

Por exemplo, a aplicação não deve responder a uma solicitação para simplesmente disponibilizar um PDF interno inteiro.

Em vez disso, orienta o usuário a fazer uma pergunta específica sobre o conteúdo.

> Esses mecanismos são controles aplicados dentro da aplicação e não substituem autenticação, autorização, auditoria, criptografia e gerenciamento adequado de segredos em um ambiente corporativo real.

---

## Validação das entradas

A API também possui validações para as perguntas recebidas.

A estrutura utilizada pelo FastAPI limita a pergunta a:

* mínimo de 1 caractere;
* máximo de 1000 caracteres.

Além disso, o conteúdo é tratado antes de ser encaminhado para o fluxo de geração.

Caso o serviço de IA ou a base vetorial estejam indisponíveis, a API retorna códigos HTTP apropriados, incluindo:

```text
400 — entrada inválida
503 — serviço/base indisponível
500 — erro interno
```

---

## API

A aplicação disponibiliza os seguintes endpoints principais:

### `GET /`

Retorna a interface web da aplicação.

### `POST /api/perguntar`

Recebe uma pergunta e retorna a resposta gerada.

Exemplo de requisição:

```json
{
  "pergunta": "O que é RAG?"
}
```

A resposta pode indicar, entre outras informações:

```json
{
  "sucesso": true,
  "pergunta": "O que é RAG?",
  "resposta": "...",
  "fontes": [],
  "modo": "geral"
}
```

Quando uma base corporativa é utilizada, o campo `modo` pode indicar:

```text
rag
```

### `GET /api/status`

Retorna informações sobre o estado da aplicação, incluindo:

* configuração da Groq;
* disponibilidade da base vetorial;
* quantidade de documentos/trechos indexados;
* modelo de LLM utilizado;
* banco vetorial utilizado;
* status do agente.

---

## Ingestão dos documentos

A indexação é realizada pelo arquivo:

```text
ingest.py
```

O processo suporta atualmente:

```text
PDF
Markdown
TXT
CSV
JSON
```

Os documentos são encontrados recursivamente dentro da pasta:

```text
documentos/
```

Para arquivos PDF, o sistema utiliza o `pypdf` para extrair o texto.

Os conteúdos são então divididos em chunks utilizando:

```text
Tamanho aproximado: 900 caracteres
Sobreposição: 150 caracteres
```

Cada trecho recebe metadados como:

* documento;
* área;
* categoria;
* tipo de arquivo;
* índice do chunk;
* página, quando aplicável.

Esses metadados permitem identificar a origem dos trechos recuperados durante a consulta.

---

## Tecnologias utilizadas

| Tecnologia                      | Utilização                             |
| ------------------------------- | -------------------------------------- |
| **Python**                      | Linguagem principal                    |
| **FastAPI**                     | Desenvolvimento da API                 |
| **Uvicorn**                     | Servidor ASGI                          |
| **Groq**                        | Acesso ao modelo de linguagem          |
| **ChromaDB**                    | Banco de dados vetorial                |
| **Sentence Transformers**       | Geração de embeddings                  |
| **pypdf**                       | Extração de conteúdo de PDFs           |
| **python-dotenv**               | Gerenciamento de variáveis de ambiente |
| **HTML**                        | Estrutura da interface                 |
| **CSS**                         | Estilização                            |
| **JavaScript**                  | Interações da interface                |
| **Docker**                      | Containerização                        |
| **Oracle Cloud Infrastructure** | Infraestrutura de deploy               |

---

## Estrutura do projeto

```text
Lamarcks-IA/
│
├── documentos/
│   ├── arquitetura/
│   ├── backend/
│   ├── frontend/
│   ├── incidentes/
│   └── onboarding/
│
├── static/
│   ├── script.js
│   └── style.css
│
├── templates/
│   └── index.html
│
├── .env.example
├── .gitignore
├── Dockerfile
├── app.py
├── config.py
├── ingest.py
├── rag.py
├── requirements.txt
└── README.md
```

A pasta `chroma_db/` é criada localmente durante a indexação e está incluída no `.gitignore`, portanto não faz parte do versionamento do repositório.

---

## Como executar

### 1. Clone o repositório

```bash
git clone https://github.com/Lamarcks/Lamarcks-IA.git
```

Entre na pasta:

```bash
cd Lamarcks-IA
```

### 2. Crie um ambiente virtual

```bash
python -m venv .venv
```

No Windows:

```powershell
.venv\Scripts\Activate.ps1
```

No Linux/macOS:

```bash
source .venv/bin/activate
```

### 3. Instale as dependências

```bash
python -m pip install -r requirements.txt
```

### 4. Configure as variáveis de ambiente

Crie um arquivo:

```text
.env
```

utilizando o `.env.example` como referência.

Configure sua chave:

```env
GROQ_API_KEY=sua_chave_aqui
```

Também é possível definir:

```env
GROQ_MODEL=llama-3.1-8b-instant
MAX_DISTANCE=0.45
```

> O arquivo `.env` não deve ser enviado para o GitHub.

### 5. Crie a base vetorial

Execute:

```bash
python ingest.py
```

Esse comando irá:

```text
Ler documentos
    ↓
Extrair conteúdo
    ↓
Criar chunks
    ↓
Gerar embeddings
    ↓
Criar coleção no ChromaDB
```

### 6. Inicie a aplicação

```bash
python -m uvicorn app:app --reload
```

Depois acesse:

```text
http://127.0.0.1:8000
```

---

## Executando com Docker

O projeto também possui um `Dockerfile` baseado em:

```text
Python 3.11-slim
```

Para criar a imagem:

```bash
docker build -t lamarcks-ia .
```

Para executar:

```bash
docker run -p 8000:8000 --env-file .env lamarcks-ia
```

A aplicação ficará disponível em:

```text
http://localhost:8000
```

A base vetorial e os documentos precisam ser disponibilizados adequadamente no ambiente do container para que o fluxo RAG funcione.

---

## Exemplos de perguntas

### Conhecimento geral

```text
O que é RAG?
```

```text
Qual a diferença entre Machine Learning e Deep Learning?
```

```text
Como funciona uma API REST?
```

### Conhecimento corporativo

```text
Como funciona o onboarding da Pegasus?
```

```text
Quais padrões de engenharia back-end são utilizados pela Pegasus?
```

```text
Como a Pegasus organiza sua arquitetura de microsserviços?
```

O sistema determina automaticamente qual fluxo utilizar.

---

## Deploy em Oracle Cloud

O projeto também foi utilizado em uma infraestrutura da **Oracle Cloud Infrastructure (OCI)**.

A arquitetura de execução pode ser representada por:

```text
Internet
   ↓
Oracle Cloud Infrastructure
   ↓
Compute
   ↓
Linux
   ↓
Aplicação Python
   ↓
FastAPI / Uvicorn
   ↓
LAMARCKS IA
```

A infraestrutura em nuvem fez parte do aprendizado do projeto, permitindo trabalhar não apenas com desenvolvimento da aplicação, mas também com aspectos relacionados a **deploy e execução de serviços em cloud**.

---

## Conceitos praticados

Durante o desenvolvimento foram trabalhados conceitos de:

* Inteligência Artificial Generativa;
* Large Language Models;
* RAG;
* Engenharia de Prompt;
* classificação de perguntas;
* embeddings;
* busca semântica;
* bancos de dados vetoriais;
* processamento de documentos;
* APIs REST;
* FastAPI;
* Python;
* validação de dados;
* tratamento de exceções;
* segurança de aplicações;
* proteção contra exposição de informações;
* Docker;
* Linux;
* Oracle Cloud Infrastructure;
* arquitetura de software.

---

## Aprendizados

Este projeto foi uma evolução importante em relação ao uso de IA apenas como uma interface de perguntas e respostas.

Durante o desenvolvimento, foi possível trabalhar com o fluxo completo de uma aplicação baseada em RAG:

```text
Documentos
    ↓
Processamento
    ↓
Embeddings
    ↓
Banco vetorial
    ↓
Recuperação
    ↓
Contexto
    ↓
LLM
    ↓
Resposta
```

Também foi possível compreender que uma aplicação com LLM precisa considerar aspectos além da geração de texto, como:

* qualidade do contexto recuperado;
* limites da informação disponível;
* validação das entradas;
* tratamento de falhas;
* proteção de informações internas;
* controle de credenciais;
* arquitetura da aplicação;
* execução em infraestrutura real.

---

## Próximos passos

Algumas evoluções possíveis para o projeto são:

* [ ] autenticação de usuários;
* [ ] controle de acesso por perfil;
* [ ] histórico de conversas;
* [ ] painel administrativo;
* [ ] monitoramento da aplicação;
* [ ] métricas de utilização do RAG;
* [ ] sistema de avaliação das respostas;
* [ ] suporte a mais formatos de documentos;
* [ ] cache de consultas frequentes;
* [ ] HTTPS e domínio próprio;
* [ ] pipeline de CI/CD;
* [ ] testes automatizados.

---

## Autor

**Ihago Lamarcks**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Ihago%20Lamarcks-0A66C2?style=for-the-badge\&logo=linkedin\&logoColor=white)](https://www.linkedin.com/in/ihago-lamarcks1/)

---

<div align="center">

**Python • FastAPI • RAG • ChromaDB • LLM • Oracle Cloud**

</div>
