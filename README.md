# PFC - Estudo Comparativo entre LLM (Gemini) e SLM (Phi-3)

Este repositório contém todo o código-fonte, notebooks de experimento e dados de teste utilizados no Trabalho de Conclusão de Curso (TCC) intitulado: **"UM ESTUDO DE CASO COMPARATIVO ENTRE LLM E SLM: DESEMPENHO, APLICABILIDADE E EFICIÊNCIA EM CONTEXTOS REAIS"**.

## 1. Sobre o Projeto

Este estudo compara um LLM de ponta (Google Gemini Flash, via API) com um SLM (Microsoft Phi-3-mini-4k-instruct, local) em duas tarefas:

1.  **Classificação de Documentos:** Classificar artigos do dataset AG News.
2.  **Extração Estruturada de JSON:** Extrair dados de textos usando um schema JSON complexo.

O objetivo era avaliar os *trade-offs* de acurácia, latência, consumo de VRAM (custo de hardware) e segurança/privacidade.

## 2. Resumo dos Resultados Reais

Ao contrário das hipóteses iniciais, os experimentos conduzidos neste ambiente de teste (Google Colab, GPU T4) concluíram que o SLM **foi inferior ao LLM em todas as métricas de desempenho**:

* **Acurácia (Classificação):** O LLM (Gemini) foi mais preciso (81%) que o SLM treinado (68%).
* **Acurácia (Extração):** O SLM falhou em executar a tarefa. A combinação do texto de entrada e do schema JSON complexo excedeu a janela de contexto de 4k do modelo (`phi-3-mini-4k-instruct`), impedindo o treinamento e a inferência. O LLM executou a tarefa, embora com baixa acurácia (≈27%).
* **Eficiência (VRAM):** O SLM (Phi-3-mini) exigiu mais de **14 GB de VRAM** para o *fine-tuning* com QLoRA, refutando a tese de viabilidade em hardware de baixo custo (ex: < 8GB).
* **Eficiência (Latência):** A inferência do SLM no ambiente Colab (≈5600 ms) foi mais de 10 vezes mais lenta que a inferência via API do LLM (≈436 ms).
* **Conclusão Principal:** A única vantagem clara do SLM foi a **soberania dos dados** (execução *on-premise*), que vem ao custo de hardware mais caro e desempenho inferior em todas as outras métricas.

## 3. Estrutura do Repositório

* `/classificacao_documentos`: Contém todos os notebooks e dados para a Tarefa 1 (Classificação).
* `/extracao_dados`: Contém todos os notebooks e dados para a Tarefa 2 (Extração de JSON).
* `LICENSE`: Licença MIT, permitindo a reprodução e uso deste código.

## 4. Como Reproduzir os Experimentos

Todos os experimentos foram executados no Google Colab com uma GPU T4.

### Pré-requisitos

1.  **Conta Google:** Para usar o Google Colab e o Google Drive.
2.  **API Key do Google AI:** Necessária para rodar os notebooks do Gemini.
3.  **Token do Hugging Face:** Necessário para baixar os modelos (Phi-3) do Hugging Face Hub.
4.  **Montar o Google Drive:** Todos os notebooks são configurados para salvar e carregar arquivos (como os adaptadores de modelo) do Google Drive. Você deve permitir a montagem do Drive (`drive.mount('/content/drive')`) em todos eles.

### Configuração de Chaves (Secrets)

Para rodar os notebooks, você precisará adicionar as seguintes chaves ao Gerenciador de *Secrets* do Google Colab (ícone de **Chave {x}** no menu):

1.  **Chave do Google AI (para o Gemini):**
    * **Notebooks:** Todos os notebooks `Gemini 2.0 Flash...`.
    * **Nome do Secret:** `GEMINI_API_KEY`
    * **Valor:** Cole sua chave de API do Google AI.

2.  **Token do Hugging Face (para o Phi-3):**
    * **Notebooks:** Todos os notebooks `Phi-3-mini...`.
    * **Nome do Secret:** `HF_TOKEN`
    * **Valor:** Cole seu *token* de acesso (com permissão de leitura) do Hugging Face. Isso é necessário para autenticar e baixar o modelo.

### Ordem de Execução dos Notebooks

Para reproduzir os resultados, siga esta ordem:

#### Tarefa 1: Classificação de Documentos

1.  **`classificacao_documentos/Script - Generate Training Jsonl file...ipynb`**
    * **O que faz:** Baixa o dataset AG News e cria o arquivo `train_ag_news.jsonl` (1.000 amostras) usado para o *fine-tuning*.
2.  **`classificacao_documentos/Train Phi-3-mini - Document Classification...ipynb`**
    * **O que faz:** Treina o adaptador QLoRA no Phi-3-mini usando o arquivo `.jsonl` acima.
    * **IMPORTANTE:** Este notebook salva o adaptador treinado no seu Google Drive (ex: `/content/drive/MyDrive/my-phi3-agnews-adapter`).
3.  **`classificacao_documentos/UNTUNNED Phi-3-mini...ipynb`**
    * **O que faz:** Roda a avaliação de acurácia, latência e VRAM do modelo *base* (sem *fine-tuning*).
4.  **`classificacao_documentos/Phi-3-mini Tunned...ipynb`**
    * **O que faz:** Roda a avaliação no modelo *com* o adaptador QLoRA.
    * **ATENÇÃO:** Você deve atualizar a variável `adapter_path` neste notebook para apontar para o local onde o Passo 2 salvou seu adaptador no Google Drive.
5.  **`classificacao_documentos/Gemini 2.0 Flash...ipynb`**
    * **O que faz:** Roda a avaliação de acurácia e latência do Gemini (Zero-Shot e Few-Shot).

#### Tarefa 2: Extração de Dados (JSON)

1.  **`extracao_dados/Script - Generate train and test jsonl file...ipynb`**
    * **O que faz:** Baixa o dataset `paraloq/json_data_extraction` e cria os arquivos `train_json_extraction.jsonl` (400 amostras) e `test_json_extraction.jsonl` (40 amostras).
2.  **`extracao_dados/Train Phi-3-Mini - Data Extraction...ipynb`**
    * **O que faz:** Tenta treinar o adaptador QLoRA.
    * **Resultado Esperado:** Este notebook **deve falhar** (travar ou dar erro de "Out of Memory" / "Context Length"), como documentado na tese. Isso comprova a limitação do modelo Phi-3-mini-4k para esta tarefa.
3.  **`extracao_dados/UNTUNNED Phi-3-mini...ipynb`**
    * **O que faz:** Tenta rodar a inferência no modelo base.
    * **Resultado Esperado:** Este notebook também **deve falhar** (não retornar resposta ou travar), pelo mesmo motivo de limite de contexto.
4.  **`extracao_dados/Gemini 2.0 Flash...ipynb`**
    * **O que faz:** Roda a avaliação de acurácia e latência do Gemini (Zero-Shot e Few-Shot), que é o único modelo que consegue executar a tarefa.

## 5. Licença

Este projeto é licenciado sob a Licença MIT. Veja o arquivo `LICENSE` para mais detalhes.
