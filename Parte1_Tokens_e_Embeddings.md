# Parte 1 — Do texto aos números: tokens e embeddings

**Aula:** Como funciona um Transformer na prática (mecanismo de atenção)
**Público:** iniciantes em programação · **Duração sugerida:** 40 a 50 min
**Notebook:** `Parte1_Tokens_e_Embeddings.ipynb`

---

## 🎯 Objetivos

Ao final desta parte, o aluno deve ser capaz de:

- explicar por que o computador precisa transformar texto em números;
- dividir uma frase em **tokens** e montar um **vocabulário**;
- entender o que é um **embedding** (vetor de significado);
- calcular o **produto escalar** entre dois vetores e interpretar o resultado.

---

## 1. Ponto de partida: o computador não lê palavras

Um computador só sabe fazer contas. Para ele, a frase *"o gato bebeu leite"* não significa nada. Por isso, **todo modelo de linguagem** (ChatGPT, Gemini, Claude, Google Tradutor...) começa transformando o texto em números.

Esse processo tem três etapas:

```
"o gato bebeu leite"  ──►  tokens  ──►  IDs  ──►  vetores (embeddings)
```

## 2. Tokens

**Token** é um pedaço de texto. Na nossa aula, cada palavra é um token:

```
"o gato bebeu leite"  ──►  ["o", "gato", "bebeu", "leite"]
```

> 💬 **Curiosidade:** nos modelos reais, os tokens costumam ser **pedaços de palavras**. Por exemplo, "inacreditável" pode virar `["in", "acredit", "ável"]`. Isso permite que o modelo entenda palavras que nunca viu, juntando pedaços conhecidos.

## 3. Vocabulário e IDs

O **vocabulário** é a lista de todos os tokens que o modelo conhece, cada um com um número (ID).

| Token | ID |
|---|---|
| o | 0 |
| gato | 1 |
| bebeu | 2 |
| leite | 3 |
| cachorro | 4 |
| comeu | 5 |
| queijo | 6 |

A frase vira: `[0, 1, 2, 3]`.

### ⚠️ O problema dos IDs

O ID é só uma "etiqueta". O número 4 (*cachorro*) não está "perto" do 1 (*gato*) por serem animais. Se o modelo usasse só IDs, ele acharia que *cachorro* (4) é mais parecido com *comeu* (5) do que com *gato* (1) — o que não faz sentido nenhum!

## 4. Embeddings: números com significado

A solução é representar cada palavra como uma **lista de números**, em que cada posição mede uma **característica**. Essa lista é o **embedding**.

Na aula usamos 3 características inventadas:

| Palavra | Animal | Comida | Ação |
|---|---|---|---|
| gato | 0.9 | 0.1 | 0.0 |
| cachorro | 0.8 | 0.1 | 0.1 |
| leite | 0.0 | 0.9 | 0.1 |
| queijo | 0.1 | 0.8 | 0.0 |
| bebeu | 0.1 | 0.3 | 0.9 |
| comeu | 0.1 | 0.4 | 0.8 |

> 💡 **Analogia:** é como uma **ficha de personagem de videogame**, com atributos como força, velocidade e magia. Dois personagens com atributos parecidos são "do mesmo tipo".

**Nos modelos reais:**
- os vetores têm **centenas ou milhares** de posições (o BERT usa 768);
- **ninguém escolhe os números à mão**: eles são **aprendidos** durante o treinamento;
- as posições não têm nomes claros como "animal" ou "comida" — o modelo descobre sozinho as características úteis.

## 5. Produto escalar: medindo se duas palavras combinam

Para comparar dois vetores, usamos o **produto escalar**: multiplica posição por posição e soma tudo.

**Exemplo à mão — gato × cachorro:**

```
gato     = [0.9, 0.1, 0.0]
cachorro = [0.8, 0.1, 0.1]

0.9 × 0.8 = 0.72
0.1 × 0.1 = 0.01
0.0 × 0.1 = 0.00
           ------
           0.73   ◄── número alto: são parecidos!
```

**Exemplo à mão — gato × leite:**

```
0.9 × 0.0 = 0.00
0.1 × 0.9 = 0.09
0.0 × 0.1 = 0.00
           ------
           0.09   ◄── número baixo: pouco parecidos
```

Resultados obtidos no notebook:

| Comparação | Produto escalar |
|---|---|
| gato × cachorro | **0.73** |
| leite × queijo | **0.72** |
| bebeu × comeu | **0.85** |
| gato × leite | 0.09 |

> 🔑 **Ideia central da aula toda:** o produto escalar é a "régua" que o Transformer usa para decidir **em quais palavras prestar atenção**.

---

## 🧑‍🏫 Roteiro sugerido em sala

| Tempo | Atividade |
|---|---|
| 5 min | Pergunta provocadora: *"Como o ChatGPT lê o que você escreve?"* |
| 10 min | Tokens e vocabulário (células 1 a 4 do notebook) |
| 10 min | Problema dos IDs + embeddings (tabela no quadro) |
| 10 min | Produto escalar **à mão no quadro**, depois no notebook |
| 10 min | Exercício: adicionar "rato" e "suco" |

## ❓ Perguntas para discussão

1. Por que não podemos usar só o ID das palavras?
2. Que outras características você criaria para os embeddings? (ex.: "é líquido?", "é grande?")
3. Se dois vetores têm produto escalar alto, o que isso significa?

## 📚 Glossário

- **Token:** pedaço do texto processado pelo modelo.
- **Vocabulário:** conjunto de todos os tokens conhecidos.
- **Embedding:** vetor de números que representa o significado de um token.
- **Vetor:** lista de números (em Python, uma `list`).
- **Produto escalar:** soma das multiplicações posição a posição entre dois vetores.

➡️ **Próxima parte:** transformar essas "notas de semelhança" em **porcentagens de atenção**.
