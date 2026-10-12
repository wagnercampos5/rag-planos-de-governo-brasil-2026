# 🏛️ RAG & Agentes Autônomos: Análise de Planos de Governo (Eleições 2026)

## 📌 Sobre o Projeto

Este projeto demonstra a construção de um **Agente de Inteligência Artificial Autônomo com Memória**, utilizando a técnica de **RAG (Retrieval-Augmented Generation)** estruturada em grafos de estado. O agente foi projetado para atuar como um analista político imparcial, lendo, consultando e comparando dados factuais dos planos de governo dos candidatos Luiz Inácio Lula da Silva e Flávio Bolsonaro para as Eleições Presidenciais de 2026.

### 🎯 Motivação

A maior limitação atual dos Grandes Modelos de Linguagem (LLMs) em cenários corporativos e políticos é a **alucinação** e o viés de conhecimento prévio. 
Para resolver esse problema, a arquitetura deste projeto restringe o escopo de raciocínio da IA estritamente aos documentos oficiais protocolados no TSE. Em vez de depender do treinamento base, o agente busca ativamente o contexto necessário nos PDFs antes de formular qualquer resposta, garantindo precisão documental e rastreabilidade da informação.

---

## ⚙️ Arquitetura da Solução

O projeto abandona as cadeias sequenciais tradicionais em favor de uma orquestração cíclica orientada a estados (State Graphs).

1. **Ingestão e Processamento Documental:**
   * Extração textual de múltiplas páginas utilizando `PyPDFLoader`.
   * Conversão do texto para representações matemáticas (vetores) através do modelo de alta precisão `mixedbread-ai/mxbai-embed-large-v1` (Hugging Face).
   * Armazenamento temporal em banco vetorial de alta performance (`InMemoryVectorStore`).

2. **Criação de Ferramentas (Tools):**
   * Encapsulamento dos recuperadores de contexto em ferramentas individuais (`@tool`) atribuídas a cada candidato:
     * `consulta_plano_lula`
     * `consulta_plano_flavio`

3. **Orquestração com LangGraph e ReAct:**
   * Utilização do framework **LangGraph** para criar um grafo de decisões.
   * O modelo (`openai/gpt-oss-120b` servido pela LPU da **Groq** para inferência de baixíssima latência) atua como um agente **ReAct** (Reason + Act). Ele recebe a pergunta, "pensa" qual candidato consultar, aciona a respectiva ferramenta e analisa o retorno.
   * **Gerenciamento de Estado:** A classe `MemorySaver` injeta um `thread_id` na sessão, permitindo que o modelo se lembre do contexto da conversa ao longo de múltiplos prompts.

---

## 🚀 Como Executar o Projeto Localmente

### Pré-requisitos

* Python 3.10 ou superior.
* Uma chave de API da [Groq](https://console.groq.com/).
* (Opcional, porém recomendado) Um token de acesso do [Hugging Face](https://huggingface.co/settings/tokens).

### Instalação

1. Clone o repositório:
```bash
git clone https://github.com/SEU_USUARIO/rag-agente-eleicoes-2026.git
cd rag-agente-eleicoes-2026
```

2. Crie um ambiente virtual e instale as dependências:
```bash
python -m venv venv
source venv/bin/activate  # No Windows: venv\Scripts\activate
pip install -r requirements.txt
```

*(Nota: Certifique-se de que os pacotes `langchain`, `langchain-community`, `langchain-groq`, `langchain-huggingface`, `langgraph`, `pypdf` e `python-dotenv` estejam no seu `requirements.txt`).*

3. Configuração de Variáveis de Ambiente:

   Crie um arquivo `.env` na raiz do projeto e insira as suas credenciais:
```env
GROQ_API_KEY=sua_chave_da_groq_aqui
HF_TOKEN=seu_token_do_huggingface_aqui
```

4. Documentos da Base de Dados:

   Certifique-se de que os arquivos `plano_lula.pdf` e `plano_flavio.pdf` estão localizados no mesmo diretório de execução do notebook.

5. Inicie o Jupyter:
```bash
jupyter notebook
```
E abra o arquivo `Curso_Flash_Skills_RAG_e_Agentes_de_IA.ipynb`.

---

## 💻 Exemplo de Uso (Showcase)

Após inicializar a função `chat_com_memoria`, você pode realizar consultas factuais ou exigir comparações complexas. A IA fará o roteamento das ferramentas automaticamente.

### 1. Consulta Direcionada
```python
chat_com_memoria(
    mensagem_usuario="Quais são as principais propostas do Lula para a área de educação?",
    thread_id="sessao_eleicoes_1",
    verbose=False
)
```

### 2. Continuidade Conversacional (Memória Ativa)
Mantendo o mesmo `thread_id`, o agente reconhecerá o histórico da conversa:
```python
chat_com_memoria(
    mensagem_usuario="E em relação ao plano do Flávio Bolsonaro sobre esse mesmo assunto?",
    thread_id="sessao_eleicoes_1",
    verbose=False
)
```

### 3. Análise Comparativa (Multi-Document Retrieval)
O modelo identificará a necessidade de consultar as duas fontes para gerar um quadro comparativo consolidado:
```python
chat_com_memoria(
    mensagem_usuario="Compare o que os planos de Lula e Flávio propõem para a segurança pública e polícias.",
    thread_id="sessao_comparativa_2",
    verbose=True  # Exibe o raciocínio e a chamada das ferramentas
)
```

---

## 🤝 Considerações Finais

Este projeto ilustra a flexibilidade do LangChain e do LangGraph para cenários profissionais complexos, como auditoria documental, due diligence automatizada e suporte corporativo com estrita checagem factual.

*Desenvolvido como projeto prático após a formação [Flash Skills: RAG e Agentes de IA](https://github.com/alura-cursos/curso-flash-rag) da Alura & Oracle Next Education.*