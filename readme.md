# Sistema de Recuperação de Informações com FAISS e RetrievalQA

Projeto acadêmico desenvolvido em Python para recuperar informações de um documento PDF e responder perguntas com base nos trechos mais relevantes do texto. A solução utiliza **LangChain**, **Ollama**, **FAISS** e o módulo **RetrievalQA**, seguindo a abordagem de *Retrieval-Augmented Generation* (RAG).

## Objetivo

Construir um sistema que:

1. Carregue o conteúdo de um documento PDF.
2. Divida o texto em trechos menores (*chunks*).
3. Gere representações vetoriais (*embeddings*) desses trechos.
4. Armazene os vetores em um índice FAISS.
5. Receba uma pergunta, recupere trechos semanticamente semelhantes e produza uma resposta utilizando um modelo de linguagem local.

## Tecnologias utilizadas

- **Python 3.12** — linguagem de programação.
- **Jupyter Notebook** — ambiente de execução e experimentação do projeto.
- **LangChain** — integração das etapas de leitura, recuperação e geração de respostas.
- **PyPDFLoader** — extração de texto do PDF.
- **RecursiveCharacterTextSplitter** — divisão do texto em chunks.
- **Ollama** — execução local dos modelos de embeddings e de linguagem.
- **nomic-embed-text** — modelo responsável pela geração de embeddings.
- **FAISS** — índice vetorial para busca por similaridade.
- **llama3.2:1b** — modelo de linguagem utilizado para gerar as respostas.
- **RetrievalQA** (`langchain-classic`) — cadeia que conecta a recuperação de trechos à geração de respostas.

> **Observação:** `RetrievalQA` é uma API legada do LangChain, mas foi mantida porque seu uso é um requisito explícito da atividade.

## Estrutura do projeto

```text
sistema-rag/
├── documents/
│   └── mestrado_ufrn.pdf
├── rag.ipynb
└── README.md
```

O arquivo `documents/mestrado_ufrn.pdf` é o documento de referência utilizado pelo notebook. O caminho é relativo ao diretório de trabalho em que o Jupyter está sendo executado.

## Pré-requisitos

- Python 3.12 instalado.
- VS Code com as extensões **Python** e **Jupyter**, ou outro ambiente compatível com notebooks.
- [Ollama](https://ollama.com/) instalado e disponível no computador.
- Arquivo PDF localizado em `documents/mestrado_ufrn.pdf`.

## Instalação

No terminal do VS Code, instale as dependências do Python:

```powershell
py -3.12 -m pip install langchain langchain-classic langchain-community langchain-ollama langchain-text-splitters faiss-cpu pypdf fonttools jupyter ipykernel
```

Baixe os modelos utilizados no projeto:

```powershell
ollama pull nomic-embed-text
ollama pull llama3.2:1b
```

Certifique-se de que o serviço do Ollama esteja em execução antes de utilizar o notebook.

## Como executar

1. Abra a pasta do projeto no VS Code.
2. Confirme que o PDF está em `documents/mestrado_ufrn.pdf`.
3. Abra o arquivo `rag.ipynb`.
4. Selecione um kernel Jupyter associado ao **Python 3.12** com as dependências instaladas.
5. Execute as células **em ordem**, da primeira até a última.
6. Observe a resposta gerada para a pergunta definida no notebook.

Atualmente, o exemplo de pergunta utilizado é:

```python
pergunta = "Qual é o assunto principal do documento?"
```

Para testar outra pergunta, altere o conteúdo da variável `pergunta` na última célula e execute-a novamente após configurar o pipeline.

## Funcionamento do sistema

### 1. Leitura do documento

O `PyPDFLoader` lê o PDF e cria objetos contendo o texto extraído de suas páginas.

### 2. Divisão em chunks

O texto é dividido com a seguinte configuração:

```python
divisor = RecursiveCharacterTextSplitter(
    chunk_size=1500,
    chunk_overlap=200
)
```

Cada trecho tem tamanho máximo configurado de aproximadamente **1.500 caracteres**, com **200 caracteres de sobreposição** entre trechos quando aplicável. A quantidade final de chunks depende do conteúdo do PDF.

### 3. Geração de embeddings

O modelo `nomic-embed-text`, executado pelo Ollama, converte os trechos em vetores numéricos que representam características semânticas do texto.

### 4. Criação do índice vetorial

Os trechos e seus embeddings são inseridos no FAISS:

```python
banco_vetorial = FAISS.from_documents(
    documents=trechos,
    embedding=embeddings
)
```

Neste projeto, o índice é criado **em memória** durante a execução do notebook; não há etapa de salvamento permanente em disco.

### 5. Busca por similaridade

O notebook demonstra uma busca direta pelos três trechos mais próximos semanticamente da pergunta:

```python
resultados = banco_vetorial.similarity_search(pergunta, k=3)
```

### 6. Resposta com RetrievalQA

O recuperador consulta o FAISS, e o `RetrievalQA` utiliza os trechos encontrados como contexto para o modelo `llama3.2:1b`:

```python
recuperador = banco_vetorial.as_retriever(
    search_kwargs={"k": 3}
)

qa = RetrievalQA.from_chain_type(
    llm=modelo,
    chain_type="stuff",
    retriever=recuperador,
    return_source_documents=True
)

resultado = qa.invoke({"query": pergunta})
print(resultado["result"])
```

A conversão da pergunta em embedding ocorre internamente durante a recuperação vetorial.

## Fluxo resumido

```text
PDF
  ↓
Extração de texto (PyPDFLoader)
  ↓
Divisão em chunks (RecursiveCharacterTextSplitter)
  ↓
Geração de embeddings (nomic-embed-text)
  ↓
Indexação no FAISS
  ↓
Pergunta do usuário
  ↓
Busca dos 3 trechos mais semelhantes
  ↓
RetrievalQA + llama3.2:1b
  ↓
Resposta gerada
```

## Limitações

- A qualidade das respostas depende do texto extraído, da relevância dos trechos recuperados e do modelo de linguagem utilizado.
- PDFs digitalizados como imagem podem exigir OCR antes da indexação.
- O modelo pode produzir respostas incorretas ou informações não sustentadas pelo documento; recomenda-se conferir os trechos recuperados.
- O índice FAISS precisa ser reconstruído após reiniciar o kernel e executar novamente o notebook.
- O notebook utiliza uma pergunta definida no código, não uma interface interativa.

## Contexto acadêmico

O projeto atende às etapas propostas na atividade **“Construção de um Sistema de Recuperação de Informações usando VectorDB e RetrievalQA”**: preparação do ambiente, divisão de textos, geração de embeddings, criação de banco vetorial com FAISS e integração do pipeline de perguntas e respostas com RetrievalQA.
