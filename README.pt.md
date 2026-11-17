# Assistente de IA para Consulta de Dados

[English](README.md) · **Português**

Projeto Final (estágio) da **Licenciatura em Engenharia Informática** da ESTG – Politécnico do Porto, desenvolvido na **StoresAce** (março–julho 2025).

Assistente inteligente integrado na plataforma de gestão de retalho e restauração da StoresAce. O utilizador faz perguntas sobre os dados do negócio em linguagem natural ("Quantas lojas temos?", "Qual o artigo com maior custo médio?") e recebe uma resposta clara, sem precisar de saber onde estão os dados nem de escrever SQL.

![Assistente a responder em português](docs/chat-pt.png)

## Como funciona

```
Pergunta ─▶ Classificador de intenção BERT ─┬─▶ Fora de âmbito ─▶ resposta educada
                                            │
                                            └─▶ Consulta SQL / Dados
                                                  │
                                                  ▼
                                LLaMA (Amazon Bedrock) gera o SQL
                                                  │
                                                  ▼
                     Validação e limpeza (só leitura, tabelas qualificadas)
                                                  │
                                                  ▼
                                      Execução em PostgreSQL
                                                  │
                                                  ▼
                      LLaMA transforma o resultado numa resposta natural
```

1. **Classificação de intenção**: um modelo BERT afinado (Hugging Face) decide se a pergunta é uma consulta de dados ou se está fora de âmbito. Previsões com confiança abaixo de 0,75 são tratadas como fora de âmbito.
2. **Geração de SQL**: o LLaMA, servido pelo Amazon Bedrock, escreve uma query PostgreSQL a partir da pergunta, do schema da base de dados e do contexto da conversa.
3. **Validação**: o SQL gerado é limpo. Só são permitidos comandos de leitura, os comandos destrutivos são bloqueados e os nomes de tabelas e colunas são qualificados com o schema dinâmico (gerado pelo Django ORM), para prevenir SQL injection.
4. **Execução**: a query corre sobre um conjunto restrito de tabelas (vendas, stocks, artigos, lojas, preços, orçamentos, previsões…).
5. **Resposta natural**: o resultado é convertido numa resposta curta, no idioma escolhido pelo utilizador.

## Por dentro: dois modelos

**BERT: classificador de intenções.** Um modelo BERT pré-treinado, afinado (fine-tuning) com um conjunto de perguntas de exemplo criado por mim, para aprender a distinguir perguntas sobre os dados da StoresAce de tudo o resto. Como o BERT lida com vários idiomas, o mesmo classificador funciona seja qual for a língua da pergunta.

**LLaMA 3: SQL e respostas.** Tudo o que o LLaMA faz assenta em prompt engineering. O prompt de geração de SQL junta:

- **Variáveis**: a pergunta, o schema da base de dados, o contexto da conversa e `reports_SA`
- **Regras (guidelines)**: o que o modelo pode e não pode gerar
- **Ficheiros de apoio**: exemplos de pergunta → SQL

`reports_SA` são os reportes que a StoresAce já tinha, cada um com a descrição e a query SQL. O modelo analisa-os e pode reaproveitar queries já testadas, o que tornou as respostas bastante mais precisas.

Perguntas usadas na demonstração ao vivo:

- *Qual é o total de stock do artigo Eggs?*
- *Qual é o stock do artigo Bananas na loja The Breakfast Club – Marylebone?*
- *Qual o artigo com o maior sale_markup que foi vendido em cada loja no mês de janeiro?*

## Arquitetura

<p align="center"><img src="docs/architecture.png" alt="Arquitetura do sistema" width="480"></p>

| Camada | Tecnologias |
|---|---|
| Frontend | Vue.js, Vuetify, AJAX |
| Backend | Python, Django |
| IA / NLP | BERT (Hugging Face), LLaMA 3 via Amazon Bedrock, MiniLM |
| Dados | PostgreSQL, Pandas |
| Processo | Scrum (sprint reviews semanais, PSI diário), Git/GitHub |

## Funcionalidades

- 💬 Interface de chat para perguntas em linguagem natural
- 🧠 Classificação de intenções com BERT e deteção automática de perguntas fora de âmbito
- 🛢️ Text-to-SQL com LLaMA no Amazon Bedrock
- 🔒 Validação de SQL: só leitura, bloqueio de comandos destrutivos, prevenção de injection
- 🌍 **8 idiomas**: o assistente responde sempre no idioma selecionado, mesmo que a pergunta venha noutro
- 🗂️ Memória de curto prazo: as últimas 3 interações da sessão são mantidas como contexto
- 👍👎 Feedback Gostei/Não Gostei; os "não gostei" críticos notificam a equipa por e-mail
- 📋 Gestão de sessões com histórico e datas
- ⚡ Modelo BERT carregado uma única vez em memória (singleton)

| Respostas multilíngues | Deteção de fora de âmbito |
|---|---|
| ![Espanhol](docs/chat-es.png) | ![Fora de âmbito](docs/out-of-scope.png) |

## Endpoints

| Método | Endpoint | Parâmetros | Descrição |
|---|---|---|---|
| `GET` | `/chatbot/` | — | Página do assistente (utilizadores autenticados) |
| `POST` | `/chatbot/message/` | `question`, `session_id` (opcional) | Envia a pergunta e devolve a resposta, a intenção e o SQL gerado |
| `POST` | `/chatbot/report-dislike/` | `question`, `answer`, `sql` (opcional) | Feedback negativo; notifica a equipa por e-mail se for crítico |

## Conteúdo do repositório

| Ficheiro | Descrição |
|---|---|
| [`PF_ER_Relatorio_8220307.pdf`](PF_ER_Relatorio_8220307.pdf) | Relatório de estágio completo |
| [`PF_ER_Relatorio_8220307.docx`](PF_ER_Relatorio_8220307.docx) | Documento fonte do relatório |
| [`Apresentacao/PF_ER_Apresentacao_8220307.pdf`](Apresentacao/PF_ER_Apresentacao_8220307.pdf) | Apresentação feita ao júri (PDF) |
| [`Apresentacao/PF_ER_Apresentacao_8220307.pptx`](Apresentacao/PF_ER_Apresentacao_8220307.pptx) | Apresentação feita ao júri (PowerPoint) |
| [`Video/PF_ER_Video_8220307.mp4`](Video/PF_ER_Video_8220307.mp4) | Vídeo de demonstração |

> O código-fonte pertence à StoresAce e não está incluído. Este repositório documenta o trabalho através do relatório, da apresentação, dos diagramas e da demonstração.

## Trabalho futuro

- Cobrir as restantes tabelas da base de dados e responder a perguntas mais complexas
- Usar o assistente para apoiar a equipa de suporte, sugerindo respostas a partir de uma base de conhecimento da StoresAce
- Guardar o modelo de intenções num bucket (ou na base de dados) em vez de num ficheiro local, para uma solução mais escalável e cloud-native

## Autor

**Pedro Pinho**: [GitHub](https://github.com/PedroPinho17) · [LinkedIn](https://www.linkedin.com/in/pedropinho19/)

Orientadores: Prof. Doutor Marco Gomes (ESTG/P.Porto) · Eng. Ivo Marinho (StoresAce)
