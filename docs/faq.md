# FAQ

## Does OpenClaiming require blockchain?

No. Claims can be stored anywhere. Blockchain anchoring is optional.

## Is this an identity system?

OpenClaiming provides primitives that identity systems can build upon.

## Can claims be revoked?

Revocation strategies depend on the application. Claims using URL-based keys can be revoked by updating the key document at that URL. Claims using inline data:key/... keys are permanently valid for as long as the signature verifies — this is intentional for use cases requiring permanent non-repudiation.

## Can multiple signatures be used?

Yes. Applications may extend the format to support multi-signature claims.

## What problem does OpenClaiming solve?

It provides a simple universal format for verifiable statements across systems without requiring centralized infrastructure.
