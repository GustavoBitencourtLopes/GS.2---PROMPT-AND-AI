# Citizen Assist AI — Assistente de Orientação sobre Documentos Pessoais

**Global Solution 2026.2 — Prompt and Artificial Intelligence — FIAP**

## Integrantes

- Gustavo Bitencourt Lopes — RM 568885
- Daniel Vieira — RM 573326

> Protótipo educacional. As informações vêm de fichas **fictícias** da Central do Cidadão de Aurora e **não** constituem um canal oficial.

---

## Sobre o projeto

O Citizen Assist AI responde dúvidas sobre documentos pessoais (RG, CPF, título de eleitor, carteira de trabalho, certidões e renovação da CNH) usando **somente** as 7 fichas de referência da Central do Cidadão de Aurora. Toda resposta cita a ficha de origem e a data de verificação, e traz o aviso de que o assistente não é um canal oficial.

Se a informação não está nas fichas, o assistente não inventa: diz que não encontrou e orienta a procurar o órgão competente.

**Notebook executado no Kaggle:** https://www.kaggle.com/code/gustavobiten/gs-2-prompt-and-ai

## Estrutura do repositório

```
├── README.md
├── gs-2-prompt-and-ai.ipynb      ← notebook executado (entrega principal)
├── requirements.txt
├── fichas/                       ← as 7 fichas de referência + LEIA-ME
└── docs/
    ├── grafo.png                 ← arquitetura do agente (LangGraph)
    └── comparacao_versoes.png    ← acerto por tipo de mensagem: v1 x v2 x v3
```

## Como rodar

O notebook foi feito para o **Kaggle**:

1. Crie um dataset no Kaggle com os arquivos da pasta `fichas/` e adicione-o ao notebook em **Add Input**.
2. Em **Settings**, ligue **Internet: On** (Accelerator pode ficar em None).
3. Crie uma chave gratuita em [console.groq.com](https://console.groq.com) e cadastre em **Add-ons → Secrets** com o nome `GROQ_API_KEY`.
4. Use **Run All**. A interface sobe no Bloco 6, com um link público `gradio.live`.

Os resultados das avaliações dependem de modelos de linguagem e podem variar um pouco entre execuções. As análises do notebook se referem às saídas salvas nele.

## Arquitetura

![Grafo do agente](docs/grafo.png)

Um único agente, construído com **LangGraph**. A IA só entra em um nó (o atendente, que redige a resposta); todos os outros são lógica e algoritmos:

| Nó | O que faz | Usa IA? |
|---|---|---|
| porteiro | valida a entrada, oculta dados pessoais e detecta prompt injection | Não |
| arquivista | escolhe as fichas relevantes (busca por palavras-chave com peso) | Não |
| roteador | decide o caminho (resposta fixa ou atendente) | Não |
| resposta_fixa | entrada vazia, longa ou sem sentido; fora do escopo; injeção; perguntas sobre o assistente | Não |
| atendente | redige a resposta com base apenas nas fichas | **Sim** |
| supervisor | oculta dados na saída e coloca a citação (ficha e data) e os avisos | Não |

**Modelos (via Groq):** agente `qwen/qwen3.8-27b`; juiz `openai/gpt-oss-120b`, de outra família, para reduzir o viés de autoavaliação.

## Organização das fichas

As fichas são lidas dos arquivos `.md` e convertidas em uma estrutura de dados (id, título, palavras-chave, campos, data de verificação e texto). A busca usa **peso por raridade** (TF-IDF simplificado): palavras que aparecem em várias fichas valem menos. A FC-07 (regras gerais) veio sem palavras-chave; criamos palavras de busca para ela, sem alterar seu conteúdo. A busca acertou as fichas esperadas nas 26 mensagens do golden dataset.

## Privacidade

- CPF, RG, telefone, e-mail, números longos e nomes são **ocultados antes** de qualquer processamento; o texto original não é guardado nem enviado ao modelo.
- A interface não guarda histórico nem registra conversas.
- Nos testes, os participantes foram identificados apenas como P1 a P5.

## LLM as a Judge: resultados no golden dataset

O juiz avalia três critérios binários: conteúdo, citação da ficha e segurança (dados pessoais e manipulação). Também há checagens determinísticas e conferência manual de uma amostra.

| Versão | O que mudou | Acerto (juiz 2) |
|---|---|---|
| v1 | linha de base (só prompt) | 22/26 |
| v2 | regras explícitas, rotas fixas, citação e avisos colocados pelo código | 23/26 |
| **v3** | regra de resposta "curta **e** completa" + correção do porteiro | **25/26** |

O único erro da v3 (#1) é do juiz: ele classificou como invenção informações de pagamento que estão na FC-07. Em todas as versões, nenhum dado pessoal vazou, a data foi citada em 19/19 respostas de escopo e o aviso apareceu em 26/26.

![Comparação entre versões](docs/comparacao_versoes.png)

## Testes com usuários

5 pessoas de fora do grupo, 3 tarefas e questionário de 5 perguntas (1 a 5):

| Indicador | Resultado |
|---|---|
| Tempo médio | T1 (documentos do RG) 45,6 s · T2 (certidão pelo portal) 27,2 s · T3 (sábados) 20,6 s |
| Sucesso nas tarefas | 12/15 (80%) |
| Facilidade de uso | 4,04 / 5 |

Principais aprendizados: falta um aviso de "carregando" quando o modelo demora, e respostas completas demais dificultam a leitura rápida.

## Limitações

- Só responde sobre as 7 fichas fictícias; não consulta sistemas, não agenda e não verifica situação cadastral.
- A busca por palavras-chave não entende sinônimos fora da lista nem perguntas que não citam o documento.
- A detecção de dados pessoais e de prompt injection por padrões não cobre todos os casos.
- O juiz também é um LLM e pode errar.
- A interface não mostra claramente quando está processando.

A análise completa, as justificativas e as limitações estão nos Blocos 8, 9 e 10 do notebook.

## Tecnologias

Python · LangGraph · LangChain (Groq) · Gradio · pandas · Kaggle
