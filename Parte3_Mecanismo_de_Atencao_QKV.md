# Parte 3 — O mecanismo de atenção: Query, Key e Value

**Aula:** Como funciona um Transformer na prática (mecanismo de atenção)
**Público:** iniciantes em programação · **Duração sugerida:** 50 a 60 min
**Notebook:** `Parte3_Mecanismo_de_Atencao_QKV.ipynb`

---

## 🎯 Objetivos

- Calcular uma **soma ponderada** de vetores;
- Mostrar como a atenção **muda o significado** de uma palavra conforme o contexto;
- Entender os papéis de **Query, Key e Value**;
- Compreender a fórmula da atenção do artigo *Attention Is All You Need* (2017).

---

## 1. Soma ponderada: misturando palavras

Os pesos da Parte 2 dizem **quanto** cada palavra olha para as outras. Agora vamos usá-los para **criar um novo vetor**:

> **novo vetor = (peso₁ × vetor₁) + (peso₂ × vetor₂) + (peso₃ × vetor₃)**

> 💡 **Analogia do suco:** misturar 50% de laranja, 30% de morango e 20% de limão. O resultado tem "um pouco de cada" na proporção dos pesos.

Para **"bebeu"** com pesos 22.3% (gato), 49.2% (bebeu) e 28.4% (leite):

```
                   [animal, comida, ação]
0.223 × gato   =   [0.20,   0.02,   0.00]
0.492 × bebeu  =   [0.05,   0.15,   0.44]
0.284 × leite  =   [0.00,   0.26,   0.03]
                   ---------------------
novo "bebeu"   ≈   [0.25,   0.43,   0.47]
```

| | animal | comida | ação |
|---|---|---|---|
| "bebeu" **antes** | 0.10 | 0.30 | 0.90 |
| "bebeu" **depois** | 0.25 | 0.43 | 0.47 |

O novo vetor ganhou "animal" (do gato) e "comida" (do leite). Ele agora **carrega o contexto da frase**.

## 2. Por que isso importa: o exemplo do "banco" 🏦🪑

| Frase | Sentido de "banco" |
|---|---|
| *Saquei dinheiro no banco* | instituição financeira |
| *Sentei no banco da praça* | assento |

O embedding de "banco" é **sempre o mesmo** `[1.0, 1.0, 0.5]` (dinheiro, descanso, ar livre). Após a atenção:

| | dinheiro | descanso | ar livre |
|---|---|---|---|
| "banco" original | 1.00 | 1.00 | 0.50 |
| "banco" na frase do **dinheiro** | **2.21** | 0.25 | 0.13 |
| "banco" na frase da **praça** | 0.34 | **1.17** | **0.99** |

> 🔑 **Mesma palavra, vetores diferentes conforme a frase.** Modelos antigos (como o Word2Vec) davam um único vetor para "banco". O Transformer gera um vetor **contextual**.

## 3. Query, Key e Value

A atenção simples tem dois problemas: cada palavra olha muito para si mesma e o **mesmo vetor** faz três papéis ao mesmo tempo. O Transformer cria **três versões** de cada palavra.

### 📚 Analogia da biblioteca

| Papel | Na biblioteca | Pergunta |
|---|---|---|
| **Query (Q)** | O que você digita na busca | "O que estou procurando?" |
| **Key (K)** | A etiqueta na lombada do livro | "O que eu ofereço?" |
| **Value (V)** | O conteúdo do livro | "Que informação eu entrego?" |

Você compara sua **busca** com as **etiquetas** para decidir quais livros abrir, e então lê o **conteúdo** deles.

- **Escores** = Query da palavra em foco · Key de cada palavra
- **Saída** = soma ponderada dos **Values**

### Como criar Q, K e V

Multiplicando o embedding por três **matrizes de pesos**: W_Q, W_K e W_V.

**Multiplicar matriz × vetor** = fazer o **produto escalar de cada linha** da matriz com o vetor:

```
| 1 0 0 |   | 1 |   | 1·1 + 0·1 + 0·1 |   | 1 |
| 0 2 0 | × | 1 | = | 0·1 + 2·1 + 0·1 | = | 2 |
| 0 0 3 |   | 1 |   | 0·1 + 0·1 + 3·1 |   | 3 |
```

Na aula, montamos a W_Q à mão com uma ideia linguística:
- **animais e comidas procuram ações** ("o que fizeram comigo?");
- **ações procuram animais e comidas** ("quem fez? com o quê?").

W_K e W_V são a **matriz identidade** (não alteram o vetor), para destacar o efeito da Query.

> ⚠️ Em modelos reais, **as três matrizes são aprendidas no treinamento**. Ninguém as escreve à mão.

## 4. A escala: dividir por √d

Como vimos na Parte 2, escores grandes deixam o softmax "decidido demais". Com vetores de 768 posições, os produtos escalares ficam enormes. Por isso, divide-se por **√d** (raiz do tamanho do vetor). Na aula, d = 3 → divide por ≈ 1.73.

## 5. A fórmula completa

$$\text{Atenção}(Q, K, V) = \text{softmax}\left(\frac{Q \cdot K}{\sqrt{d}}\right) \cdot V$$

Traduzindo para português:

| Pedaço da fórmula | O que significa | Parte da aula |
|---|---|---|
| Q · K | Quanto o que eu procuro combina com o que cada um oferece | Parte 1 |
| ÷ √d | Diminuir escores grandes demais | Parte 3 |
| softmax | Transformar em porcentagens | Parte 2 |
| · V | Misturar as informações com esses pesos | Parte 3 |

## 6. Resultado com Q, K e V

| Quem olha ↓ / para quem → | gato | bebeu | leite |
|---|---|---|---|
| **gato** | 14.4% | **68.5%** | 17.1% |
| **bebeu** | **37.8%** | 27.7% | **34.5%** |
| **leite** | 17.2% | **63.1%** | 19.8% |

Compare com a Parte 2, em que cada palavra olhava ~50% para si mesma. Agora:
- **gato** olha para **bebeu** → "o que eu fiz?";
- **bebeu** olha para **gato** e **leite** → "quem? o quê?";
- **leite** olha para **bebeu** → "o que fizeram comigo?".

---

## 🧑‍🏫 Roteiro sugerido em sala

| Tempo | Atividade |
|---|---|
| 10 min | Soma ponderada (analogia do suco) — calcular "bebeu" no quadro |
| 10 min | Exemplo do "banco" no notebook — momento "uau" da aula |
| 15 min | Analogia da biblioteca + multiplicação matriz × vetor no quadro |
| 10 min | Fórmula da atenção, peça por peça |
| 10 min | Tabela final + exercícios (alterar o "3" da W_Q) |

## ❓ Perguntas para discussão

1. Por que é útil separar Query, Key e Value em vez de usar só o embedding?
2. Que outras palavras ambíguas existem em português? (manga, pena, vela, cravo...)
3. O que aconteceria se não dividíssemos por √d em um modelo com vetores de 1000 posições?

## 📚 Glossário

- **Soma ponderada:** soma em que cada parcela é multiplicada por um peso.
- **Self-attention:** atenção em que as palavras de uma frase olham para as palavras da **mesma** frase.
- **Query / Key / Value:** versões do embedding para buscar, ser encontrado e entregar informação.
- **Matriz:** tabela de números (em Python, uma lista de listas).
- **Matriz identidade:** matriz com 1 na diagonal e 0 no resto; não altera o vetor.
- **Vetor contextual:** vetor de uma palavra que depende das outras palavras da frase.

➡️ **Próxima parte:** montar o **bloco Transformer completo**.
