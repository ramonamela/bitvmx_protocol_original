# bitvmx_protocol

⚠️ **WARNING: This repository is a work in progress. Some challenges are missing, and the implementation is not complete.**

This project implements the main ideas presented in the bitvmx whitepaper. For more information, please refer to the [bitvmx whitepaper](https://bitvmx.org/files/bitvmx-whitepaper.pdf).

## Repository Description

This Proof of Concept (PoC) implements the protocol through HTTP servers. This ensures that all communications are encrypted and the different components can be easily deployed both in the cloud and locally.

⚠️ **WARNING:** Currently, deploying to the cloud is not advisable since the main component needs to hold some private keys during the setup phase.


## Build

To build the project, follow these steps:

1. Init the submodule BitVMX-CPU with the command `git submodule update --init --recursive`
2. Follow the instructions in the BitVMX-CPU submodule and ensure that everything works correctly. In this step, you should obtain a valid `.elf` file that will be used in the following steps.
3. Generate the commitment file for this `.elf` file, place it in `$PROJECT_ROOT/execution_files/{instruction_commitment_filename}.txt`, and set up the correct `.elf` and `.txt` names in `bitvmx_protocol_library/bitvmx_execution/services/execution_trace_generation_service.py`.
4. Run the command `docker compose build` in the root of the repository.

## Execution

0. Create the following folders in the project's root:

   a. `prover_files`

   b. `verifier_files`

   c. `execution_files` (this one can be renamed from the existing one that already contains a valid example)

1. Start both microservices:

   a. `docker compose up prover-backend`
   
   b. `docker compose up verifier-backend`
   
2. Open the prover Swagger UI at `http://0.0.0.0:8080/docs`.

3. Rename the example environment files from `.example_env_{common/prover/verifier}` to `.env_{common/prover/verifier}`.

4. Generate a setup by executing the endpoint (EP) `api/v1/setup/fund/`. This will both get some funds from the mutinynet faucet and perform the setup ceremony. At the end, you'll see the transaction that locks the funds.

5. Take the `setup_uuid` from the response and call the EP `/api/v1/input` to set the input in the prover's server.

   Note: In a real-world scenario, the program will most likely contain a STARK verifier. The light client that will compute the proof to free the funds will run independently. Once the proof is generated, it is uploaded using this EP.

6. Once the input is available, call the EP `/api/v1/next_step` with the correct `setup_uuid`. 

    Note: In a real-world scenario, the `next_step` function would be a cron job executed at regular intervals to check the blockchain for messages from the counterparty that need a response. However, to simplify the current implementation, the `/api/v1/next_step` EP is used as a trigger mechanism. Once this EP has been processed, the analogous EP in the verifier is called. This process is repeated, allowing the protocol to run much faster. Since everything is dockerized and packaged, changing this behavior is straightforward.

In the default behavior, the execution challenge gets triggered (since the verifier introduces a failure in the execution). Different failures from both parties can be generated by modifying the file `bitvmx_protocol_library/bitvmx_execution/services/bitvmx_wrapper.py`. Note that the servers need to be restarted when this file is modified to apply the changes.


## Code Structure

The project prioritizes the use of services containing a single call. Dependencies are injected in the `__init__` call, enforcing SOLID principles and hexagonal architecture. These are the main folders:

1. **bitvmx_protocol_library**
   - This folder contains the common functionalities needed by both parties. For more details, see the [bitvmx_protocol_library README](./bitvmx_protocol_library/README.md).

2. **BitVMX-CPU**
   - Submodule containing the RISC-V emulator.

3. **blockchain_query_services**
   - Services and entities used to query the different supported blockchains (Bitcoin mainnet, testnet, and mutinynet).

4. **prover_app**
   - Prover microservice.

5. **verifier_app**
   - Verifier microservice.


## Future Work

- Complete the implementation of the remaining challenges
- Add support for multiparty configuration
- Implement the timeout transaction feature
- Enable the option for child-pays-for-parent (CPFP) transactions

## References

Some publicly executed protocols can be explored in https://bitvmx-explorer.com

For example, you can find an execution challenge protocol with a SNARK verification in mainnet [here](https://bitvmx-explorer.com/protocol?network=mainnet&txid=315fdd660aec892f452791827e5e961283851df411c1132e5544505d9d855279)

It is possible to show other examples by changing the network and emptying the input box.
