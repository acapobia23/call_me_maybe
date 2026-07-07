# 📞 Call Me Maybe
### *Study Notes - Understanding the Subject Before Coding*

> "Don't write code until you know exactly what problem you're solving."

---

# 📚 Roadmap

```text
                START
                  │
                  ▼
     What is Function Calling?
                  │
                  ▼
      How does an LLM generate text?
                  │
                  ▼
      What are Tokens and Logits?
                  │
                  ▼
      How is the next token chosen?
                  │
                  ▼
      What is Constrained Decoding?
                  │
                  ▼
      Why does it solve the problem?
                  │
                  ▼
      How does llm_sdk help me?
                  │
                  ▼
              START CODING
```

---

# 🎯 The Goal

The project is **NOT** about executing functions.

The project is about converting:

```
Natural Language
```

into

```
Structured Function Call
```

---

# ❓1 - What is Function Calling?

## Question

What exactly is Function Calling?

## Answer

Function Calling is the process of transforming a natural language request into a function invocation.

Instead of answering the user's question, the model identifies:

- which function should be called
- which parameters should be passed

Example:

User says:

```
What is the sum of 4 and 9?
```

The output is **NOT**

```
13
```

The output becomes

```json
{
    "name": "fn_add_numbers",
    "parameters":
    {
        "a": 4,
        "b": 9
    }
}
```

The function is **never executed**.

---

## Mental Model

```
          User

"I want to add 4 and 9"

            │
            ▼

   Which function?

            │

            ▼

 fn_add_numbers(4,9)

            │

            ▼

 Produce JSON only
```

---

# ❓2 - Why don't we execute the function?

## Question

Why doesn't the project return the final answer?

## Answer

Because this project is only responsible for understanding the user's intent.

Another program could later execute that function.

Our job finishes once we correctly identify

- the function
- the arguments

Think of yourself as a translator.

```
Human Language

        │

        ▼

Machine Instruction
```

---

# ❓3 - What information do we have?

## Question

How do we know which functions exist?

## Answer

They are stored inside a JSON file.

Example

```
functions_definition.json
```

Inside we can find

```
Function Name

Description

Arguments

Argument Types

Return Type
```

Our program must read this information.

---

# ❓4 - Can I simply search for keywords?

## Question

Can I use something like

```
if "sum" in prompt:
```

?

## Answer

No.

The subject explicitly forbids this.

The LLM must decide which function is the correct one.

Otherwise it would not be AI anymore.

---

# ❓5 - What is an LLM?

## Question

What is a Large Language Model?

## Answer

An LLM predicts the next token.

Nothing more.

Everything it writes is produced one token at a time.

---

## Mental Model

```
Prompt

↓

Token

↓

Token

↓

Token

↓

Token

↓

Sentence
```

---

# ❓6 - What is a Token?

## Question

Is a token a word?

## Answer

Not necessarily.

Sometimes

```
hello
```

is one token.

Sometimes

```
playing
```

becomes

```
play
ing
```

Sometimes spaces are tokens.

Sometimes punctuation is a token.

The model never reasons using characters.

It reasons using tokens.

---

# ❓7 - What happens inside an LLM?

```
               Prompt

                  │

                  ▼

          Tokenization

                  │

                  ▼

           Token IDs

                  │

                  ▼

               LLM

                  │

                  ▼

             Logits

                  │

                  ▼

       Next Token Selection

                  │

                  ▼

          Generated Token

                  │

                  ▼

      Repeat until finished
```

---

# ❓8 - What are Token IDs?

## Question

Can an LLM understand text?

## Answer

No.

It only understands numbers.

Example

```
hello
```

may become

```
[4217]
```

The tokenizer performs this conversion.

---

# ❓9 - What are Logits?

## Question

What are logits?

## Answer

Imagine the model has to choose the next token.

For every token in its vocabulary it produces a score.

Example

```
Token A → 3.5

Token B → 8.2

Token C → -1.4

Token D → 4.1
```

These scores are called

```
Logits
```

Higher score

↓

Higher probability

---

# ❓10 - How is the next token selected?

Normally

```
Highest Logit

↓

Chosen Token
```

or

```
Randomly

weighted by probability
```

depending on the sampling strategy.

---

# ❓11 - Why is JSON difficult?

Small LLMs make mistakes.

Example

```
{
"name"
```

or

```
{
"name":
```

or

```
{
"name":"hello"
```

The JSON becomes invalid.

This happens surprisingly often.

---

# ❓12 - What is Constrained Decoding?

## Question

What does constrained decoding do?

## Answer

Instead of allowing every token...

```
Vocabulary

↓

30000 possible tokens
```

we restrict the possible choices.

Example

```
Allowed

{

"

name

:

}

```

Everything else becomes impossible.

---

## Visual Representation

```
Model proposes

A
B
C
D
E
F

↓

Validator

✓
✗
✓
✗
✗
✓

↓

Remaining Tokens

A

C

F
```

The model can only choose between A C and F.

---

# ❓13 - Why does this guarantee valid JSON?

Because every generated token is checked BEFORE being accepted.

```
Generate Token

↓

Would JSON stay valid?

↓

YES

↓

Accept

-------------

NO

↓

Reject
```

The model literally cannot generate invalid syntax.

---

# ❓14 - What does "Schema" mean?

JSON validity is not enough.

This

```json
{
    "age":"hello"
}
```

is valid JSON.

But if

```
age

↓

number
```

then

```
"hello"
```

is invalid.

Constrained Decoding must respect

- JSON syntax

AND

- data types

---

# ❓15 - Why is Constrained Decoding the heart of the project?

Without it

```
LLM

↓

Maybe valid JSON

Maybe not
```

With it

```
LLM

↓

Restricted Tokens

↓

Always Valid JSON
```

This is exactly what the project wants you to build.

---

# ❓16 - What is the role of llm_sdk?

The SDK is simply your interface with the model.

Think of it like this:

```
                Your Code

                     │

                     ▼

               llm_sdk

                     │

                     ▼

              Qwen Model
```

The SDK lets you

- tokenize text
- obtain logits
- access the vocabulary

It does **not** solve the project for you.

---

# ❓17 - What is my program really doing?

```
Read Functions

        │

        ▼

Read Prompt

        │

        ▼

Give Prompt to LLM

        │

        ▼

Receive Logits

        │

        ▼

Apply Constraints

        │

        ▼

Choose Next Token

        │

        ▼

Repeat

        │

        ▼

Produce JSON
```

---

# ❓18 - What should I understand before writing code?

You should be able to answer YES to every question below.

```
[ ] I know what Function Calling is.

[ ] I know why we don't execute functions.

[ ] I know what Tokens are.

[ ] I know what Token IDs are.

[ ] I know what Logits are.

[ ] I know how an LLM generates text.

[ ] I know why JSON generation is difficult.

[ ] I understand Constrained Decoding.

[ ] I know why schema validation matters.

[ ] I understand the overall architecture of the project.

[ ] Only now am I ready to study llm_sdk.
```

---

# 🧠 Final Mental Model

```
                Human

                  │

                  ▼

      "Reverse hello"

                  │

                  ▼

          Tokenization

                  │

                  ▼

               LLM

                  │

                  ▼

             Logits

                  │

                  ▼

      Constrained Decoding

                  │

                  ▼

      Valid JSON Generation

                  │

                  ▼

{
    "name":"fn_reverse_string",
    "parameters":
    {
        "s":"hello"
    }
}
```

---

# 🚀 Only after this...

Now you're ready to open `llm_sdk` and understand **how to obtain the logits**, because you'll finally know **why** you need them.