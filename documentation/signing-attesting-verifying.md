# Understanding Signing, Attesting, and Verifying

This primer clarifies the three core actions in Sigstore-based supply-chain security—**signing**, **attesting**, and **verifying**—and how they relate to each other.

---

## Signing

**What it is**  
“Signing” produces a cryptographic **detached signature** (`*.sig`) that binds an artifact’s cryptographic digest (SHA-256) to a signer’s private key or keyless certificate.

**Why it matters**  
* Proves *integrity*: the artifact has not changed since it was signed.  
* Provides *authenticity*: the signature can be traced to a specific key or X.509 identity.

**Key points**  
* Implemented in Sigstore via `cosign sign` (images) or `cosign sign-blob` (files).  
* Output can be stored locally, in an OCI registry, and/or uploaded to Rekor for public transparency.  
* Keys can live in files, cloud KMS, HSM/PKCS#11, or be issued on-demand by Fulcio (keyless).

---

## Attesting

**What it is**  
“Attesting” creates a **signed statement** (`*.att`) that couples an artifact digest with **metadata**—typically supply-chain information such as an SPDX SBOM, CycloneDX BOM, or in-toto provenance.

**Why it matters**  
* Captures *how* the artifact was built (compiler flags, build environment, dependencies).  
* Enables downstream consumers to enforce policy (e.g., “only run images with SBOM and provenance”).

**Key points**  
* Implemented via `cosign attest` (images) or `cosign attest-blob` (files).  
* The metadata payload is called a **predicate**; its schema is application-specific.  
* Like signatures, attestations can be stored locally, in registries, and/or logged in Rekor.

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

**Key points**  
* Implemented via `cosign verify`, `cosign verify-blob`, and `cosign verify-attestation`.  
* Offline verification is possible using the `.bundle` file generated at signing time.  
* Verification returns a machine-readable exit code plus structured JSON output when `--output json` is used.

---

## How They Fit Together

1. **Signer** generates a detached `*.sig` layer for an artifact.  
2. **Attester** (often the same CI pipeline) attaches additional `*.att` metadata.  
3. **Verifier** runs in downstream environments (release gates, runtime policy engines) to ensure both the signature and attestation are present, valid, and policy-compliant before the artifact is trusted.
