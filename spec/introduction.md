## Introduction

Similar to the [W3C Verifiable Credentials Data Model](https://www.w3.org/TR/vc-data-model/) or the “Trust Triangle” of the [Trust over IP Model](https://trustoverip.org/wp-content/toip-model/), the AnonCreds credential protocol follows the model of [[ref: Verifiable Credential] and [[ref: Verifiable Presentation]]. Verifiable Presentations used in the AnonCreds protocol are just-in-time synthetizations from one or several Verifiable Credentials that do not contain the original credential or signatures and can be presented to third parties for validation. 

AnonCreds ZKP verifiable credentials provide capabilities that many see as important for digital identity use cases in particular, and verifiable data in general. These features include:

- A full implementation of the Layer 3 verifiable credential “Trust Triangle” of the [Trust over IP Model](https://trustoverip.org/wp-content/toip-model/).
- Complete flows for issuing verifiable credentials (Issuer to Holder), and requesting, generating and verifying presentations of verifiable claims (Holder to Verifier).
- Fully defined data models for all of the objects in the flows, including verifiable credentials, presentation requests and presentations sourced from multiple credentials.
- Fully defined applications of cryptographic primitives.
- The use of Zero Knowledge Proofs (ZKPs) in the verifiable presentation process to enhance the privacy protections available to the holder in presenting data to verifiers, including:
  - Blinding issuer signatures to prevent correlation based on those signatures.
  - The use of unrevealed identifiers for holder binding to prevent correlation based on such identifiers.
  - The use of predicate proofs to reduce the sharing of PII and potentially correlating data, especially dates (birth, credential issuance/expiry, etc.).
  - A revocation scheme that proves a presentation is based on credentials that have not been revoked by the issuers without revealing correlatable revocation identifiers.

The AnonCreds v0.1 Specification matches the existing [Hyperledger Indy SDK (“libindy”)](https://github.com/hyperledger/indy-sdk/blob/master/libindy/src/api/anoncreds.rs) and [Indy Credential Exchange (“cred-x”)](https://github.com/hyperledger/indy-shared-rs/tree/main/indy-credx) implementations.

The next version of the specification (tentatively v1.0) will remove from the v0.1 specification any dependence on Hyperledger Indy by removing any requirements related to the storage of the objects used in AnonCreds, whether they be stored remotely on a “verifiable data registry” (including Hyperledger Indy) or in local secure storage.
