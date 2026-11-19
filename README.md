# 🧑‍💻 Agente Entrevistador LLM

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Rubziguin/Agente-Entrevistador-LLM/blob/main/Simulador_Entrevistas_Agente_LLM.ipynb)
![Python](https://img.shields.io/badge/Python-3.10+-blue)
![OpenAI SDK](https://img.shields.io/badge/OpenAI_SDK-Chat_Completions-black)
![Gemini](https://img.shields.io/badge/LLM-Gemini-4285F4)

Mini agente de IA que simula uma **entrevista técnica de programação** no estilo das Big Techs. Ele apresenta desafios de lógica, conduz o candidato passo a passo com dicas graduais, **executa de verdade o código enviado contra casos de teste** e, no fim, gera uma **avaliação estruturada** com notas e feedback construtivo.

> Projeto da disciplina de **Inteligência Artificial** — Ciência da Computação, **PUC-SP**
> Atividade *Agente IA LLM* · Grupo D (Engenharia de Chat) · Projeto *Simulador de Entrevistas de Código*

---

## ✨ Funcionalidades

- **Memória de conversa:** a API do LLM não guarda estado, então o agente mantém o histórico e o reenvia a cada turno.
- **Tool calling:** o LLM decide quando chamar funções Python para sortear um desafio ou rodar os testes. O feedback se baseia em resultados reais, não em "achismo" do modelo.
- **Execução isolada:** o código do candidato roda num subprocesso separado (`python -I`, diretório temporário, timeout de 5 s).
- **Dicas graduais:** o agente nunca entrega a solução. Primeiro faz uma pergunta guiada, depois sugere uma estrutura de dados e só então mostra pseudocódigo parcial.
- **Saída estruturada:** a avaliação final é um JSON validado com **Pydantic** (notas de 0 a 10, pontos fortes, pontos a melhorar e veredito).
- **Visualização:** gráfico das notas com Seaborn.
- **Multi-provedor:** funciona com Gemini, OpenAI ou Hugging Face trocando só `base_url`, chave e modelo.
- **Resiliência:** em caso de sobrecarga do servidor (erro 503/429), tenta de novo automaticamente e usa modelos reserva.
- **Modo demo:** sem chave de API, roda com falas simuladas do entrevistador, mas os testes continuam sendo executados de verdade.

## 🧠 Arquitetura

```
 candidato ──► [histórico de mensagens] ──► LLM (OpenAI SDK · Chat Completions)
                       ▲                         │
                       │                ┌────────┴─────────┐
                       │          resposta texto     tool_calls
                       │                │                  │
                       └────────────────┘     ┌────────────▼────────────┐
                                              │ Ferramentas Python      │
                                              │  • sortear_desafio      │
                                              │  • executar_testes      │
                                              └────────────┬────────────┘
                                                           │ resultado (JSON)
                                     volta para o histórico ┘ → LLM decide o próximo passo
```

**Percepção** (mensagem do candidato + resultado dos testes) → **raciocínio** (LLM) → **ação** (ferramentas ou resposta) → **memória** (histórico).

| Ferramenta | O que faz |
|---|---|
| `sortear_desafio(nivel)` | Escolhe um desafio ainda não usado (fácil ou médio) e devolve o enunciado |
| `executar_testes(desafio_id)` | Roda o último código do candidato contra os casos de teste e devolve quais passaram |

**Desafios disponíveis:** Two Sum · Parênteses válidos · Palíndromo · Maior substring sem repetição · Mesclar intervalos

## 💬 Exemplo de interação

> **🧑 Candidato:** *(envia uma solução que devolve os valores em vez dos índices)*
>
> **🤖 Entrevistador:** Rodei os testes: **0/5 passaram**. Para a entrada `[2, 7, 11, 15], 9` era esperado `[0, 1]` e veio `[2, 7]`. Dica: releia o que a função deve **devolver**.
>
> **🧑 Candidato:** *(corrige usando um dicionário)*
>
> **🤖 Entrevistador:** Excelente, **5/5 testes passaram**! ✅ Qual a complexidade de tempo e de espaço da sua solução?

## 🚀 Como executar

1. Clique no botão **Open in Colab** acima.
2. Crie uma chave de API gratuita do Gemini em <https://aistudio.google.com/apikey>.
3. No Colab, abra o painel **🔑 Secrets**, adicione `GEMINI_API_KEY` com a sua chave e ative **Acesso ao notebook**.
4. Clique em **Ambiente de execução → Executar tudo**.
5. Para fazer a entrevista você mesmo, mude `RODAR_INTERATIVO = True` na seção 9 e rode a célula.

Provedores aceitos (basta **um**):

| Provedor | Secret | Modelo padrão |
|---|---|---|
| Google Gemini | `GEMINI_API_KEY` ou `GOOGLE_API_KEY` | `gemini-flash-latest` |
| OpenAI | `OPENAI_API_KEY` | `gpt-5-nano` |
| Hugging Face | `HF_KEY` | `meta-llama/Llama-3.3-70B-Instruct` |

> 🔒 Nunca coloque a chave direto no código. Os Secrets do Colab não vão para o GitHub.

## 🛠️ Tecnologias

Python · [OpenAI Python SDK](https://github.com/openai/openai-python) · Google Gemini (endpoint compatível com OpenAI) · Pydantic · Pandas · Matplotlib / Seaborn · Google Colab

## 📁 Estrutura

```
.
├── Simulador_Entrevistas_Agente_LLM.ipynb   # notebook do agente
└── README.md
```

## ⚠️ Limitações e próximos passos

- A execução do código usa subprocesso com timeout, mas **não é um sandbox completo**; em produção, usaríamos um container.
- O banco tem 5 desafios; próximos passos são adicionar nível difícil e **testes ocultos**.
- Resumir mensagens antigas para controlar o custo em tokens em entrevistas longas.
- Criar uma interface web com **Gradio**.

## 👥 Autores

| Nome | GitHub |
|---|---|
| Rubens Rodrigues Maranesi | [@Rubziguin](https://github.com/Rubziguin) |
| Pedro Gabriel Takenobu Serafim | [@pedrogts](https://github.com/pedrogts) |

## 📚 Referências

- [OpenAI — Function calling](https://platform.openai.com/docs/guides/function-calling)
- [Gemini — Compatibilidade com OpenAI](https://ai.google.dev/gemini-api/docs/openai)
- [Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers)
- [Pydantic](https://docs.pydantic.dev/)
- Russell & Norvig, *Artificial Intelligence: A Modern Approach*, 4ª ed.
