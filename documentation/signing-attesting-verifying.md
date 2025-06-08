# Understanding Signing, Attesting, and Verifying

This primer clarifies the three core actions in Sigstore-based supply-chain security — **signing**, **attesting**, and **verifying** — and how they relate to each other. Below we cover a Sigstore / _cosign_ workflow and shows concrete, copy-paste-ready examples for each action.  

---

## Table of Contents
1. [Signing](#signing)  
2. [Attesting](#attesting)  
3. [Verifying](#verifying)  

---

## Signing

**What it is**  
“Signing” produces a cryptographic **detached signature** (`*.sig`) that binds an artifact’s cryptographic digest (SHA-256) to a signer’s private key or keyless certificate.

**Why it matters**  
* Proves *integrity*: the artifact has not changed since it was signed.  
* Provides *authenticity*: the signature can be traced to a specific key or X.509 identity.

**Key points**  
* `cosign sign` → images | `cosign sign-blob` → files  
* Signatures can be stored on disk, in an OCI registry, and/or uploaded to Rekor for public transparency.  
* Keys can live in files, cloud KMS, HSM/PKCS#11, or be issued on-demand by Fulcio (keyless).

---

### Examples

#### S-1 — Sign a local file, save `.sig` locally
```bash
cosign sign-blob --key cosign.key ./hello.tgz > hello.tgz.sig
```
`hello.tgz.sig` (truncated):  
```bash
MEUCIQDn4Qv7lF1pRc0X...lWbQIoimLxEcQIgI4lR+Q5e6b6w...
```

#### S-2 — Sign an OCI image already in a registry
```bash
cosign sign --key cosign.key registry.local/hello:1.0
```
The registry now contains:
```bash
sha256:<digest>          (manifest)
└── sha256:<digest>.sig  (signature layer)
```
#### S-3 — Keyless sign a local file (OIDC + Fulcio)
```bash
# environment must have $COSIGN_EXPERIMENTAL=1 and $OIDC_TOKEN
cosign sign-blob --keyless ./hello.tgz > hello.tgz.sig
```

## Attesting

**What it is**  
“Attesting” creates a detached **signed statement** (`*.att`) that couples an artifact digest with **metadata** — typically supply-chain information such as an SPDX SBOM, CycloneDX BOM, or in-toto provenance.

**Why it matters**  
* Captures *how* the artifact was built (compiler flags, build environment, dependencies, source).  
* Enables downstream consumers to enforce policy (e.g., “only run images with SBOM and provenance”).

**Key points**  
* `cosign attest` → images | `cosign attest-blob` → files  
* The metadata payload is called a **predicate**; its schema is application-specific (SPDX, CycloneDX, in-toto, custom JSON).  
* Like signatures, attestations can be stored locally, in registries, and/or logged in Rekor for public transparency.  

---

### Examples

#### Predicate file (`sbom.json`)

```json
{
  "predicateType": "https://spdx.dev/Document",
  "predicate": {
    "SPDXID": "SPDXRef-DOCUMENT",
    "name": "hello-archive",
    "packages": [
      { "SPDXID": "SPDXRef-Pkg-1", "name": "libfoo", "versionInfo": "1.2.3" }
    ]
  }
}
```

#### A-1 — Attest a local file, save `.att` locally

```bash
cosign attest-blob \
  --key cosign.key \
  --predicate sbom.json \
  ./hello.tgz > hello.tgz.att
```
`hello.tgz.att` (truncated DSSE envelope):
```bash
eyJhc3NlcnRpb24iOnsidHlwZSI6ImNoYXJnZSIsInJlZGF0Y...
```

#### A-2 — Attest an OCI image & push to registry

```bash
cosign attest \
  --key cosign.key \
  --predicate cyclonedx.json \
  registry.local/hello:1.0
```
The registry now stores:
```bash
sha256:<digest>.att   (attestation layer)
```

#### A-3 — Rekor-only attestation (no registry push)

```bash
cosign attest-blob --key cosign.key \
  --predicate provenance.json \
  --bundle ./hello.tgz > hello.tgz.att
```
A `.bundle` inclusion proof is embedded for offline verification.

---

## Verifying

**What it is**  
“Verifying” checks that a detached signature or attestation:

1. Cryptographically matches the artifact’s digest.  
2. Chains back to a trusted root (key, Fulcio CA, or hardware-backed key).  
3. Optionally appears in Rekor (proving it hasn’t been hidden or altered).  
4. (Optional) Satisfies policy filters—certificate identity, issuer, annotations, etc.

**Why it matters**  
* Prevents tampered or untrusted artifacts from entering production.  
* Enables automated policy gates in CI/CD pipelines and Kubernetes admission controllers.
* Provides audit evidence for compliance (SLSA, NIST SSDF).

**Key points**  
* `cosign verify(-blob)` → signatures | `cosign verify-attestation` → attestations
* Offline verification is possible using the `.bundle` file generated at signing time.  
* Verification returns a machine-readable exit code plus structured JSON output when `--output json` is used.

---

## How They Fit Together

1. **Signer** generates a detached `*.sig` layer for an artifact.  
2. **Attester** (often the same CI pipeline) attaches additional `*.att` metadata.  
3. **Verifier** runs in downstream environments (release gates, runtime policy engines) to ensure both the signature and attestation are present, valid, and policy-compliant before the artifact is trusted.

---

### Examples

#### V-1 — Verify local file with detached `.sig`

```bash
cosign verify-blob \
  --signature hello.tgz.sig \
  ./hello.tgz
```
Output: Verified OK (exit 0)

#### V-2 — Verify OCI image signature in registry

```bash
cosign verify registry.local/hello:1.0
```
JSON output (truncated):
```json
{
  "critical": { "...": "..." },
  "optional": { "tlogIndex": 654321 }
}
```

#### V-3 — Verify attestation bundle offline

```bash
cosign verify-attestation \
  --type https://spdx.dev/Document \
  --signature hello.tgz.att \
  ./hello.tgz
```
Output:
```json
{"verified":true,"tlogIndex":54321,"attestation_type":"spdx"}
```

## Putting It Together

| Artifact                 | Produced by        | Purpose                                             |
|--------------------------|--------------------|-----------------------------------------------------|
| `hello.tgz`              | build step         | Primary binary blob                                 |
| `hello.tgz.sig`          | Signer             | Detached integrity/authenticity proof               |
| `sbom.json`              | build step         | Human-readable SBOM                                 |
| `hello.tgz.att`          | Attester           | Signed DSSE envelope of SBOM                        |
| `bundle.json` (optional) | Signer / Attester  | Rekor inclusion proof for offline verification       |

Armed with these artifacts, any consumer (human or pipeline) can reproduce the verifying commands above to establish full trust in both the artifact and its supply-chain metadata.
