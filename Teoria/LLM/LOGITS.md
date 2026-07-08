# 🧠 Come una LLM sceglie il prossimo token

## Context Vector, Embedding, Logits e spazio vettoriale

---

# Introduzione

Una delle parti più importanti per capire il funzionamento interno di una LLM
(**Large Language Model**) è capire come passa da:

```
Testo in input
```

a:

```
Probabilità dei prossimi token
```

Quando una LLM deve generare il prossimo token non ragiona come un essere umano.

Non pensa:

```
"The cat is"

→ un gatto normalmente dorme

→ quindi sleeping
```

In realtà esegue una enorme quantità di operazioni matematiche che trasformano
il testo in vettori numerici e calcolano quali token sono più compatibili
con il contesto corrente.

---

# 🔄 Flusso generale di generazione

```
                 TESTO

                   |
                   v

              TOKENIZER

                   |
                   v

              TOKEN IDS

                   |
                   v

             EMBEDDINGS

                   |
                   v

          TRANSFORMER NETWORK

                   |
                   v

           CONTEXT VECTOR

                   |
                   v

          OUTPUT PROJECTION

                   |
                   v

                LOGITS

                   |
                   v

               SOFTMAX

                   |
                   v

            PROSSIMO TOKEN
```

---

# 1. Dal testo ai Token

Una LLM non lavora direttamente con le parole.

Prendiamo:

```
"The cat is"
```

Il tokenizer divide il testo in token.

Esempio:

```
"The"   -> 1542
" cat"  -> 8321
" is"   -> 901
```

Otteniamo:

```
INPUT IDS

[
 1542,
  8321,
  901
]
```

Questi numeri sono solamente identificatori.

Il modello non conosce ancora il significato di:

```
8321
```

Per comprendere il token deve trasformarlo in una rappresentazione matematica.

---

# 2. Token Embedding

Ogni token possiede un embedding.

Un embedding è un insieme di numeri che rappresenta il token
all'interno di uno spazio matematico.

Esempio:

## Token:

```
cat
```

Embedding:

```
[
 0.25,
 0.71,
-0.12,
 0.43
]
```

---

## Token:

```
banana
```

Embedding:

```
[
-0.80,
 0.15,
 0.60,
-0.30
]
```

---

Questi numeri NON significano:

```
0.25 = animale

0.71 = piccolo
```

Non esiste una posizione precisa per ogni informazione.

Il significato è distribuito nella combinazione di tutti i valori.

---

# 3. Lo spazio vettoriale

Quando diciamo:

> "Il token si trova in uno spazio matematico"

intendiamo che ogni token è rappresentato come un punto
in uno spazio con migliaia di dimensioni.

Esempio semplificato in 2 dimensioni:

```
                 ANIMALI


                    cat
                     *
                    /
                   /
                  *
                dog



                              CIBO


                           banana
                              *

```

Nella realtà non abbiamo:

```
x
y
```

ma migliaia di coordinate:

```
[
0.234,
-0.512,
0.871,
0.123,
...
]
```

---

# 4. Il Transformer crea il Context Vector

Un singolo token ha un significato.

Ma:

```
cat
```

è diverso da:

```
The cat is
```

Il Transformer analizza tutti i token insieme.

---

## Flusso interno

```
                 INPUT

          "The"  "cat"  "is"


             |      |      |

             v      v      v


          EMBEDDING LAYER


             |      |      |

             v      v      v


       TRANSFORMER LAYERS


                    |
                    |
                    v


             CONTEXT VECTOR
```

---

Alla fine otteniamo:

```
h =

[
 0.42,
-0.15,
 0.77,
 0.31,
 ...
]
```

Questo vettore rappresenta:

```
"Qual è il significato del contesto attuale?"
```

---

# 5. Cosa contiene il Context Vector?

Una cosa fondamentale:

Il vettore NON contiene informazioni scritte.

Non contiene:

```
cat = animale

is = verbo

sleeping = azione
```

Non è un dizionario.

Le informazioni sono distribuite:

```
Numero 1
+
Numero 2
+
Numero 3
+
...
+
Numero N

=

significato complesso
```

Esempio:

```
h =

[
0.42,
-0.15,
0.77,
0.31
]
```

La combinazione di questi valori rappresenta:

```
"The cat is ..."
```

---

# 6. I token hanno dei pesi di uscita

La LLM deve trasformare:

```
Context Vector
```

in:

```
Probabilità di tutti i token possibili
```

Per fare questo utilizza una matrice chiamata:

```
Output Projection Matrix
```

Questa matrice contiene pesi associati ai token.

---

Esempio:

## sleeping

```
[
 0.40,
-0.10,
 0.80,
 0.30
]
```

---

## running

```
[
 0.20,
 0.50,
 0.10,
 0.90
]
```

---

## banana

```
[
-0.70,
 0.30,
 0.20,
-0.40
]
```

---

Questi vettori rappresentano la direzione matematica associata
a ogni possibile output.

---

# 7. Il confronto tra Context Vector e Token

La rete calcola:

> Quanto il contesto è compatibile con ogni token?

L'operazione utilizzata è simile al:

```
PRODOTTO SCALARE
```

Formula:

```
score = h · token_weight
```

cioè:

```
(context[0] * token[0])
+
(context[1] * token[1])
+
(context[2] * token[2])
+
...
```

---

# 8. Esempio matematico

## Context Vector

```
h =

[
0.4,
-0.1,
0.8
]
```

---

## Token "sleeping"

```
sleeping =

[
0.5,
-0.2,
0.9
]
```

Calcolo:

```
0.4 * 0.5
+
(-0.1) * (-0.2)
+
0.8 * 0.9


=

0.2
+
0.02
+
0.72


=

0.94
```

Risultato:

```
sleeping score = 0.94
```

---

## Token "banana"

```
banana =

[
-0.8,
0.3,
-0.1
]
```

Calcolo:

```
0.4 * -0.8
+
(-0.1) * 0.3
+
0.8 * -0.1


=

-0.32
-
0.03
-
0.08


=

-0.43
```

Risultato:

```
banana score = -0.43
```

---

Quindi:

```
sleeping = 0.94

banana   = -0.43
```

Il modello considera:

```
sleeping

più compatibile con:

"The cat is"
```

---

# 9. Flusso completo del calcolo

```
                  INPUT

             "The cat is"


                    |
                    v


               TOKENIZER


                    |
                    v


             [1542,8321,901]


                    |
                    v


              EMBEDDINGS


                    |
                    v


          VETTORI NUMERICI


                    |
                    v


             TRANSFORMER


                    |
                    v


            CONTEXT VECTOR


                    |
                    v


       +-----------------------+
       | confronto con token   |
       +-----------------------+


                    |
                    v


        sleeping      8.5

        running       5.2

        banana       -2.4

        dog            3.1


                    |
                    v


                 LOGITS


                    |
                    v


                SOFTMAX


                    |
                    v


        sleeping      92%

        running        5%

        dog            2%

        banana         1%


                    |
                    v


             PROSSIMO TOKEN


                sleeping
```

---

# 10. Perché sleeping riceve un valore alto?

Perché durante il training la rete ha visto milioni di esempi simili.

Esempi:

```
The cat is sleeping

The dog is sleeping

The baby is sleeping

The man is sleeping
```

Ogni volta che sbagliava:

```
Input:

"The cat is"


Output corretto:

sleeping


Output generato:

banana
```

veniva calcolato un errore.

---

## Processo di apprendimento

```
              ERRORE

                 |
                 v

          BACKPROPAGATION

                 |
                 v

          MODIFICA PESI

                 |
                 v

      PREVISIONE MIGLIORE
```

---

Dopo miliardi di esempi:

```
"The cat is"
```

produce un vettore molto compatibile con:

```
sleeping
```

---

# 11. La LLM non memorizza frasi

Una LLM NON contiene:

```
"The cat is" --> sleeping
```

come una tabella.

Non funziona come:

```
DATABASE


Input              Output

cat is             sleeping

dog is             sleeping
```

---

Funziona così:

```
INPUT

  |
  v

RETE MATEMATICA

  |
  v

TRASFORMAZIONE DEI VETTORI

  |
  v

PROBABILITÀ
```

---

# 12. Collegamento con il Constrained Decoding

Normalmente:

```
             Context Vector

                    |
                    v

                  LOGITS


        sleeping      8.5

        running       6.2

        banana       -3.2


                    |
                    v


             SCELTA TOKEN
```

---

Con constrained decoding:

```
             Context Vector

                    |
                    v

                  LOGITS


        sleeping      8.5

        running       6.2

        banana       -3.2


                    |
                    v


            CONTROLLO SCHEMA


        running  ❌ vietato

        banana   ❌ vietato


                    |
                    v


            LOGITS MODIFICATI


        sleeping      8.5

        running       -INF

        banana        -INF


                    |
                    v


               sleeping
```

---

# Conclusione

Una LLM sceglie un token perché:

```
1. Trasforma il testo in vettori.

2. Il Transformer crea una rappresentazione del contesto.

3. Ogni possibile token possiede pesi appresi.

4. La rete calcola la compatibilità tra contesto e token.

5. Questa compatibilità diventa un logit.

6. I logits diventano probabilità.

7. Il token più probabile viene scelto.
```

---

Il concetto chiave:

```
CONTEXT VECTOR

        +

PESI APPRESI DURANTE IL TRAINING

        +

OPERAZIONI MATEMATICHE

        =

LOGITS

        =

PROBABILITÀ DEL PROSSIMO TOKEN
```

---

La LLM non "sa" che:

```
sleeping
```

è corretto.

Ha imparato una funzione matematica che rende:

```
sleeping
```

il risultato più probabile quando il contesto
assomiglia a situazioni viste durante il training.