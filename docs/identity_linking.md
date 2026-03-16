# Identity Linking

Users frequently maintain identities across multiple platforms. OpenClaiming allows these identities to be cryptographically linked.

A claim may assert that two identifiers belong to the same entity.

Example:

A GitHub account signing a claim stating ownership of a domain.

Example statement:

{
  "github": "alice",
  "domain": "example.com"
}

If the domain publishes the same claim signed by its own key, verifiers gain strong evidence that the two identities are controlled by the same entity.

## Cross-Domain Trust

Identity linking enables decentralized identity verification without requiring centralized identity providers.

Multiple services can independently verify relationships using claims.
