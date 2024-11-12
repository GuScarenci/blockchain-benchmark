# Using HyperLedger Caliper to benchmark of Blockchain

Most of the files in this repository were not written by me. My goal with this project was to compile tools for testing blockchain benchmarking. During the project’s development, I created tutorials for areas I felt were insufficiently covered in the original documentation. Additionally, I compiled links and files that streamlined the project execution process, which is what you’ll find here.

# Useful (Essential) Links

## Caliper Documentation Links
- Getting Started: https://hyperledger-caliper.github.io/caliper/v0.6.0/getting-started/
- Installing Caliper: https://hyperledger-caliper.github.io/caliper/v0.6.0/installing-caliper/#the-caliper-cli
- Paper Explaining Blockchain Benchmarking: https://www.lfdecentralizedtrust.org/learn/publications/blockchain-performance-metrics

## Caliper Benchmark GitHub Links
- Caliper Benchmarks GitHub: https://github.com/hyperledger-caliper/caliper-benchmarks
- Caliper Benchmarks GitHub page with a tutorial on how to run some Fabric Sample Tests: https://github.com/hyperledger-caliper/caliper-benchmarks/blob/main/networks/fabric/README.md
- Caliper Benchmarks GitHub page with a tutorial on how to run some Besu Samples Testes: https://github.com/hyperledger-caliper/caliper-benchmarks/tree/main/networks/besu/1node-clique

## Fabric Links 
- Getting Started Fabric: https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html
- (Extra) Deploying a Production Network: https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html

## Besu Links
- Getting Started Besu: https://besu.hyperledger.org/stable/private-networks/get-started/install
- Miguel's (SSC0958 - Criptomoedas e Blockchain (2024) Monitor) Video.

# My Step By Step

## Sample Benchmarking on Fabric
Assuming a totally new enviroment

When following the step below and on the suggested links, be aware of the version between softwares and compatibilities between them. **In this case, it is very important to pay attention to the Fabric Docker Version and the Caliper Bind command**

1. Go to **Getting Started Fabric Link** and follow the steps to install Fabric and clone the samples repository, follow it until you get to step of starting the `test-network`. 
2. Go to **Installing Caliper Link** and follow the steps to install Caliper, Local NPM installation recommended.
3. On **Caliper Benchmarks GitHub page with a tutorial on how to run some Fabric Sample Tests Link** follow to steps to perform your first **benchmarking on Fabric** Test Network. I used the Fabcar example. 
4. You will see your benchmark result on `reports.html`.

## Sample Benchmarking on Besu

When following the step below and on the suggested links, be aware of the version between softwares and compatibilities between them. **In this case, it is very important to pay attention to the Besu Version and the Caliper Bind command**. It is worth nothing that the documentation for the Fabric use for Caliper is much better and does much more for you than the Caliper one,so when setting up for Besu on Caliper you will need to do much more things  like changing specific adresses on code to your specific use, at least in my experience.

1. Follow the **Miguel's Video** and **Getting Started Besu Link** until the step where you downloaded Besu, do not start the network showed in the tutorial yet, just download Besu.
2. Now that you have Besu installed, create a directory for the project and clone the Caliper Benchmarks Repository:
```bash
mkdir besu-erc20-benchmark
cd besu-erc20-benchmark
git clone https://github.com/hyperledger/caliper-benchmarks 
cd caliper-benchmarks
```
3. **Navigate to the Besu Network Directory**:
```bash
cd networks/besu/1node-clique
```
4. **Modify docker-compose.yml:** Update the `command` section to include the HTTP RPC flags. Here’s how your `command` section should look:
```yaml
command: 
	- --genesis-file=/root/genesis.json
    - --node-private-key-file=/root/.ethereum/keystore/key
    - --min-gas-price=0 
    - --revert-reason-enabled 
    - --rpc-http-enabled
    - --rpc-http-host=0.0.0.0
    - --rpc-http-port=8545
    - --rpc-http-cors-origins=*
    - --rpc-http-apis=ADMIN,ETH,MINER,WEB3,NET,PRIV,EEA
    - --rpc-ws-enabled 
    - --rpc-ws-host=0.0.0.0 
    - --host-allowlist=* 
    - --rpc-ws-apis=ADMIN,ETH,MINER,WEB3,NET,PRIV,EEA 
    - --graphql-http-enabled
    - --discovery-enabled=false
```

5. **Start the Besu Network**: This command will spin up the network as specified in the `docker-compose.yml`:
```bash
docker-compose up -d
```

Now that the Sample **1node-clique network** is running, we need to Deploy a contract on it. Lets start by deploying the ERC-20.sol on it using the Remix IDE. 

6. First install MetaMask and add your Localhost network to it, if you did the steps above in the default way, you should connect to your local network on Metamask like shown below:
   ![[Pasted image 20241110150754.png]] 
7. On RemixIDE, connect to your Local Network through Metamask by using **Injected Provider - Metamask** on Enviroment on the "Deploy & run transactions" section.
8. Copy the ERC-20.sol code that is on the `caliper-benchmarks/src/ethereum/ERC-20` folder in your cloned caliper-benchmark repository and paste it on the RemixIDE file explorer on the src folder, I had to change a few things in the sample code for it to work, I did with the help of Remix correction, and change the compiler, then finally I could compile.
9. Deploy the ERC-20 on the Network, in this case your Local Network.
10. Now we will finally Benchmark this contract in this Network. Navigate to the root folder of the caliper-benchmarks repository.
11. Bind the Caliper to the besu, and remember to unbind to other Tool that you've used before, for example:
```bash
npx caliper unbind --caliper-undbind- npx caliper unbind --caliper-bind-sut besu:latest
sut fabric:2.5
npx caliper unbind --caliper-bind-sut besu:latest
```
12. Update `erc20networkconfig.json` with the Deployed Contract Address, get this Contract Adress on RemixIDE.
13. Update `ERC-20.json` that is on the `caliper-benchmarks/src/ethereum/ERC-20` folder with the new **abi** and **bytecode** generated by the **Solidity Compiler** on the Remix IDE, remember to add "0x" in the beggining of the **bytecode** string. 
14. If all went right, you should be able to benchmark ERC-20 Smart Contract on your 1node-clique local network by using:
```bash
npx caliper launch manager \
    --caliper-benchconfig benchmarks/scenario/ERC-20/config.yaml \
    --caliper-networkconfig networks/besu/1node-clique/erc20networkconfig.json \
    --caliper-workspace .
```
15. You will see your benchmark result on `reports.html`.

# Extra information
Here are some things I learned along the way that I thought would be useful to include here:

## Caliper command explanation:

```bash
npx caliper launch manager --caliper-workspace ./ --caliper-networkconfig networks/fabric/test-network.yaml --caliper-benchconfig benchmarks/samples/fabric/fabcar/config.yaml --caliper-flow-only-test --caliper-fabric-gateway-enabled
```

This command runs Caliper, a benchmarking tool for blockchains, and sets up a specific test on the Hyperledger Fabric network.

- `npx caliper launch manager`:

`npx` is a Node.js utility that runs packages without requiring a global installation. Here, it calls Caliper to perform a test, specifically using the launch manager subcommand, which manages the benchmarking execution process.

- `--caliper-workspace ./`:
Sets the current directory (./) as the workspace for Caliper. This is where the test’s configuration files and resources are located.

- `--caliper-networkconfig networks/fabric/test-network.yaml`:
Tells Caliper where the network configuration file for Hyperledger Fabric is. The test-network.yaml file specifies details of the Fabric network to be tested, such as node addresses, credentials, organizations, and other essential parameters for connecting to and operating the test.

- `--caliper-benchconfig benchmarks/samples/fabric/fabcar/config.yaml`:
Defines the benchmark configuration file to be executed. In this case, config.yaml specifies the FabCar test specifications. FabCar is a standard smart contract example in Fabric, and the configuration file outlines the transactions to be performed, like reading and writing data on the network.

- `--caliper-flow-only-test`:
This flag instructs Caliper to execute only the test phase of the standard workflow, skipping the preparation or post-execution analysis stages. This allows performance tests to be run directly on the network, assuming it is already configured and operational.

- `--caliper-fabric-gateway-enabled`:

Enables the Fabric Gateway for the test, which simplifies interaction between the application and the Hyperledger Fabric network by using the Fabric Gateway API. The Gateway API facilitates communication with the network by automatically managing connectivity aspects like certificates and signatures.
```bash
npx caliper launch manager --caliper-benchconfig benchmarks/scenario/simple/config.yaml --caliper-networkconfig networks/besu/1node-clique/networkconfig.json --caliper-workspace .
This command sets up and runs a Caliper test on a Hyperledger Besu network in a single-node Clique environment. Here’s the detailed breakdown:
```

- `npx caliper launch manager`:
Runs Caliper (without requiring global installation thanks to npx) with the launch manager subcommand, which is the main manager for starting the benchmarking test.

- `--caliper-benchconfig benchmarks/scenario/simple/config.yaml`:
Specifies the benchmark configuration file. The config.yaml in benchmarks/scenario/simple/ defines the test scenario, including the types of transactions to be performed and the performance metrics to be collected. This "simple" test scenario suggests a series of basic operations (like read/write transactions).

- `--caliper-networkconfig networks/besu/1node-clique/networkconfig.json`:
Sets the network configuration file for Hyperledger Besu, specifically for a single-node Clique network (1node-clique). The networkconfig.json file contains information about the Ethereum network using the Clique protocol (a PoA consensus algorithm), including the node address, accounts, keys, and connection details needed for Caliper to connect to Besu.

- `--caliper-workspace .`:

Defines the current directory (.) as the workspace for Caliper, where all test files and configurations are located. This is where Caliper looks for the benchmark and network configuration files.
This command sets up Caliper to execute a simple test on the single-node Clique network of Hyperledger Besu, measuring the performance of operations defined in the specified benchmark scenario.

## ABI and Bytecode 

1. **ABI (Application Binary Interface)**:
    
    - The ABI is a JSON-formatted description of a smart contract's functions, events, and data types. It acts as a contract's "interface" and defines how you can interact with it.
    - It includes function names, argument types, return types, and whether functions are read-only or alter the blockchain state.
    - When you want to interact with a contract from your code (e.g., using Web3.js or ethers.js), the ABI is required to understand how to encode and decode the function calls and handle responses.

2. **Bytecode**:
    
    - Bytecode is the compiled, low-level machine code that represents the smart contract's logic and is deployed on the blockchain.
    - It’s generated by the Solidity compiler (`solc`) or another language compiler and is uploaded to the blockchain when the contract is deployed.
    - The bytecode includes all instructions for the Ethereum Virtual Machine (EVM) to execute the contract’s functions.

In practical terms:

- **Deploying a contract** requires its bytecode.
- **Interacting with a contract** requires its ABI for encoding function calls and interpreting responses.
