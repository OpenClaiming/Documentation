# Intercloud Claims

OpenClaiming can be used to exchange verifiable statements across distributed systems.

In large distributed environments, systems must often accept information produced by other nodes. OpenClaims allow those statements to be verified cryptographically.

## System State Claims

Nodes may publish claims describing:

- configuration state
- policy decisions
- resource availability
- event logs

## Chilling Effects in Distributed Systems

The Intercloud model relies on signed statements forming a causal history of what nodes claim to have observed.

Each node signs claims referencing information received from other nodes. Because these claims are signed and may later be cross-verified, dishonest statements become provably detectable.

This creates a "chilling effect": participants behave honestly because false claims can be cryptographically exposed later.

Rather than relying on centralized consensus or voting, systems rely on the threat of provable misbehavior.

## Advantages

- strong auditability
- scalable distributed trust
- reduced reliance on global consensus mechanisms
