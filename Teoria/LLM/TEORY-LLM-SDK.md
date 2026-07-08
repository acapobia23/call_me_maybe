# LLM Function Calling - Constrained Decoding

## Introduzione

Questo progetto riguarda l'interazione con un Large Language Model (LLM) attraverso un SDK
personalizzato chiamato `Small_LLM_Model`.

L'obiettivo principale non è semplicemente chiedere al modello di generare una risposta,
ma controllare **come il modello genera ogni singolo token**, imponendo dei vincoli.

In particolare il sistema deve essere in grado di generare output JSON:

- sintatticamente validi;
- conformi ad uno schema preciso;
- sempre parsabili;
- compatibili con una definizione di funzione.

Il concetto fondamentale è:

> Non dobbiamo sperare che il modello generi JSON corretto.
> Dobbiamo controllare il processo di generazione affinché ogni token prodotto sia valido.

Questo approccio prende il nome di:

**Constrained Decoding**

---

# 1. Come funziona un LLM

Un Large Language Model non lavora direttamente con le parole.

Il modello riceve numeri chiamati:

```
Token IDs
```

e restituisce probabilità per il token successivo.

La generazione avviene un token alla volta.

Esempio:

Input:

```
What is the sum of 2 and 3?
```

Il modello non vede:

```
"What"
"is"
"the"
"sum"
```

ma una sequenza numerica:

```
[892,318,262,4771,286,16,290,17,30]
```

Questi numeri rappresentano token presenti nel vocabolario.

---

# 2. Il Vocabolario (Vocabulary)

Ogni LLM possiede un file vocab.

Questo file contiene una relazione:

```
TOKEN ID <------> STRINGA
```

Esempio:

```
ID       Token

892      "What"
318      " is"
262      " the"
4771     " sum"
286      " of"
16       " 2"
290      " and"
17       " 3"
30       "?"
```

Il modello lavora solamente con gli ID.

Per capire cosa significa un token dobbiamo fare il mapping:

```
        ENCODING

"hello world"
        |
        v

[15496, 995]


        DECODING

[15496,995]
        |
        v

"hello world"
```

---

# 3. Pipeline completa di generazione

La generazione standard segue questi passaggi:

```
                USER PROMPT

                     |
                     v

          +----------------+
          |  TOKENIZATION  |
          +----------------+

                     |
                     v

             TOKEN IDS

        [15496, 995, 220]

                     |
                     v

          +----------------+
          |      LLM       |
          +----------------+

                     |
                     v

                LOGITS

     token_1 : 0.01
     token_2 : 0.85
     token_3 : 0.02

                     |
                     v

          TOKEN SELECTION

              token_2

                     |
                     v

          TOKEN AGGIUNTO

       prompt + token_2

                     |
                     v

              RIPETI
```

---

# 4. Cosa sono i logits

I logits sono valori numerici prodotti dal modello.

Rappresentano quanto il modello considera probabile ogni token.

Esempio:

```
ID     Token       Logit

10     "cat"       1.2
20     "dog"       5.8
30     "car"       0.3
40     "tree"     -2.1
```

Il modello sceglierà probabilmente:

```
dog
```

perché ha il valore più alto.

Prima della scelta:

```
              LOGITS

        +----------------+
        | token -> score |
        +----------------+

        cat  -> 1.2
        dog  -> 5.8
        car  -> 0.3
        tree -> -2.1
```

Dopo:

```
        selected token

              dog
```

---

# 5. Problema del JSON generato dal modello

Supponiamo di chiedere:

```
Genera un JSON per chiamare questa funzione:

add_numbers(a:number,b:number)
```

Un LLM potrebbe generare:

```json
{
 "a": 10,
 "b": "ciao"
}
```

oppure:

```json
{
 "a": 10,
```

oppure:

```json
{
a:10
}
```

Il modello può sbagliare perché:

- genera testo probabilistico;
- non conosce realmente una grammatica JSON;
- non garantisce lo schema.

---

# 6. Perché il Prompting non basta

Una soluzione ingenua sarebbe:

```
Prompt:

"Genera solamente JSON valido"
```

Risultato:

```json
{
 "number": 10
}
```

A volte funziona.

A volte no.

Perché?

Perché il modello ragiona in probabilità.

Non esiste nessuna regola interna che impedisce:

```json
{
"number": "hello"
}
```

La soluzione corretta:

intervenire **DURANTE** la generazione.

---

# 7. Constrained Decoding

Il constrained decoding modifica il comportamento del modello.

Normalmente:

```
LLM

        produce logits

              |
              v

        sceglie token
```

Constrained decoding:

```
LLM

        produce logits

              |
              v

+----------------------------+
| CONTROLLO VINCOLI          |
|                            |
| - JSON valido              |
| - Schema corretto          |
| - Tipo corretto            |
+----------------------------+

              |
              v

        token valido
```

---

# 8. Concetto fondamentale

Ad ogni passo il modello propone:

```
"Qualsiasi token possibile"
```

Noi invece vogliamo:

```
"Solo token che mantengono valido il JSON"
```

Esempio:

Stiamo generando:

```json
{
 "age":
}
```

Il modello potrebbe proporre:

```
10
"hello"
true
[
{
```

Ma lo schema dice:

```
age : integer
```

quindi:

VALIDI:

```
10
20
30
```

INVALIDI:

```
"hello"
true
[
{
```

---

# 9. Modifica dei logits

Il modello produce:

```
TOKEN        LOGIT

10           4.5
"hello"      5.1
true         3.0
[            2.2
20           4.0
```

Normalmente vincerebbe:

```
"hello"
```

perché:

```
5.1 > 4.5
```

Ma il decoder controlla lo schema.

Diventa:

```
TOKEN        LOGIT

10           4.5
"hello"     -INF
true        -INF
[           -INF
20           4.0
```

Ora:

```
max(logits)

=

10
```

---

# 10. Perché usare -∞

Mettere:

```
logit = -infinity
```

significa:

```
"questo token non può mai essere scelto"
```

Matematicamente:

```
softmax(-∞)=0
```

quindi:

```
probabilità = 0%
```

---

# 11. Il ruolo del vocabulary file

Per modificare i logits dobbiamo sapere:

```
Quale token rappresenta quale stringa?
```

Esempio:

Il modello restituisce:

```
token id 15496
```

Dal vocab:

```
15496 -> "hello"
```

Ora possiamo capire:

```
Questo token è valido?
```

Flusso:

```
              LOGITS

                |
                v

        TOKEN ID POSSIBILI

                |
                v

        CONSULTA VOCAB

                |
                v

        TOKEN -> STRINGA

                |
                v

        CONTROLLO JSON

                |
                v

        VALIDO / INVALIDO
```

---

# 12. Schema Validation

Non basta controllare JSON.

Serve controllare anche lo schema.

Esempio:

Schema:

```json
{
"name":"string",
"age":"integer"
}
```

JSON valido:

```json
{
"name":"Mario",
"age":20
}
```

JSON non valido:

```json
{
"name":20,
"age":"ciao"
}
```

Entrambi sono JSON validi.

Ma solo il primo rispetta lo schema.

---

# 13. Generazione token per token con schema

Esempio:

Schema:

```json
{
"number":"integer"
}
```

Generazione:

Step 1:

```
{
```

Valido.


Step 2:

```
{"
```

Valido.


Step 3:

```
{
"number"
```

Valido.


Step 4:

```
{
"number":
```

Ora sono ammessi solo numeri.

Il decoder blocca:

```
"a"
"hello"
true
```

---

# 14. Pipeline completa finale

```
                    USER

                     |
                     v

             Natural Language

                     |
                     v

              TOKENIZER

                     |
                     v

              TOKEN IDS

                     |
                     v

                 LLM

                     |
                     v

                LOGITS

                     |
                     v

        +-----------------------+
        | CONSTRAINED DECODER   |
        |                       |
        | 1. Legge vocab        |
        | 2. Decodifica token   |
        | 3. Controlla schema   |
        | 4. Blocca invalidi    |
        +-----------------------+

                     |
                     v

              VALID TOKEN

                     |
                     v

              JSON OUTPUT
```

---

# 15. SDK utilizzato

La classe:

```
Small_LLM_Model
```

fornisce:

---

## get_logits_from_input_ids()

Input:

```python
[
15496,
995
]
```

Output:

```python
[
0.1,
5.2,
0.4
]
```

Restituisce i logits del prossimo token.

---

## encode()

Trasforma:

```python
"hello"
```

in:

```python
[15496]
```

---

## decode()

Trasforma:

```python
[15496]
```

in:

```python
"hello"
```

---

## get_path_to_vocab_file()

Restituisce:

```
path/to/vocab.json
```

necessario per capire il significato dei token.

---

# Conclusione

Un LLM normale genera testo probabilisticamente.

Un sistema di Function Calling affidabile deve invece controllare la generazione.

Il constrained decoding crea un ponte tra:

```
Probabilità del modello
```

e

```
Regole deterministiche dello schema
```

Il modello continua a decidere cosa vuole dire,
ma il decoder decide cosa è permesso dire.

Il risultato è:

```
LLM
 +
Vocabulary
 +
Schema Validator
 +
Logit Filtering

=

Output JSON sempre valido
```