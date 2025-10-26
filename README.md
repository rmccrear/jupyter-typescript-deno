# HOW TO: TypeScript (Deno) in Jupyter — quick setup and GitHub Models usage

This guide documents the steps we used in this workspace to:
- Install Deno
- Install and/or use JupyterLab
- Register the Deno Jupyter kernel
- Run a working GitHub Models inference request from a Deno notebook (models.github.ai)

Follow these steps in the devcontainer or your local machine. Commands assume a Bash shell.

---

## 1) Prerequisites
- Deno (we used a recent Deno release)
- Python 3 and pip (for installing Jupyter if needed)
- A GitHub token with access to the Models endpoint(s) you plan to use. Set it as an env var named `GITHUB_TOKEN`.

Keep your token private. Example (do not paste the real token in public):

```bash
export GITHUB_TOKEN="ghp_..."
```

Optionally set these environment variables to control host/model:

```bash
# models.github.ai is the inference host we used
export GITHUB_MODELS_URL="https://models.github.ai"
# choose a model (we used "openai/gpt-4.1" successfully)
export GITHUB_MODEL="openai/gpt-4.1"
```

---

## 2) Install Deno
Install Deno with the official installer (this installs to `~/.deno` by default):

```bash
curl -fsSL https://deno.land/install.sh | sh
```

After install, add Deno to your PATH (the installer can do this for you). In `~/.bashrc`:

```bash
export DENO_INSTALL="$HOME/.deno"
export PATH="$DENO_INSTALL/bin:$PATH"
```

Then reload the shell:

```bash
source ~/.bashrc
deno --version
```

---

## 3) Ensure Jupyter is available
If JupyterLab is not installed, install it with pip:

```bash
python3 -m pip install --upgrade pip
pip install jupyterlab
```

You can start JupyterLab (no browser) with:

```bash
jupyter lab --no-browser --ip=127.0.0.1 --port=8888
```

Tip: start Jupyter from the same shell where `GITHUB_TOKEN` and Deno are available so kernels inherit those env vars.

---

## 4) Register the Deno kernel for Jupyter
Deno provides a `jupyter` subcommand to install a kernelspec. Run:

```bash
deno jupyter --install --display "Deno"
```

Confirm Jupyter sees the kernel:

```bash
jupyter kernelspec list
# you should see a `deno` entry
```

If VS Code does not list the kernel, restart the Jupyter server or restart VS Code. If VS Code uses a managed Jupyter server, prefer connecting it to a local server you started (see Troubleshooting below).

---

## 5) Example: Call the working inference host (models.github.ai) from a Deno notebook
Create a TypeScript cell (Deno kernel) and paste this snippet. It uses `models.github.ai/inference/chat/completions` (this is the host that worked in our session):

```ts
// Deno kernel — call GitHub Models inference endpoint (chat completion)
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

Notes:
- We included `X-GitHub-Api-Version: 2022-11-28` because some GitHub endpoints require a supported version header. If you previously saw a 400 error complaining about the version, use `2022-11-28` (or omit the header).
- If you want to call `api.github.com/responses` instead, that can work for some accounts but may return 404 if your account/org doesn't have access to that API surface.

---

## 6) VS Code: selecting the Deno kernel in a notebook
1. Open the notebook (`.ipynb`) in VS Code.
2. Click the kernel selector in the top-right of the notebook editor (it might say "Select Kernel").
3. Choose `Deno` (or the display name you used when installing the kernelspec).
4. If the kernel isn't listed, restart the Jupyter server and try again; ensure you installed the kernelspec in the same user as the server.

Quick test cell to run in the Deno kernel:

```ts
console.log("Deno version:", Deno.version);
console.log("GITHUB_TOKEN visible:", Boolean(Deno.env.get("GITHUB_TOKEN")));
```

---

## 7) Troubleshooting
- 400 with `/user` complaining about `X-GitHub-Api-Version`: use `2022-11-28` or remove the header. GitHub will report supported versions in the error.
- 404 when calling `api.github.com/responses`: token is valid but your account or Enterprise server may not expose the Responses API. Try `models.github.ai` (inference host) or check your org/account access for GitHub Models.
- `deno jupyter --install` did not add a visible kernel: run `jupyter kernelspec list` to locate the kernel directory. Backups are created under `~/.deno/.shellRcBackups/` if the installer modified shell files.
- If notebook kernels don't inherit env vars: start Jupyter from the same shell where you exported `GITHUB_TOKEN` so the kernel process inherits it.

---

## 8) Next steps / enhancements
- Add a small notebook cell with a text input widget to prompt for questions interactively.
- Add streaming support if you want token-by-token updates (requires streaming endpoint support).
- Add example wrappers that save responses to a local file or convert them to markdown cells.

If you want, I can add an interactive cell to the `notebooks/ask_github_model_deno.ipynb` notebook that prompts for a question and runs the working `models.github.ai` snippet.

---

End of guide.
