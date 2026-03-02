# An-Empirical-Security-Evaluation-of-LLM-Generated-Cryptographic-Rust-Code

A research pipeline that automatically generates, compiles, and security-analyzes Rust cryptographic code from three LLMs (Gemini 2.5 Pro, ChatGPT GPT-4o, DeepSeek Coder) using four prompt strategies.

---

## What It Does

1. **Generates** Rust code for AES-256-GCM and ChaCha20-Poly1305 encryption from each LLM
2. **Compiles** the code with `cargo clippy` and classifies errors (API hallucination, type error, trait error, unresolved import)
3. **Analyzes** compiled samples with CodeQL (`rust-security-extended.qls`)
4. **Analyzes** compiled samples with a custom rule-based crypto-specific analyzer

---

## Requirements

### System
- **Rust + Cargo** — [Install via rustup](https://rustup.rs)
- **CodeQL CLI** — downloaded automatically in the notebook
- **Python 3.10+**
- Designed to run on **Google Colab** (Linux environment)


### API Keys
Set the following before running:

| Variable | Model |
|---|---|
| `OPENAI_API_KEY` | ChatGPT (GPT-4o) |
| `DEEPSEEK_API_KEY` | DeepSeek Coder |
| Gemini API key | Set inline in `generate_rust_code_gemini_safe()` |

---

## Quick Start

1. Open `Main.ipynb` in Google Colab
2. Run the **Installation & Prerequisites** cells (installs Rust, CodeQL, Python packages)
3. Set your API keys
4. Run the experiment cells 

---

## Experiment Design

| Variable | Values |
|---|---|
| LLM Models | Gemini 2.5 Pro, ChatGPT (GPT-4o), DeepSeek Coder |
| Algorithms | AES-256-GCM, ChaCha20-Poly1305 |
| Prompt Strategies | Zero-shot, Constraint-based, Chain-of-thought, Security-focused |
| Samples per combination | 10 |
| **Total samples** | **240** |

All models use `temperature=0.0` and `seed=42` for reproducibility.

---

## Custom Security Analyzer Rules

The rule-based crypto-specific analyzer detects:

| Rule | CWE |
|---|---|
| Hardcoded secrets (key/nonce literals) | CWE-798 |
| Nonce reuse in loop | CWE-329 |
| Nonce reuse across multiple calls | CWE-329 |
| Static nonce (`Nonce::from_slice` with literal) | CWE-329 |
| Weak randomness (non-CSPRNG usage) | CWE-338 |
| Unsafe error handling (`unwrap()`/`expect()`) | CWE-252 |
| Key from external input without KDF | CWE-326 |
| Deprecated APIs (`NewAead`, `new_varkey()`) | CWE-327 |
| Missing secure nonce generation | CWE-330 |

---

## Output

- `model`, `algorithm`, `prompt_type`
- `compiled` (bool)
- `clippy_errors`, `clippy_warnings`
- `compilation_error_type`
- `codeql_vulnerabilities`, `codeql_warnings`
- `manual_security_issues`, `manual_security_details`

---

## Notes

- The notebook auto-retries API calls on rate limits with exponential backoff 
- CodeQL analysis runs only on successfully compiled samples 
- Dependency fixes (`hex`, `rand_core`) are injected into `Cargo.toml` automatically when detected in generated code
