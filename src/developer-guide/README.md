# Developer Guide

(WIP)

Not including requirements/basic dev environment set up here like the requirement for a specific version of Rust to have been installed - that may go better in the README.md of https://github.com/anoma/anoma

## Setting up a local chain

1. make a simple network config
2. `anomac utils init-network`
3. `anomac utils join-network`
4. start your ledger node
5. make a test transaction to ensure things work

## Writing a custom transaction
How to write a simple tx, compile the WASM blob and then submit it to your local chain.

## Writing a custom validity predicate

## Running tests
Details of how to run tests (especially E2E tests).
St