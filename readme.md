# Sistema de Recuperação de Informações com FAISS e RetrievalQA

Projeto em Python que implementa um sistema de **Recuperação Aumentada por Geração (RAG)** para responder perguntas a partir do conteúdo de um documento PDF. O sistema usa **LangChain**, **Ollama**, **FAISS** e **RetrievalQA**.

## Objetivo

Construir um pipeline que:

1. Carregue um documento PDF e extraia seu texto.
2. Divida o conteúdo em trechos (*chunks*).
3. Gere *embeddings* para os trechos.
4. Armazene os vetores em um índice FAISS.
5. Receba perguntas, recupere trechos relevantes e produza respostas com um modelo de linguagem local.

## Tecnologias

- **Python 3.12** — linguagem utilizada no projeto.
- **Jupyter Notebook** — ambiente de execução do arquivo `.ipynb`.
- **LangChain** — organização das etapas do pipeline.
- **PyPDFLoader** (`langchain-community`) — leitura do PDF.
- **RecursiveCharacterTextSplitter** (`langchain-text-splitters`) — divisão do texto em chunks.
- **Ollama** — execução local dos modelos.
- **nomic-embed-text** — geração de embeddings.
- **FAISS** (`faiss-cpu`) — índice vetorial e busca por similaridade.
- **llama3.2:1b** — geração de respostas.
- **RetrievalQA** (`langchain-classic`) — integração da recuperação de trechos com o modelo de linguagem.

## Estrutura do projeto

```text
projeto-rag/
├── documents/
│   └── inteligencia_artificial.pdf
├── rag.ipynb
├── requirements.txt
└── README.md
```

> O notebook mais recente foi enviado como `rag(1).ipynb`. Você pode manter esse nome ou renomeá-lo para `rag.ipynb` ao organizar o repositório. O PDF deve estar em `documents/inteligencia_artificial.pdf`, relativo ao diretório de execução do notebook.

## Pré-requisitos

- Python 3.12 instalado.
- VS Code com as extensões **Python** e **Jupyter**, ou outro ambiente compatível com notebooks.
- Ollama instalado e disponível no computador.

### 1. Instalar as dependências Python

O arquivo `requirements.txt` contém as dependências Python utilizadas no projeto:

```txt
langchain
langchain-classic
langchain-community
langchain-ollama
langchain-text-splitters
faiss-cpu
jupyter
ipykernel
pypdf
```

Na pasta do projeto, execute no terminal do VS Code:

```powershell
py -3.12 -m pip install -r requirements.txt
```

O `requirements.txt` instala as bibliotecas Python, mas **não instala o Ollama nem baixa seus modelos**. Caso apareçam avisos relacionados à interpretação de fontes do PDF, é possível instalar adicionalmente `fonttools` com `py -3.12 -m pip install fonttools`.

### 2. Baixar os modelos do Ollama

```powershell
ollama pull nomic-embed-text
ollama pull llama3.2:1b
```

Certifique-se de que o Ollama esteja em execução antes de executar as células que geram embeddings ou respostas.

### 3. Abrir e executar o notebook

1. Abra a pasta do projeto no VS Code.
2. Confirme que o PDF está no caminho `documents/inteligencia_artificial.pdf`.
3. Abra `rag.ipynb` (ou `rag.ipynb`).
4. Selecione o kernel do Python 3.12.
5. Execute as células em ordem, de cima para baixo.
6. Na última célula, digite uma pergunta sobre o documento quando solicitado.

## Como o sistema funciona

```text
PDF
  ↓
PyPDFLoader (extração do texto)
  ↓
RecursiveCharacterTextSplitter (chunks)
  ↓
OllamaEmbeddings: nomic-embed-text
  ↓
FAISS (índice vetorial)
  ↓
Pergunta do usuário → busca de trechos relevantes
  ↓
RetrievalQA + ChatOllama: llama3.2:1b
  ↓
Resposta
```

### Divisão em chunks

O notebook utiliza:

```python
divisor = RecursiveCharacterTextSplitter(
    chunk_size=1500,
    chunk_overlap=200
)

trechos = divisor.split_documents(documentos)
```

Cada trecho tem como referência até 1.500 caracteres, com sobreposição de 200 caracteres entre trechos consecutivos, quando aplicável. Essa sobreposição ajuda a preservar o contexto entre divisões.

### Criação do índice vetorial

```python
embeddings = OllamaEmbeddings(model="nomic-embed-text")

banco_vetorial = FAISS.from_documents(
    documents=trechos,
    embedding=embeddings
)
```

O índice FAISS é criado **em memória** durante a execução do notebook. A versão atual não salva nem recarrega o índice do disco; ao reiniciar o kernel, é necessário executar novamente as células de preparação.

### Busca por similaridade

O notebook demonstra uma consulta direta ao FAISS:

```python
pergunta = "Do que se trata o documento?"
resultados = banco_vetorial.similarity_search(pergunta, k=3)
```

O parâmetro `k=3` indica a recuperação de três trechos mais próximos da pergunta segundo a busca vetorial.

### Configuração do RetrievalQA

```python
modelo = ChatOllama(model="llama3.2:1b", temperature=0)

recuperador = banco_vetorial.as_retriever(
    search_kwargs={"k": 3}
)

qa = RetrievalQA.from_chain_type(
    llm=modelo,
    chain_type="stuff",
    retriever=recuperador,
)
```

O `RetrievalQA` recupera os trechos relacionados à pergunta e os envia como contexto ao modelo de linguagem, que gera uma resposta.

### Perguntas interativas

A última célula do notebook permite inserir uma pergunta:

```python
pergunta = input("Digite sua pergunta sobre o documento: ")

resultado = qa.invoke({"query": pergunta})

print("\nRESPOSTA:")
print(resultado["result"])
```

Para fazer outra pergunta, basta executar novamente essa última célula, desde que o kernel permaneça ativo e as etapas anteriores já tenham sido executadas.

## Limitações e observações

- A qualidade das respostas depende do conteúdo extraído do PDF e dos trechos recuperados.
- O modelo pode produzir respostas incorretas; é recomendável conferir informações importantes no documento original.
- PDFs digitalizados sem texto selecionável podem exigir OCR, não implementado neste notebook.
- O código utiliza `RetrievalQA`, disponível em `langchain-classic`; trata-se de uma API legada mantida aqui por corresponder ao requisito da atividade.
- A versão atual imprime a resposta, mas não apresenta automaticamente as páginas usadas como fonte.

## Relação com os requisitos da atividade

| Requisito | Implementação |
| --- | --- |
| Preparação do ambiente | Python, LangChain, FAISS e dependências |
| Escolha de textos | `documents/inteligencia_artificial.pdf` |
| Divisão em chunks | `RecursiveCharacterTextSplitter` |
| Geração de embeddings | `OllamaEmbeddings` com `nomic-embed-text` |
| Banco de dados vetorial | FAISS |
| Recuperação por similaridade | `similarity_search` e `as_retriever` |
| Integração com RetrievalQA | `RetrievalQA.from_chain_type` |
| Pergunta do usuário | `input()` |
| Resposta com base no contexto recuperado | `qa.invoke()` e `ChatOllama` |

## Licença

Nenhuma licença foi definida neste README. Caso o projeto seja publicado, escolha uma licença apropriada e verifique se o PDF utilizado pode ser redistribuído.
