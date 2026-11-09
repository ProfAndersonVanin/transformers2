# Parte 4 — Montando um bloco Transformer completo

**Aula:** Como funciona um Transformer na prática (mecanismo de atenção)
**Público:** iniciantes em programação · **Duração sugerida:** 50 a 60 min
**Notebook:** `Parte4_Bloco_Transformer_Completo.ipynb`

---

## 🎯 Objetivos

- Entender por que o Transformer precisa da **codificação posicional**;
- Compreender a **atenção com múltiplas cabeças** (*multi-head*);
- Conhecer **conexão residual**, **normalização** e **feed-forward**;
- Entender a **máscara** e como um modelo **prevê a próxima palavra**;
- Usar um Transformer real em português (BERTimbau) — bônus.

---

## 1. O problema da ordem

| Frase | Notícia? |
|---|---|
| *cachorro mordeu homem* | Não 🐕 |
| *homem mordeu cachorro* | Sim! 📰 |

Executando a atenção da Parte 3 nas duas frases, o vetor de "cachorro" sai **idêntico**: `[0.39, 0.14, 0.59]`.

A atenção trata a frase como um **saco de palavras**: compara todas com todas, mas **não sabe a ordem**.

## 2. Codificação posicional 📍

Solução: **somar ao embedding um vetor que representa a posição** da palavra.

```
vetor final = embedding da palavra + vetor da posição
```

O artigo original usa **ondas de seno e cosseno** com velocidades diferentes: cada posição gera uma "assinatura" única. No notebook usamos uma versão simplificada:

| Posição | Vetor de posição |
|---|---|
| 0 | [0.00, 0.30, 0.00] |
| 1 | [0.25, 0.26, 0.07] |
| 2 | [0.27, 0.16, 0.14] |
| 3 | [0.04, 0.02, 0.20] |

> 💡 **Analogia:** é como o **número da poltrona** no cinema. Duas pessoas iguais em poltronas diferentes estão em situações diferentes.

Com isso, "cachorro" na posição 0 e na posição 2 passa a ter vetores **diferentes**.

## 3. Múltiplas cabeças de atenção (multi-head) 🐙

Uma única atenção olha a frase de **um jeito só**. O Transformer roda **várias atenções em paralelo**, cada uma com suas próprias W_Q, W_K e W_V.

> 💡 **Analogia:** um texto analisado por vários professores — o de português olha a gramática, o de história olha o contexto, o de geografia olha os lugares. Depois, juntam as anotações.

No notebook, para a palavra **"gato"**:

| Cabeça | gato | bebeu | leite | O que ela "procura" |
|---|---|---|---|---|
| 1 (gramática) | 10.1% | **76.9%** | 13.0% | A ação |
| 2 (semelhança) | **46.8%** | 22.8% | 30.4% | Palavras do mesmo tipo |

As saídas das cabeças são **concatenadas** (coladas lado a lado) e passam por uma matriz **W_O** que volta ao tamanho original.

| Modelo | Cabeças por camada |
|---|---|
| BERT-base | 12 |
| GPT-3 | 96 |

## 4. Residual, normalização e feed-forward

| Peça | O que faz | Analogia |
|---|---|---|
| **Conexão residual** | Soma a entrada com a saída da atenção | Manter o texto original e só acrescentar anotações à margem |
| **Normalização** | Deixa os números com média 0 e escala parecida | Ajustar o volume para nada estourar |
| **Feed-forward** | Pequena rede neural aplicada a cada palavra | Cada palavra "pensa sozinha" sobre o que aprendeu |

A rede feed-forward usa a **ReLU**: negativo vira 0, positivo continua igual.

```
relu([-2, 0.5, -0.1]) = [0, 0.5, 0]
```

## 5. O bloco Transformer

```
 entrada (embedding + posição)
        │
        ▼
 ┌────────────────────────────┐
 │ Multi-Head Attention       │
 │ + Residual  → Normalização │
 │ Feed-Forward (ReLU)        │
 │ + Residual  → Normalização │
 └────────────────────────────┘
        │
        ▼
 saída (mesmo tamanho da entrada)
```

Como a saída tem o **mesmo tamanho** da entrada, os blocos podem ser **empilhados**: o BERT-base tem 12 blocos; o GPT-3, 96.

## 6. A máscara 🙈

Modelos que **geram texto** (GPT) produzem uma palavra por vez. No treinamento, uma palavra **não pode ver as que vêm depois** — seria "colar" a resposta.

A **máscara** troca os escores das palavras futuras por `-1000`; o softmax transforma isso em **0%**:

| Quem olha ↓ / para quem → | o | gato | bebeu | leite |
|---|---|---|---|---|
| **o** | 100% | 0% | 0% | 0% |
| **gato** | 56.7% | 43.3% | 0% | 0% |
| **bebeu** | 7.4% | 33.0% | 59.6% | 0% |
| **leite** | 10.8% | 10.6% | 66.0% | 12.7% |

Repare no "triângulo" de zeros: ninguém enxerga o futuro.

| Tipo | Exemplo | Máscara? | Uso típico |
|---|---|---|---|
| **Encoder** | BERT | Não (olha os dois lados) | Classificar, entender textos |
| **Decoder** | GPT, Claude | Sim (só o passado) | Gerar texto |
| **Encoder-Decoder** | T5, tradutores | Encoder sem, decoder com | Traduzir, resumir |

## 7. Prevendo a próxima palavra 🔮

1. A frase passa pelos blocos (com máscara);
2. Pega-se o vetor da **última palavra**;
3. Compara-se esse vetor (produto escalar) com o embedding de **todas** as palavras do vocabulário;
4. **Softmax** → probabilidade de cada palavra ser a próxima.

No nosso modelo, para *"o gato bebeu ___"*, as palavras mais prováveis foram... **outros verbos**! Isso acontece porque o modelo **não foi treinado**.

> 🔑 **Treinamento** = mostrar milhões de frases e ajustar, pouco a pouco, **todos os números** (embeddings, W_Q, W_K, W_V, W_O, W_1, W_2) até as previsões ficarem boas. Um modelo grande tem **bilhões** desses números (os famosos "parâmetros").

## 8. Bônus: BERTimbau

O notebook termina com o **BERTimbau** (`neuralmind/bert-base-portuguese-cased`), um Transformer treinado em português, adivinhando palavras escondidas com `[MASK]`:

- *"Fui ao [MASK] sacar dinheiro."*
- *"Sentei no [MASK] da praça para descansar."*

É o momento de mostrar que **tudo o que construímos à mão existe, em escala gigante, nos modelos reais**.

---

## 🧑‍🏫 Roteiro sugerido em sala

| Tempo | Atividade |
|---|---|
| 10 min | Problema da ordem + codificação posicional |
| 10 min | Multi-head (analogia dos professores) |
| 10 min | Residual, normalização, feed-forward e o bloco completo |
| 10 min | Máscara e previsão da próxima palavra |
| 10 min | Bônus BERTimbau + fechamento |

## ❓ Perguntas para discussão

1. Por que o ChatGPT precisa de máscara e o BERT não?
2. Por que ter várias cabeças de atenção é melhor que uma só?
3. Nosso modelo previu um verbo depois de "o gato bebeu". O que precisaria acontecer para ele acertar?

## 🗺️ Mapa geral da aula

| Parte | Pergunta respondida | Ferramenta principal |
|---|---|---|
| 1 | Como transformar texto em números? | Tokens, embeddings, produto escalar |
| 2 | Como transformar notas em %? | Softmax |
| 3 | Como a palavra ganha contexto? | Q, K, V e soma ponderada |
| 4 | Como montar o Transformer? | Posição, multi-head, residual, feed-forward, máscara |

## 📚 Glossário

- **Codificação posicional:** vetor somado ao embedding para indicar a posição da palavra.
- **Multi-head attention:** várias atenções em paralelo, cada uma com suas matrizes.
- **Concatenar:** colar listas lado a lado.
- **Conexão residual:** somar a entrada de uma etapa à sua saída.
- **Normalização:** ajustar os números para média 0 e escala parecida.
- **ReLU:** função que troca números negativos por 0.
- **Feed-forward:** pequena rede neural aplicada a cada palavra separadamente.
- **Máscara:** bloqueia a atenção para palavras futuras.
- **Parâmetros:** todos os números ajustados durante o treinamento.

## 📖 Para saber mais

- Vaswani et al. (2017). *Attention Is All You Need.* — o artigo original do Transformer.
- Jay Alammar. *The Illustrated Transformer.* — explicação visual muito usada em aulas.
- Souza, Nogueira e Lotufo (2020). *BERTimbau: Pretrained BERT Models for Brazilian Portuguese.*
