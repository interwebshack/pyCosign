# pyCosign Design Specification  

> **Version 0.1 – June 2025**  
> This living document captures the architecture, roles, and primary use‑cases for _pyCosign_ — a Python façade around the `cosign` executable and Sigstore services.  

## Table of Contents
1. [Purpose & Scope](#purpose--scope)  
2. [Reference Architecture](#reference-architecture)  
3. [Role Overview](#role-overview)  
4. [Signer](#signer)  
5. [Verifier](#verifier)  
6. [Attester](#attester)  
7. [Glossary](#glossary)

## Purpose & Scope
_pyCosign_ offers a **thin, class-based wrapper** that orchestrates the trusted `cosign` CLI while hiding subprocess and storage details from application developers. The project supports:

* Detached signatures & attestations for any local file  
* OCI-native signatures/attestations for container images and other OCI artifacts  
* Rekor transparency-log integration (optional or offline bundle)  
* Multiple key sources (file, Fulcio keyless, HSM/PKCS#11)

> ⚠️ This document intentionally limits itself to the _foundational_ use-cases required in sprint 1. More advanced flows (HSM, batch, policy-verify) appear only as placeholders.


