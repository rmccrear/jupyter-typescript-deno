
# 🚀 LangChain + GitHub Models (TypeScript / Deno / Node)

This project shows how to use **LangChain** with **GitHub Models** (like `gpt-4o`, LLaMA, etc.) using only **minimal configuration changes** — no custom provider required.

---

## ✅ Features

✔ Use GitHub Models via LangChain’s `ChatOpenAI`
✔ Supports tool calling (e.g. PokéAPI)
✔ Works in **TypeScript**, **Node.js**, or **Deno**
✔ No Azure or OpenAI API key required — just your `GITHUB_TOKEN`

---

## 📦 Install

```bash
npm install @langchain/openai @langchain/core zod dotenv
```

For Deno:

```ts
// You can import directly:
import { ChatOpenAI } from "npm:@langchain/openai";
```

---

## 🔑 Environment Variables (`.env`)

```
GITHUB_TOKEN=ghu_xxxxxxxxxxxxxxxxxxxxxxxxxxxx   # Required (with models:read)
GITHUB_MODEL=openai/gpt-4o                      # Optional
```

---

## 💬 Minimal Example (LangChain + GitHub Models)

```ts
import "dotenv/config";
import { ChatOpenAI } from "@langchain/openai";

const llm = new ChatOpenAI({
  apiKey: process.env.GITHUB_TOKEN,
  model: process.env.GITHUB_MODEL ?? "openai/gpt-4o",
  temperature: 0.2,
  configuration: {
    baseURL: "https://models.github.ai/inference",
    defaultHeaders: {
      "X-GitHub-Api-Version": "2022-11-28",
      Accept: "application/json",
    },
  },
});

const res = await llm.invoke("Explain LangChain in 1 sentence.");
console.log(res);
```

---

## 🛠 Example with Tool Calling (PokéAPI)

Full example in `langchain-pokeapi-github.ts`:

✔ Defines a `get_pokemon` tool using PokéAPI
✔ Uses GitHub Models to decide when to call it
✔ Returns a natural language response

---

## ⚠️ Troubleshooting

| Error                                     | Cause                                      | Fix                                                   |
| ----------------------------------------- | ------------------------------------------ | ----------------------------------------------------- |
| `401 Incorrect API key provided`          | GitHub token sent to OpenAI by mistake     | Make sure `configuration.baseURL` and headers are set |
| `tool({...}) schema undefined error`      | Wrong LangChain version or wrong signature | Use `tool(func, { name, description, schema })`       |
| No response, just `Promise { <pending> }` | Forgot `await`                             | Add `await llm.invoke(...)`                           |

---

## ✅ Why This Is Cool

✔ No need for Azure or OpenAI keys
✔ GitHub-hosted models (GPT-4o, LLaMA, Mistral, etc.)
✔ Fully compatible with LangChain’s chains, tools, agents, memory, etc.
✔ Works in Codespaces, Deno, or local Node projects

