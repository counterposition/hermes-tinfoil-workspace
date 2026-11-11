# Hermes Agent workspace on Tinfoil

A [Tinfoil Container](https://docs.tinfoil.sh/containers/overview) for
[Hermes Agent](https://github.com/NousResearch/hermes-agent) to work in. With
the [`tinfoil-sandbox`](https://github.com/counterposition/tinfoil-hermes-provider)
backend selected, Hermes' terminal, file and code tools run in this Ubuntu
workspace instead of on your machine. The workspace runs in a hardware enclave
that neither the cloud host nor Tinfoil can look into, and every connection
checks the enclave's attestation first.

Pair it with the `tinfoil-provider` plugin and both the model and the tools
run in enclaves.

## Set it up

You need a Tinfoil organization with Containers enabled, an organization admin
key, the [Tinfoil CLI](https://docs.tinfoil.sh/containers/cli) and an
inference key (`tk_...`).

1. **Use this template** → **Create a new repository**. Keep it public: the
   config and its release evidence are what clients verify.
2. In `tinfoil-config.yml`, replace the key under `SSH_KEYS` with your SSH
   public key (`cat ~/.ssh/id_ed25519.pub`).
3. Publish a measured release:

   ```bash
   gh workflow run tinfoil-release.yml -f version=v0.1.0
   ```

4. Create the container once both release workflows have finished:

   ```bash
   tinfoil container create hermes-workspace --repo <you>/<your-repo> --tag v0.1.0
   ```

5. Point Hermes at it:

   ```bash
   hermes plugins install counterposition/tinfoil-hermes-provider/sandbox
   ```

   ```bash
   hermes config set terminal.tinfoil_container hermes-workspace
   ```

   ```bash
   hermes config set terminal.backend tinfoil
   ```

   Hermes reads `TINFOIL_API_KEY` (an inference key) to open the tunnel.

## What's in it

- Tinfoil's [confidential-ubuntu](https://github.com/tinfoilsh/confidential-ubuntu)
  image, pinned by digest: Ubuntu 24.04 with Python 3, git, curl and sshd.
- 2 CPUs and 8 GB of memory. Raise `cpus` and `memory` in the config and
  release a new version if your agent needs more.
- Open internet access, so the agent can install packages and clone
  repositories.
- No persistent disk: files live in the enclave's memory and are gone after a
  restart. Tinfoil's
  [encrypted volumes](https://docs.tinfoil.sh/containers/encrypted-volumes)
  add one.

Built from Tinfoil's
[containers template](https://github.com/tinfoilsh/tinfoil-containers-template).
