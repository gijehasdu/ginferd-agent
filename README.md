# ginferd-agent

Proof Gatekeeper agent for the zkVerify Agent Games (Season 01), competing as **ginferd**.

## How it plays

The platform POSTs signed tasks to the agent's HTTPS endpoint. For every task the agent:

1. Verifies the `x-agent-games-signature` HMAC-SHA256 over the raw body before parsing.
2. Runs the decision checklist in order: proof reuse (persistent seen-set), VK substitution,
   threshold, expiry, recipient, amount cap.
3. Decides APPROVE / REJECT with a reason code — fail-closed: never APPROVE without a proof.
4. Generates a Groth16 proof of its own decision locally with snarkjs, using the official
   `gatekeeper` circuit artifacts (downloaded from the platform's `/api/circuit-info`),
   self-verifies it, and returns it with the signed response.
5. Persists every answer by `task_id`, so platform retries get byte-identical replies.

## Run

```sh
npm install
mkdir -p circuits
curl -o circuits/gatekeeper.wasm https://agentgames.zkverify.io/circuits/gatekeeper.wasm
curl -o circuits/gatekeeper_final.zkey https://agentgames.zkverify.io/circuits/gatekeeper_final.zkey
AGENT_GAMES_HMAC_SECRET=... node agent.mjs   # listens on 127.0.0.1:8787, POST /agent
```

The endpoint must be reachable over public HTTPS; any tunnel or reverse proxy in front of
the local port works.
