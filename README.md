<div align="center">

# LAMARCKS IA

### Assistente de Inteligência Artificial com RAG Corporativo

**Inteligência Artificial • Engenharia de Software • Dados • Cloud**

Projeto desenvolvido durante o desafio final da **Alura + Oracle Next Education (ONE)**, com foco na construção de um assistente capaz de responder perguntas gerais e consultar uma base de conhecimento corporativa por meio de RAG.

<br>

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge\&logo=python\&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?style=for-the-badge\&logo=fastapi\&logoColor=white)
![Groq](https://img.shields.io/badge/Groq-LLM-F55036?style=for-the-badge)
![ChromaDB](https://img.shields.io/badge/ChromaDB-Vetor-5A67D8?style=for-the-badge)
![RAG](https://img.shields.io/badge/RAG-IA-6C63FF?style=for-the-badge)
![Oracle Cloud](https://img.shields.io/badge/Oracle_Cloud-OCI-F80000?style=for-the-badge\&logo=oracle\&logoColor=white)

<br>

### Acessar a aplicação

**[Abrir o Lamarcks IA](http://163.176.27.55:8000)**

<a href="http://163.176.27.55:8000">
<img width="1919" height="1079" alt="Preview da aplicação Lamarcks IA" src="https://github.com/user-attachments/assets/4316accc-0f97-4213-9602-04c5243ec032" />
</a>

</div>

---

## Sobre o projeto

O **Lamarcks IA** é um assistente de Inteligência Artificial desenvolvido para combinar dois tipos de atendimento:

* perguntas gerais sobre tecnologia e áreas relacionadas;
* perguntas específicas sobre uma base de conhecimento corporativa.

Para isso, o projeto utiliza uma arquitetura de **RAG (Retrieval-Augmented Generation)**, permitindo que informações relevantes sejam recuperadas de documentos antes de serem utilizadas na geração da resposta.

A aplicação foi construída com **Python e FastAPI**, utilizando **Groq** como provedor do modelo de linguagem, **Sentence Transformers** para geração de embeddings e **ChromaDB** como banco de dados vetorial.

---

## Objetivo

O objetivo do projeto é demonstrar, na prática, como construir uma aplicação de IA que vai além de simplesmente enviar uma pergunta para um modelo de linguagem.

O projeto trabalha conceitos como:

* integração com modelos de linguagem;
* RAG;
* embeddings;
* busca semântica;
* banco de dados vetorial;
* classificação de perguntas;
* processamento de documentos;
* API REST;
* validação de entradas;
* controle de informações;
* Docker;
* deploy em Cloud.

---

## Como funciona

O fluxo principal da aplicação pode ser resumido da seguinte forma:

```text
Usuário
   │
   ▼
Pergunta
   │
   ▼
FastAPI
   │
   ▼
Classificação da pergunta
   │
   ├───────────────┐
   │               │
   ▼               ▼
GERAL        CORPORATIVO
   │               │
   │               ▼
   │        Busca no ChromaDB
   │               │
   │               ▼
   │        Contexto relevante
   │               │
   └───────┬───────┘
           ▼
       Groq / LLM
           │
           ▼
        Resposta
```

A aplicação identifica primeiro se a pergunta pertence ao contexto corporativo ou se pode ser respondida utilizando conhecimento geral.

### Perguntas gerais

Quando a pergunta é classificada como geral, a aplicação utiliza o modelo de linguagem sem realizar uma busca na base corporativa.

Exemplos:

```text
O que é uma API REST?
```

```text
Qual a diferença entre Python e JavaScript?
```

```text
O que é Docker?
```

### Perguntas corporativas

Quando a pergunta está relacionada ao ambiente corporativo, a aplicação consulta a base vetorial.

O fluxo é:

1. receber a pergunta;
2. classificar como corporativa;
3. gerar o embedding da pergunta;
4. consultar o ChromaDB;
5. recuperar os trechos mais relevantes;
6. adicionar o contexto encontrado ao prompt;
7. enviar o contexto para o modelo;
8. gerar a resposta baseada nas informações recuperadas.

---

## RAG — Retrieval-Augmented Generation

O projeto utiliza **Retrieval-Augmented Generation** para permitir que o modelo consulte uma base externa de conhecimento.

Em vez de depender exclusivamente do conhecimento do modelo, a aplicação recupera informações relevantes dos documentos previamente processados.

### Pipeline

```text
Documentos
    │
    ▼
Extração do conteúdo
    │
    ▼
Divisão em chunks
    │
    ▼
Embeddings
    │
    ▼
ChromaDB
    │
    │
    ▼
Pergunta do usuário
    │
    ▼
Embedding da pergunta
    │
    ▼
Busca semântica
    │
    ▼
Trechos relevantes
    │
    ▼
LLM
    │
    ▼
Resposta
```

Os documentos podem ser processados a partir dos seguintes formatos:

* PDF
* Markdown
* TXT
* CSV
* JSON

---

## Busca semântica

Os documentos são divididos em pequenos trechos antes de serem transformados em embeddings.

Configurações utilizadas no projeto:

| Configuração                    |              Valor |
| ------------------------------- | -----------------: |
| Tamanho aproximado do chunk     |     900 caracteres |
| Sobreposição                    |     150 caracteres |
| Quantidade máxima de resultados |                  5 |
| Distância máxima configurada    |               0.45 |
| Modelo de embedding             | `all-MiniLM-L6-v2` |

Os embeddings são armazenados no **ChromaDB**, permitindo comparar semanticamente a pergunta do usuário com os conteúdos existentes na base.

---

## Roteamento de perguntas

Uma das características do projeto é o roteamento entre perguntas gerais e corporativas.

A classificação é realizada pelo próprio modelo de linguagem, que retorna uma das duas categorias:

```text
GERAL
```

ou

```text
CORPORATIVO
```

Essa separação evita realizar buscas desnecessárias na base corporativa quando a pergunta não depende dessas informações.

---

## Controle de informações corporativas

O projeto possui regras para evitar que informações internas sejam expostas diretamente.

A aplicação possui bloqueios para solicitações como:

* pedir documentos completos;
* copiar documentos internos;
* solicitar o prompt do sistema;
* solicitar caminhos internos;
* listar arquivos ou diretórios internos;
* acessar diretamente o banco vetorial;
* tentar ignorar as regras definidas para o agente.

Além disso, quando uma pergunta corporativa não possui contexto relevante na base, o sistema evita inventar uma resposta específica.

---

## Validação das entradas

As perguntas enviadas para a API passam por validações antes de serem processadas.

A aplicação possui limite de tamanho para as perguntas:

```text
Máximo: 1000 caracteres
```

Também são tratados erros relacionados à disponibilidade do serviço e falhas inesperadas durante o processamento.

Principais respostas HTTP utilizadas:

| Código | Situação                 |
| ------ | ------------------------ |
| `200`  | Pergunta processada      |
| `400`  | Entrada inválida         |
| `503`  | Serviço RAG indisponível |
| `500`  | Erro interno             |

---

## API

A aplicação utiliza **FastAPI** para disponibilizar os endpoints.

### Página inicial

```http
GET /
```

Retorna a interface da aplicação.

### Fazer uma pergunta

```http
POST /api/perguntar
```

Exemplo de requisição:

```json
{
  "pergunta": "O que é uma API REST?"
}
```

### Consultar status

```http
GET /api/status
```

O endpoint retorna informações relacionadas ao estado do agente, incluindo:

* configuração do Groq;
* disponibilidade da base vetorial;
* quantidade de documentos/chunks;
* modelo utilizado;
* banco vetorial.

---

## Ingestão dos documentos

O arquivo `ingest.py` é responsável por preparar os documentos utilizados pelo RAG.

O processo envolve:

1. localizar os documentos;
2. identificar o tipo de arquivo;
3. extrair o conteúdo;
4. dividir o conteúdo em chunks;
5. gerar embeddings;
6. armazenar os embeddings no ChromaDB.

O projeto utiliza uma coleção chamada:

```text
rede_vida_conhecimento
```

O banco vetorial é persistido localmente no diretório:

```text
chroma_db/
```

Esse diretório é ignorado pelo Git para evitar o versionamento dos dados gerados localmente.

---

## Tecnologias utilizadas

### Backend

* Python
* FastAPI
* Uvicorn
* Pydantic

### Inteligência Artificial

* Groq
* Large Language Model
* Sentence Transformers
* Embeddings
* RAG

### Banco vetorial

* ChromaDB

### Processamento de documentos

* PyPDF

### Infraestrutura

* Docker
* Oracle Cloud Infrastructure (OCI)

### Configuração

* Python-dotenv
* Variáveis de ambiente

---

## Estrutura do projeto

A estrutura principal do projeto é organizada da seguinte forma:

```text
Lamarcks-IA/
│
├── app.py
├── config.py
├── rag.py
├── ingest.py
├── requirements.txt
├── Dockerfile
├── .env.example
├── .gitignore
│
├── documentos/
│   └── ...
│
├── templates/
│   └── index.html
│
├── static/
│   └── ...
│
└── chroma_db/
    └── ...
```

> O diretório `chroma_db/` é criado localmente durante a utilização do sistema e não deve ser versionado.

---

## Configuração do ambiente

Crie um arquivo `.env` baseado no `.env.example`.

Exemplo:

```env
GROQ_API_KEY=sua_chave_groq
GROQ_MODEL=llama-3.1-8b-instant
MAX_DISTANCE=0.45
```

A chave da API deve permanecer apenas no ambiente local ou no serviço de hospedagem.

Ela **não deve ser adicionada ao GitHub**.

---

## Como executar localmente

### 1. Clonar o repositório

```bash
git clone https://github.com/Lamarcks/Lamarcks-IA.git
```

### 2. Entrar no diretório

```bash
cd Lamarcks-IA
```

### 3. Criar um ambiente virtual

Windows:

```bash
python -m venv .venv
```

Ativação:

```bash
.venv\Scripts\activate
```

Linux/macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Instalar as dependências

```bash
pip install -r requirements.txt
```

### 5. Configurar as variáveis de ambiente

Crie o arquivo:

```text
.env
```

e adicione sua chave da Groq.

### 6. Executar a aplicação

```bash
uvicorn app:app --reload
```

A aplicação ficará disponível localmente em:

```text
http://127.0.0.1:8000
```

---

## Executando com Docker

O projeto também possui um `Dockerfile` para facilitar a execução em ambientes compatíveis com containers.

Construção da imagem:

```bash
docker build -t lamarcks-ia .
```

Execução:

```bash
docker run -p 8000:8000 --env-file .env lamarcks-ia
```

Depois disso:

```text
http://localhost:8000
```

---

## Segurança e boas práticas

Durante o desenvolvimento, foram consideradas algumas práticas para reduzir riscos comuns em aplicações de IA:

* utilização de variáveis de ambiente para credenciais;
* `.env` ignorado pelo Git;
* limite de tamanho das perguntas;
* validação das entradas com Pydantic;
* tratamento de erros da API;
* bloqueio de solicitações para exposição de informações internas;
* separação entre conhecimento geral e corporativo;
* prevenção contra respostas inventadas sobre informações específicas da empresa;
* proteção do banco vetorial e dos arquivos internos contra acesso direto pela interface.

---

## Exemplos de perguntas

### Conhecimento geral

```text
O que é uma API?
```

```text
Como funciona o Docker?
```

```text
Qual a diferença entre banco SQL e NoSQL?
```

### Conhecimento corporativo

```text
Quais informações existem na base corporativa sobre determinado assunto?
```

```text
O que a documentação interna informa sobre determinado processo?
```

Nesse segundo caso, a aplicação realiza a busca semântica antes de gerar a resposta.

---

## Deploy em Cloud

O projeto também foi utilizado como experiência prática com **Oracle Cloud Infrastructure (OCI)**.

O ambiente de Cloud permitiu trabalhar conceitos relacionados a:

* infraestrutura;
* servidor;
* Linux;
* Docker;
* exposição de aplicações web;
* configuração de ambiente;
* execução de serviços Python em Cloud.

A experiência de deploy faz parte da evolução do projeto além do ambiente local.

---

## Principais conceitos praticados

Durante o desenvolvimento, foram trabalhados conceitos de diferentes áreas:

### Inteligência Artificial

* LLMs;
* engenharia de prompts;
* classificação de perguntas;
* geração de respostas;
* RAG;
* embeddings;
* busca semântica.

### Desenvolvimento de software

* APIs REST;
* FastAPI;
* validação de dados;
* tratamento de exceções;
* organização de código;
* variáveis de ambiente.

### Dados

* processamento de documentos;
* transformação de texto;
* criação de embeddings;
* armazenamento vetorial;
* recuperação de informações.

### Infraestrutura

* Docker;
* Linux;
* Oracle Cloud;
* execução de aplicações em servidor.

### Segurança

* proteção de credenciais;
* controle de informações internas;
* validação de entradas;
* prevenção contra exposição de dados;
* tratamento de solicitações maliciosas.

---

## Aprendizados

O desenvolvimento do Lamarcks IA permitiu aplicar conhecimentos de forma integrada, principalmente na conexão entre **Inteligência Artificial, desenvolvimento web, dados e infraestrutura**.

Um dos principais aprendizados foi entender que construir uma aplicação com IA envolve mais do que integrar um modelo de linguagem.

Também é necessário pensar em:

* origem das informações;
* qualidade do contexto recuperado;
* controle das respostas;
* segurança;
* tratamento de erros;
* organização da aplicação;
* infraestrutura;
* experiência do usuário.

O projeto também ajudou a compreender na prática como uma arquitetura RAG pode transformar documentos externos em uma fonte de conhecimento consultável por um assistente de IA.

---

## Próximos passos

Algumas possibilidades de evolução do projeto:

* autenticação de usuários;
* controle de acesso por perfil;
* histórico de conversas;
* painel administrativo;
* monitoramento da aplicação;
* testes automatizados;
* avaliação da qualidade das respostas do RAG;
* métricas de utilização;
* cache de respostas;
* melhoria do sistema de recuperação;
* HTTPS e domínio próprio;
* CI/CD;
* suporte a mais formatos de documentos.

---

## Autor

**Ihago Lamarcks**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Ihago_Lamarcks-0A66C2?style=for-the-badge\&logo=linkedin\&logoColor=white)](https://www.linkedin.com/in/ihagolamarcks/)

---

<div align="center">

**Lamarcks IA - Inteligência Artificial aplicada a conhecimento corporativo**

</div>
