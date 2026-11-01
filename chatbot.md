# 08. Rule-Based Chatbot

## 1) Code

```python
import re
import random

intents = {
    "greeting": {
        "patterns": ["hello", "hi", "hey", "good morning",
                     "good evening", "good afternoon"],
        "responses": ["Hello!", "Hi there!", "Greetings!",
                      "Hey! How can I help you?"]
    },

    "farewell": {
        "patterns": ["bye", "goodbye", "see you later",
                     "farewell", "take care"],
        "responses": ["Goodbye!", "Take care!",
                      "See you later!", "Have a great day!"]
    },

    "thanks": {
        "patterns": ["thank you", "thanks", "thank you so much",
                     "much appreciated", "thanks a lot"],
        "responses": ["You're welcome!", "Glad I could help!",
                      "Anytime!", "No problem!"]
    },

    "bot_name": {
        "patterns": ["what is your name", "who are you",
                     "tell me your name"],
        "responses": ["I am your friendly chatbot!",
                      "You can call me Chatbot!",
                      "I am a chatbot created to help you."]
    },

    "bot_purpose": {
        "patterns": ["what can you do", "how can you help me",
                     "what do you do"],
        "responses": ["I can answer basic questions and chat with you!",
                      "I am here to help you.",
                      "I can respond to simple queries."]
    },

    "feeling": {
        "patterns": ["how are you", "how are you doing",
                     "are you okay"],
        "responses": ["I am doing great!",
                      "I am feeling helpful today!",
                      "I am ready to help you."]
    },

    "age": {
        "patterns": ["how old are you", "what is your age",
                     "when were you created"],
        "responses": ["I do not have an age like humans.",
                      "Age is just a number for a chatbot!"]
    },

    "weather": {
        "patterns": ["what is the weather",
                     "how is the weather today",
                     "tell me the weather"],
        "responses": ["I cannot check live weather right now.",
                      "Please check your weather app for updates."]
    },

    "joke": {
        "patterns": ["tell me a joke", "make me laugh",
                     "tell a joke"],
        "responses": [
            "Why did the computer go to the doctor? It had a virus!",
            "I would tell you a UDP joke, but you might not get it!"
        ]
    },

    "help": {
        "patterns": ["help me", "i need help",
                     "can you help me"],
        "responses": ["Sure! What do you need help with?",
                      "Of course! How can I assist you?"]
    }
}


def preprocess(text):
    text = text.lower()
    text = re.sub(r'[^\w\s]', '', text)
    tokens = text.split()
    return tokens


def match_intent(processed_input):
    for intent, data in intents.items():
        for pattern in data["patterns"]:
            processed_pattern = preprocess(pattern)

            if all(word in processed_input
                   for word in processed_pattern):
                return intent

    return None


def get_response(intent):
    return random.choice(intents[intent]["responses"])


def chatbot():
    print("Chatbot: Hello! I am your chatbot.")
    print("Type 'bye' or 'exit' to end the conversation.")

    while True:
        user_input = input("You: ").strip()

        if user_input.lower() in ["bye", "exit"]:
            print("Chatbot: Goodbye!")
            break

        processed_input = preprocess(user_input)
        matched_intent = match_intent(processed_input)

        if matched_intent:
            response = get_response(matched_intent)
        else:
            response = "Sorry, I could not understand that."

        print("Chatbot:", response)


if __name__ == "__main__":
    chatbot()
```

## 2) Theory to write

### Question

**Official syllabus: Part B, assignment 4.** Develop a simple chatbot using predefined rules and pattern matching.

### Aim

To develop a simple conversational chatbot using predefined intent rules and word-based pattern matching.

### Objectives

1. Store conversational intents, patterns and possible responses.
2. Preprocess user text to reduce case/punctuation differences.
3. Match input against rules and select a response.
4. Handle unknown messages and allow the user to exit.

### Requirements

Python 3 with the standard-library `re` and `random` modules. No model training, Internet API or external package is used.

### Theory

A **chatbot** is a program that exchanges conversational messages with a user. A **rule-based chatbot** maps predefined input conditions to predefined responses. An **intent** is a category of user purpose, such as greeting, gratitude, asking the bot's name or requesting help.

This program's knowledge base is a nested dictionary of ten intents. Each intent contains several pattern strings and response strings. Preprocessing performs lowercasing, punctuation removal and whitespace tokenization.

The matching rule is: **if every token from a stored pattern occurs in the input tokens, select that intent**. This ignores word order, allows extra input words, and stops at the first match. It is not exact sentence equality, probabilistic intent classification, semantic understanding or a trained language model.

The regular expression is used for **text cleaning**. Intent recognition itself uses token membership with `all`, not `re.match` or `re.search`. A random response varies the wording but does not make the chatbot learn.

### Algorithm

1. Create a dictionary containing intents, patterns and response lists.
2. Display a greeting and exit instructions.
3. Read a message and strip leading/trailing spaces.
4. If its lowercased raw text is exactly `bye` or `exit`, say goodbye and stop.
5. Otherwise lowercase it, remove punctuation and split into tokens.
6. For each intent and each of its patterns, preprocess that pattern too.
7. If all pattern tokens occur in the input tokens, select the first matching intent.
8. Choose a random stored response for that intent, or a fallback if none matches.
9. Print the response and repeat.

### Observation / result

Inputs such as `HELLO!!!`, `what is your name`, and `thank you` select greeting, bot-name and thanks responses. An unknown input such as `xyz123` gets “Sorry, I could not understand that.” Exact `bye` or `exit` ends the conversation.

### Conclusion

The chatbot demonstrates predefined conversational rules, basic text normalization, intent selection and response generation. Its capability is limited to token patterns and responses stored in the knowledge base.

## 3) Explaining the question, code and functioning

### Understanding the question

The question asks for a small interactive program that responds to user messages using rules and patterns. It does not require model training or an Internet-based chatbot service.

**Say this in the oral:** “My chatbot stores intents, example patterns and response lists. It lowercases text, removes punctuation and tokenizes it. If all words of a pattern occur in the input, it selects that intent and returns a random predefined response. Unknown inputs get a fallback response.”

Memory cue: **Input -> Normalize -> Tokens -> Match intent -> Choose response -> Repeat.**

### How the whole program runs

Python imports modules, builds the intent dictionary and defines four functions. Direct execution reaches the main guard and calls `chatbot`. For each non-exit message, `chatbot` calls `preprocess`, then `match_intent`, then `get_response` for a match. `match_intent` also preprocesses every pattern it tests.

### Understand the knowledge base

Ten categories are supplied: `greeting`, `farewell`, `thanks`, `bot_name`, `bot_purpose`, `feeling`, `age`, `weather`, `joke` and `help`.

For example:

```python
"greeting": {
    "patterns": ["hello", "hi", "hey", ...],
    "responses": ["Hello!", "Hi there!", ...]
}
```

This is an illustrative excerpt; the `.py` files contain the complete lists. `intents["greeting"]["responses"]` retrieves that intent's response list. These are stored phrases, not training data used to learn a model in this program.

### Code in blocks

#### Block A: imports and data

The corresponding program lines import regular expressions and randomness, then define the nested dictionary. Each outer key names an intent. Dictionary insertion order determines which intent is tried first in supported modern Python 3 versions. If several intents match, the earlier one wins.

#### Block B: preprocessing

Relevant code block:

```python
def preprocess(text):
    text = text.lower()
    text = re.sub(r'[^\w\s]', '', text)
    tokens = text.split()
    return tokens
```

- `.lower()` normalizes capitalization.
- `re.sub(pattern, replacement, text)` replaces matches of a regular expression. Here matches are replaced by the empty string.
- The `r` before a string makes it a **raw string**, so backslashes reach the regex engine conveniently.
- In `[^\w\s]`, square brackets define a character class; `^` at its start means negation; `\w` means word characters, including letters/digits/underscore; `\s` means whitespace.
- The effect is to remove characters that are neither word characters nor whitespace. Digits and underscores are retained; do not say this keeps letters only.
- `.split()` with no separator splits on whitespace and returns a list of tokens.

Example: `preprocess("HELLO!!! How are YOU?")` returns `['hello','how','are','you']`.

Removal is not the same as replacing punctuation with a space: `"hello-world"` becomes `"helloworld"`, and `"what's"` becomes `"whats"`. Hyphenated/contracted phrases may therefore fail a stored pattern.

#### Block C: match the first intent

Relevant code block:

```python
def match_intent(processed_input):
    for intent, data in intents.items():
        for pattern in data["patterns"]:
            processed_pattern = preprocess(pattern)

            if all(word in processed_input
                   for word in processed_pattern):
                return intent

    return None
```

`.items()` yields `(intent,data)` pairs. The inner loop tries each stored pattern. Preprocessing both input and patterns puts them into the same form.

`all(word in processed_input for word in processed_pattern)` is true if every pattern word appears **somewhere** in the input list. Word order is not checked. A pattern `"how are you"` even matches `"you are how"`. Extra input tokens are allowed.

`return intent` immediately exits both loops by ending the function. `return None` means no intent was found. Repeated words are not counted properly as occurrences: testing membership twice does not require two copies of a word in the input.

#### Block D: choose response wording

Relevant code block:

```python
def get_response(intent):
    return random.choice(intents[intent]["responses"])
```

`random.choice` returns one element of a nonempty list. The caller supplies a recognized intent, so its response list exists. Repeating the same input can give a different sentence with the same intended meaning. Random response wording is not a measure of reasoning or learning.

#### Block E: conversation loop

Relevant code block:

```python
def chatbot():
    print("Chatbot: Hello! I am your chatbot.")
    print("Type 'bye' or 'exit' to end the conversation.")

    while True:
        user_input = input("You: ").strip()

        if user_input.lower() in ["bye", "exit"]:
            print("Chatbot: Goodbye!")
            break

        processed_input = preprocess(user_input)
        matched_intent = match_intent(processed_input)

        if matched_intent:
            response = get_response(matched_intent)
        else:
            response = "Sorry, I could not understand that."

        print("Chatbot:", response)
```

`while True` repeats until an explicit `break`. `.strip()` removes outside whitespace, not punctuation. The exact exit test happens **before** preprocessing.

Thus `bye` and ` BYE ` stop the loop, but `bye!` does not. `bye!` is subsequently cleaned to `bye`, recognized as farewell, and gets a farewell response while the loop continues. Similarly, `goodbye` can get a farewell response without ending the program. This is a small behavioral limitation you should be able to explain.

`if matched_intent` tests a nonempty intent string as true; `None` is false and uses the fallback. `print` displays the selected sentence; `get_response` returns it to the driver.

#### Block F: main guard

Relevant code block:

```python
if __name__ == "__main__":
    chatbot()
```

When Python runs the file directly, `__name__` is `"__main__"` and the loop starts. Importing the module defines its data/functions without automatically asking for input.

### Trace some messages

| Raw input | Tokens | Match/result |
| --- | --- | --- |
| `HELLO!!!` | `['hello']` | greeting; choose a stored greeting |
| `what is your name?` | `['what','is','your','name']` | bot_name |
| `thank you` | `['thank','you']` | thanks |
| `you are how` | `['you','are','how']` | feeling; word order is ignored |
| `hello what is your name` | includes both greeting and name words | greeting wins because it is earlier |
| `xyz123` | `['xyz123']` | no intent; fallback |
| `bye!` | `['bye']` | farewell response, conversation continues |
| `bye` | not preprocessed in this branch | exact exit; loop stops |

To add an intent, provide a new key with the same `patterns` and `responses` structure. Consider where it appears if its rules overlap existing ones.

### Complexity and limitations

If an input has `N` tokens and all patterns together contain `W` tokens, repeated list membership tests can cost about `O(W*N)`, plus input/pattern preprocessing and dictionary iteration. The fixed small database makes this inexpensive. For large rule sets, preprocessed patterns and input token sets can improve speed, though token sets still ignore word order and occurrence counts.

The chatbot has no context memory, semantic understanding, negation handling or live weather access. It may choose a rule from an unrelated longer sentence if the required words happen to occur. An empty input finds no supplied pattern. End-of-file input is not caught, so run it interactively and use the exit command.

If asked for one improvement, suggest prioritized rules, phrase/sequence matching, a normalized exit check, or conversational state. Explain the existing version first. Do not claim those improvements are already implemented.

### Viva questions with short answers

1. **What is an intent?** A category of the user's purpose, such as greeting or asking for help.
2. **Is the bot machine-learning based?** No. It uses fixed rules and stored responses.
3. **Why preprocess text?** To reduce case/punctuation variation before matching.
4. **What is tokenization here?** Splitting the cleaned text by whitespace into words.
5. **What does the regex remove?** Characters that are neither word characters nor whitespace.
6. **What does `r'...'` mean?** A raw string literal, useful for regex backslashes.
7. **Does the program use regex to detect each intent?** No. Regex cleans text; intent matching uses token membership.
8. **What does `all` check?** Every stored-pattern token must appear in the input tokens.
9. **Does order matter in the match?** No. This rule ignores word order.
10. **What if two intents match?** The first encountered intent/pattern wins.
11. **Why `random.choice`?** To vary response wording within the selected intent.
12. **Does random response selection mean learning?** No. No knowledge or parameters are updated.
13. **What is a fallback?** A default response when no rule matches.
14. **Why does `bye!` not stop?** The exact exit comparison happens before punctuation removal.
15. **Can it report actual current weather?** No. Its weather response says it cannot check live weather.
16. **What does the main guard do?** Starts chat on direct execution but not ordinary import.
17. **What is the main limitation?** Token overlap is weaker than understanding meaning and context.
18. **How do you extend it?** Add intents/patterns/responses and manage overlaps or improve matching.

### Recreate the code from memory

First define a small intent dictionary, then `preprocess`, `match_intent`, `get_response`, and the input loop. The dictionary may be long but the logic is short. Explain that most of the file is data, not algorithm complexity. Run the supplied full knowledge base if that is what your examiner expects.

### Keywords, functions and expressions to recognize

A keyword is reserved language syntax. Names such as `all`, `min`, `set`, `range` and `enumerate` are built-ins; they are not Python keywords.

| Expression | What it means here |
| --- | --- |
| `import` | Keyword loading re and random. |
| `r"..."` | A raw-string literal, convenient for regular-expression backslashes. |
| `all(word in ... for word in ...)` | Require every pattern token somewhere in the input; order is ignored. |
| `return None` | Report that no intent matched. |
| `while True`, `break` | Run the conversation loop until an explicit exit. |
| `__name__ == "__main__"` | Starts interactive chat on direct execution, rather than ordinary import. |
