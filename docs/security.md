# Security Considerations

OpenClaiming security depends primarily on correct cryptographic verification.

## Mandatory Checks

Verifiers must confirm:

- canonicalization correctness
- signature validity
- issuer public key authenticity
- expiration timestamps
- claim integrity

## Key Compromise

If a private key is compromised, attackers may produce valid claims. Systems should support key rotation and revocation policies.

## Replay and Context

Claims should be interpreted within their context. Applications may add additional checks such as nonce validation or domain binding. Replay protection is an application concern. Systems that require nonce or sequence guarantees should include them in the stm payload, which is covered by the signature.

## Transport Security

Although claims remain verifiable regardless of transport, secure transport such as HTTPS prevents tampering and improves reliability.
