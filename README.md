# pyCosign

> A Pythonic façade around the 🎯 **Sigstore `cosign` CLI** — sign, attest, and verify artifacts from pure Python or the command line.

[![Build](https://img.shields.io/github/actions/workflow/status/your-org/pycosign/ci.yml?branch=main)](https://github.com/your-org/pycosign/actions)
[![License](https://img.shields.io/github/license/your-org/pycosign)](LICENSE)

---

## Table of Contents
1. [Motivation](#motivation)  
2. [Features](#features)  
3. [Quick Start](#quick-start)  
4. [CLI Examples](#cli-examples)  
5. [Library Usage](#library-usage)  
6. [Documentation](#documentation)  
7. [Contributing](#contributing)  
8. [License](#license)

---

## Motivation
`cosign` is the gold-standard CLI for software signing, but orchestration in Python projects often requires boilerplate subprocess calls, error parsing, and registry plumbing. **pyCosign** wraps those concerns in a class-based API with async support—letting you focus on *why* you sign, not *how*.

---

## Features
* 📜 **Sign, verify, attest** files or OCI artifacts
* 🔑 **Key sources**: PEM file, Fulcio keyless, HSM / PKCS #11
* 🌐 **Storage targets**: Local filesystem, OCI registry, Rekor log
* ⚡ **Async orchestration** for batch pipelines
* 📝 **Rich logging** and typed return objects (`Signature`, `VerificationResult`, `Attestation`)
* 🐍 Python 3.10+ & MIT-licensed

---

## Quick Start

```bash
# install cosign (v2.2+) first
brew install sigstore/tap/cosign        # macOS example

# install pyCosign
pip install pycosign
```

Create a signature for hello.tgz:

```bash
pycosign sign-blob --key cosign.key ./hello.tgz   # → hello.tgz.sig
```

Verify it:

```bash
pycosign verify-blob --signature hello.tgz.sig ./hello.tgz
```

---

## CLI Examples

| Task | Command |
|------|---------|
| **Sign OCI image** | `pycosign sign registry.local/hello:1.0` |
| **Sign local file → push signature to registry** | `pycosign sign-blob --push registry.local/hello:1.0 ./hello.tgz` |
| **Attest local file with SPDX SBOM** | `pycosign attest-blob --predicate sbom.json ./hello.tgz` |
| **Verify local file with detached `.sig`** | `pycosign verify-blob --signature hello.tgz.sig ./hello.tgz` |
| **Verify attestation bundle offline** | `pycosign verify-attestation --type https://spdx.dev/Document --signature hello.tgz.att ./hello.tgz` |

>See `pycosign --help` for the full option matrix.

## Library Usage

```python
from pycosign import Signer, Verifier, KeyManager

signer = Signer(key_manager=KeyManager.from_key_file("cosign.key"))
sig_obj = signer.sign_blob_local("hello.tgz")
print(sig_obj.sig_path)           # Path('hello.tgz.sig')

verifier = Verifier()
result = verifier.verify_local("hello.tgz", sig_obj.sig_path)
assert result.verified
```

Async batch signing example in `/examples/async_batch.py`.

## Documentation

* **Design Specification** - high-level architecture & diagrams  
  → [pyCosign/documentation/design_specification.md](./documentation/design_specification.md)  
* **Signing · Attesting · Verifying Primer** - conceptual deep-dive with sample artifacts
  → [pyCosign/documentation/signing-attesting-verifying.md](./documentation/signing-attesting-verifying.md)  

## Contributing
Bug reports, feature requests, and PRs are welcome! Please see `[CONTRIBUTING.md](CONTRIBUTING.md)` for the workflow, coding style, and DCO sign-off requirements.

## License
Released under the [MIT License](LICENSE).
