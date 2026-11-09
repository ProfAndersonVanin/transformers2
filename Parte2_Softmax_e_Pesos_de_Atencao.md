# Parte 2 — De pontuações para porcentagens: a função softmax

**Aula:** Como funciona um Transformer na prática (mecanismo de atenção)
**Público:** iniciantes em programação · **Duração sugerida:** 40 a 50 min
**Notebook:** `Parte2_Softmax_e_Pesos_de_Atencao.ipynb`

---

## 🎯 Objetivos

- Entender o que é um **escore de atenção**;
- Perceber por que dividir pela soma **não funciona** com números negativos;
- Calcular a função **softmax** passo a passo;
- Interpretar uma **tabela de atenção**.

---

## 1. A ideia de "prestar atenção"

Quando lemos *"o gato bebeu leite"*, para entender a palavra **"bebeu"** nosso cérebro automaticamente pergunta: *quem bebeu?* e *bebeu o quê?*. Ou seja, **prestamos atenção** em "gato" e "leite".

O Transformer faz o mesmo, mas com números: cada palavra calcula **quanto (em %) deve olhar para cada uma das outras**.

> 💡 **Analogia:** em uma sala de aula, quando o professor faz uma pergunta, cada aluno distribui sua atenção: 60% no professor, 30% no quadro, 10% no colega ao lado. A soma é sempre **100%**.

## 2. Escores

O primeiro passo é calcular o **produto escalar** (Parte 1) entre a palavra em foco e cada palavra da frase. Esses números são os **escores**.

Foco: **"bebeu"** na frase `["gato", "bebeu", "leite"]`

| bebeu olhando para | Escore |
|---|---|
| gato | 0.12 |
| bebeu | 0.91 |
| leite | 0.36 |

## 3. Tentativa 1: dividir pela soma

Soma = 0.12 + 0.91 + 0.36 = 1.39

| Palavra | Cálculo | % |
|---|---|---|
| gato | 0.12 ÷ 1.39 | 8.6% |
| bebeu | 0.91 ÷ 1.39 | 65.5% |
| leite | 0.36 ÷ 1.39 | 25.9% |

Funciona aqui... mas veja com escores **negativos** (que são muito comuns nos modelos reais):

```
escores = [2.0, -1.0, 0.5]     soma = 1.5

 2.0 ÷ 1.5 =  133.3%   ❌ maior que 100%
-1.0 ÷ 1.5 =  -66.7%   ❌ porcentagem negativa!
 0.5 ÷ 1.5 =   33.3%
```

## 4. A solução: exponencial

A função exponencial `exp(x)` (o número *e* ≈ 2.718 elevado a *x*):

| x | exp(x) |
|---|---|
| -2 | 0.135 |
| -1 | 0.368 |
| 0 | 1.000 |
| 1 | 2.718 |
| 2 | 7.389 |
| 3 | 20.086 |

Duas propriedades importantes:
1. **Sempre positiva** → acaba o problema das porcentagens negativas;
2. **Cresce muito rápido** → "destaca o vencedor".

## 5. A função softmax

**Softmax = exponencial de cada escore ÷ soma das exponenciais**

Exemplo com `[2.0, -1.0, 0.5]`:

```
1) Exponenciais:  exp(2.0) = 7.389
                  exp(-1.0) = 0.368
                  exp(0.5) = 1.649

2) Soma:          7.389 + 0.368 + 1.649 = 9.406

3) Divisão:       7.389 ÷ 9.406 = 78.6%
                  0.368 ÷ 9.406 =  3.9%
                  1.649 ÷ 9.406 = 17.5%
                                  ------
                                  100%  ✅
```

Os resultados do softmax são chamados de **pesos de atenção**.

## 6. Tabela de atenção

No Transformer, **todas as palavras olham para todas ao mesmo tempo**. Resultado do notebook:

| Quem olha ↓ / para quem → | gato | bebeu | leite |
|---|---|---|---|
| **gato** | **50.5%** | 25.1% | 24.4% |
| **bebeu** | 22.3% | **49.2%** | 28.4% |
| **leite** | 22.8% | 29.9% | **47.3%** |

Duas observações que vão motivar a Parte 3:

1. **Cada palavra olha mais para si mesma** — natural (uma palavra é sempre igual a ela mesma), mas pouco útil para entender o contexto.
2. **A atenção está "indecisa"** — nenhum valor passa muito de 50%, porque os escores são números pequenos e próximos.

## 7. O efeito do tamanho dos escores

| Escores de "bebeu" | gato | bebeu | leite |
|---|---|---|---|
| Normais `[0.12, 0.91, 0.36]` | 22.3% | 49.2% | 28.4% |
| × 10 `[1.2, 9.1, 3.6]` | 0.0% | **99.6%** | 0.4% |

- Escores **pequenos** → atenção **espalhada** (indecisa);
- Escores **grandes** → atenção **concentrada** (decidida demais).

> 🔑 Nos modelos reais, os vetores são enormes e os escores ficam gigantes. Por isso o Transformer **divide os escores por um número** antes do softmax (veremos na Parte 3).

---

## 🧑‍🏫 Roteiro sugerido em sala

| Tempo | Atividade |
|---|---|
| 5 min | Analogia da atenção na sala de aula |
| 10 min | Escores e a tentativa de dividir pela soma (mostrar o erro!) |
| 10 min | Exponencial e softmax — calcular um exemplo **à mão** |
| 10 min | Barrinhas e tabela de atenção no notebook |
| 10 min | Exercícios: outra frase, escores × 0.1 e × 100 |

## ❓ Perguntas para discussão

1. Por que as porcentagens precisam somar 100%?
2. O que acontece com o softmax se todos os escores forem iguais? (Resposta: todos recebem a mesma porcentagem.)
3. Por que é ruim uma palavra prestar atenção só nela mesma?

## 📚 Glossário

- **Escore:** "nota" de quanto uma palavra combina com outra (produto escalar).
- **Exponencial (`math.exp`):** função que transforma qualquer número em um número positivo.
- **Softmax:** transforma uma lista de escores em pesos que somam 1 (100%).
- **Peso de atenção:** porcentagem de atenção que uma palavra dá para outra.

➡️ **Próxima parte:** usar os pesos para **misturar as palavras** e criar o mecanismo de atenção com **Query, Key e Value**.
