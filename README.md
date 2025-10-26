# 🦕 Deno + Jupyter Dev Container

This repository provides a ready-to-use **VS Code Dev Container** that lets you write and run **Deno** code directly inside Jupyter Notebooks (`.ipynb`).
It automatically installs:

✅ Deno
✅ Deno Jupyter kernel
✅ Jupyter VS Code extensions
✅ Example Deno notebook using GitHub Models API

---

## 🚀 Getting Started

1. **Open this repository in VS Code**
2. When prompted, click **“Reopen in Dev Container”**
3. After the container builds, open `example-deno-notebook.ipynb` (If it does not render properly, wait for the container to complete its loading phase and try again)
4. Select **Kernel → Deno**
5. Run cells!

---

## 📁 Project Structure

```
.
├── .devcontainer/
│   └── devcontainer.json
├── example-deno-notebook.ipynb
└── README.md
```

---

## ⚙️ Dev Container Configuration

The Dev Container installs Deno and registers it as a Jupyter kernel:

```jsonc
{
  "name": "deno-jupyter",
  "image": "mcr.microsoft.com/devcontainers/base:latest",

  "remoteUser": "vscode",

  "remoteEnv": {
    "DENO_INSTALL": "/home/vscode/.deno",
    "PATH": "${containerEnv:PATH}:/home/vscode/.deno/bin"
  },

  "onCreateCommand": "bash -lc \"set -e; curl -fsSL https://deno.land/install.sh | sh -s -- -y; /home/vscode/.deno/bin/deno jupyter --install --display 'Deno'\"",

  "customizations": {
    "vscode": {
      "extensions": [
        "ms-toolsai.jupyter",
        "ms-toolsai.jupyter-renderers",
        "ms-toolsai.jupyter-keymap"
      ]
    }
  }
}
```

---

## 🔐 Environment Variables

This example uses the **GitHub Models API**.
It requires authentication using `GITHUB_TOKEN`.

### ✅ In GitHub Codespaces

**GitHub Codespaces automatically provides a `GITHUB_TOKEN`**, so most users won’t need to configure anything.

You can check by running:

```bash
echo $GITHUB_TOKEN
```

If it prints a token value, you're all set.

---

### 🛠 If You’re Running Locally or the Token is Missing

You can manually provide `GITHUB_TOKEN` in several ways:

**Option 1 – `devcontainer.json`**

```json
"containerEnv": {
  "GITHUB_TOKEN": "your_token_here"
}
```

**Option 2 – `.env` file**

```
GITHUB_TOKEN=your_token_here
```

**Option 3 – Terminal (before launching VS Code)**

```bash
export GITHUB_TOKEN=your_token_here
```

Other optional variables:

| Variable            | Default                    | Purpose               |
| ------------------- | -------------------------- | --------------------- |
| `GITHUB_MODEL`      | `openai/gpt-4.1`           | Model to use          |
| `GITHUB_MODELS_URL` | `https://models.github.ai` | API endpoint override |

---

## 📓 Example Notebook Code (`example-deno-notebook.ipynb`)

```ts
const token = Deno.env.get("GITHUB_TOKEN");
if (!token) throw new Error("GITHUB_TOKEN environment variable is not set.");

const apiBase = Deno.env.get("GITHUB_MODELS_URL") ?? "https://models.github.ai";
const endpoint = `${apiBase}/inference/chat/completions`;

const model = Deno.env.get("GITHUB_MODEL") ?? "openai/gpt-4.1";
const question = "What is the capital of France?";

const payload = {
  model,
  messages: [{ role: "user", content: question }]
};

const resp = await fetch(endpoint, {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${token}`,
    "Accept": "application/json",
    "Content-Type": "application/json",
    "X-GitHub-Api-Version": "2022-11-28"
  },
  body: JSON.stringify(payload),
});

if (!resp.ok) {
  const text = await resp.text();
  console.error(`Request failed (${resp.status}): ${text}`);
} else {
  const json = await resp.json();
  const reply =
    json.choices?.[0]?.message?.content ??
    json.choices?.[0]?.content ??
    json.output_text ??
    JSON.stringify(json, null, 2);
  console.log("Model reply:\n", reply);
}
```

---

## ✅ Example Output

```
Model reply:
Paris is the capital of France.
```

---

## 💡 Tips

* If the **Deno kernel doesn’t appear**, run:
  **Command Palette → Developer: Reload Window**
* To create a new Notebook, run:  **Command Palette → Create: New Jupyter Notebook**
* To view installed kernels:

  ```bash
  ls ~/.local/share/jupyter/kernels
  ```
* To upgrade Deno:

  ```bash
  deno upgrade
  ```
