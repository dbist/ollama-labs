
Ollama itself is a "raw" engine. It does not have built-in parental controls or a "child mode" switch. By default, it will process whatever prompt it is given and output whatever the specific model (like Llama 3 or Mistral) was trained to say.

To make Ollama safe for a child, you need to implement a **multi-layered approach.**

---

### 1. Create a Custom "Child-Safe" Model (Modelfile)
The easiest way to add guards within Ollama is to create a custom model based on an existing one, but with a strict **System Prompt**.

1.  Create a file on your computer named `Modelfile`.
2.  Paste the following content (using `llama3` as an example):

```dockerfile
FROM llama3

# Set the safety parameters
PARAMETER temperature 0.5
PARAMETER stop "Child:"

# Set the System Prompt (The Guard)
SYSTEM """
You are a safe, educational, and friendly assistant for a 10-year-old child. 
1. Use simple language appropriate for a child.
2. If the user asks about violence, adult content, self-harm, or illegal activities, politely refuse and suggest a positive alternative topic.
3. Do not give medical, legal, or financial advice.
4. Keep answers concise and encouraging.
5. If you are unsure of a fact, tell the child to 'ask a grown-up' to help verify it.
"""
```

3.  Open your terminal in that folder and run:
    `ollama create safe-llama -f Modelfile`
4.  Tell your child to only use the model named **"safe-llama."**

---

### 2. Use a "Guard" Model (The Referee)
Meta released a specific model called **Llama Guard** designed to check if inputs or outputs are safe. 

If you are a developer or using a tool like **Open WebUI**, you can set up a pipeline where:
1.  The child's question is sent to `llama-guard3`.
2.  If `llama-guard3` says "Safe," the question goes to the main model.
3.  If it says "Unsafe," the system blocks the response.

---

### 3. Use a Frontend with User Management (Recommended)
Ollama is just a command-line tool. To truly manage a "Child Account," you should run a frontend like **Open WebUI** (formerly Ollama WebUI).

Open WebUI allows you to:
*   **Create separate accounts:** You can have an Admin account (you) and a User account (your child).
*   **Model Access:** You can restrict the child's account so they **only** see the "safe-llama" model you created in Step 1.
*   **Global System Prompts:** You can force a system prompt on all users that prevents inappropriate content.
*   **Functions/Filters:** Open WebUI has a "Functions" community where you can download "Filters" that automatically scrub bad words or toxic content before it reaches the child.

---

### 4. Choose "Aligned" Models (Avoid "Uncensored")
When downloading models from the Ollama library, pay attention to the names:
*   **Safe:** Stick to official releases like `llama3`, `gemma2`, or `mistral`. These have been "RLHF-aligned," meaning the creators trained them to refuse harmful requests.
*   **Unsafe:** Avoid any models with **"Uncensored," "Dolphin,"** or **"Abolished"** in the name. These have had their safety guards intentionally removed.

---

### 5. Network-Level Monitoring
Since Ollama runs locally, standard internet filters (like Disney Circle or NetNanny) usually **cannot** see what is happening inside the chat because the data isn't leaving your computer.

*   **Log Checking:** You can periodically check the history if you are using a frontend like Open WebUI.
*   **Terminal Logs:** If they use the terminal, you can check the command history, but this is easily deleted.

### Summary Checklist for Parents:
1.  **Don't use the terminal:** Use **Open WebUI** for a child-friendly interface.
2.  **Create a "Safe-Llama" custom model:** Use the `Modelfile` instructions above.
3.  **Lock it down:** In Open WebUI, disable the child's ability to "pull" new models so they can't download uncensored ones.
4.  **Co-use:** Sit with them for the first few sessions to see how the model reacts to their specific questions.

### The Modelfile

```dockerfile
# 1. Choose the base model (Gemma is Google's open model for Ollama)
FROM gemma2:2b

# 2. Set parameters to keep the AI focused and not too "random"
PARAMETER temperature 0.6
PARAMETER top_p 0.9
PARAMETER stop "<start_of_turn>"
PARAMETER stop "<end_of_turn>"

# 3. The System Prompt - This is where the magic happens
SYSTEM """
You are a friendly, patient, and cheerful AI companion for a child. 
Your name is "Buddy".

Follow these rules strictly:
1. Language: Use simple words that a 7-year-old can understand. Avoid complex jargon.
2. Tone: Be encouraging, kind, and curious. Use emojis occasionally to be fun! 🌟
3. Safety: 
   - Never use bad words or talk about scary, violent, or "grown-up" topics.
   - If the child asks about something dangerous (like fire or sharp objects), tell them: "That sounds like a question for a grown-up! Why don't you go ask your mom, dad, or teacher?"
   - Do not give medical or legal advice.
4. Educational: If the child asks a "why" question, explain it using a fun analogy.
5. Conciseness: Keep your answers short (2-4 sentences) so the child doesn't get bored.
6. Privacy: Never ask the child for their real name, address, or school name.
"""

# 4. Optional: A "First Message" to set the mood
MESSAGE assistant "Hi there! I'm Buddy. I'm so happy to talk to you! What's your favorite animal, or do you want to hear a fun fact?"
```

---

### How to set this up:

1.  **Create the file:** Save the code above as a file named `Modelfile` (no file extension) in a folder.
2.  **Open your Terminal/Command Prompt** in that folder.
3.  **Build the model:** Run this command:
    ```bash
    ollama create buddy -f Modelfile
    ```
4.  **Run the model:**
    ```bash
    ollama run buddy
    ```

### Why these specific settings?

*   **Model choice (`gemma2:2b`):** The 2-billion parameter model is smaller and faster. It’s "smarter" than older small models but lightweight enough to run on most home computers.
*   **Temperature (0.6):** A lower temperature makes the AI more predictable. If it’s too high (like 1.0), the AI might start making up nonsense or get too "weird" for a child.
*   **The System Prompt:** 
    *   **The "Grown-up" clause:** This is a crucial safety feature. It prevents the AI from trying to handle topics it isn't qualified for.
    *   **Simplicity:** By telling the AI to talk to a 7-year-old, it will naturally use smaller words and shorter sentences.
    *   **The "Buddy" Persona:** Giving the AI a name and a goal (being a companion) helps the child feel comfortable.

### Note on "Gemini" vs "Gemma"
Ollama cannot run the official **Google Gemini** (Pro/Ultra) because those are closed-source models kept on Google's servers. **Gemma** is the "local" version built by Google using the same technology, and it is the closest you can get to running Gemini locally on your own computer.