# Architecture

![bridge architecture](https://user-images.githubusercontent.com/4614574/42094368-f260f648-7b85-11e8-91d4-e602253a6560.png)

### Watcher
A watcher listens for a certain event and creates proper jobs in the queue. These jobs contain the transaction data (without the nonce) and the transaction hash for the related event. The watcher runs on a given frequency, keeping track of the last processed block.

There are three Watchers:
- Signature Request Watcher: Listens to `UserRequestForSignature` events on Home network.
- Collected Signatures Watcher: Listens to `CollectedSignatures` events on Home network.
- Affirmation Request Watcher: Listens to `UserRequestForAffirmation` events on Foreign network.

### Sender
A sender subscribes to the queue and keeps track of the nonce. It takes jobs from the queue, extract transaction data, adds proper nonce and sends it to the network.

There are two Senders:
- Home Sender: Sends transaction to Home network.
- Foreign Sender: Sends transaction to Foreign network.

# How to use

1. Deploy bridge contracts
    1. Clone repo: `git clone https://github.com/poanetwork/poa-bridge-contracts`
    2. `cd poa-bridge-contracts`
    3. Checkout branch `v2` : `git checkout v2`
    4. Compile contracts: `truffle compile`
    5. Go to deploy folder: `cd deploy`
    6. create a `.env` file: `cp .env.example .env` (look at `.env.example` to see the variables that need to be present)
    7. Execute `node deploy.js`

2. Install [RabbitMQ](https://www.rabbitmq.com/) and [Redis](https://redis.io/)
  - RabbitMQ version: `3.7`
  - Redis version: `4.0`
  
3. Create a `.env` file: `cp .env.example .env` and fill in the information using the output data from previous deploy step. Check the `.env.example` file to see the required variables.

## Run the processes

### Native to Erc mode
  - `npm run watcher:signature-request`
  - `npm run watcher:collected-signatures`
  - `npm run watcher:affirmation-request`
  - `npm run sender:home`
  - `npm run sender:foreign`

To send deposits to home contract run `node tests/sendUserTxToHome.js`

To send withdrawals to foreign contract run `node tests/sendUserTxToForeign.js`

Make sure your `HOME_MIN_AMOUNT_PER_TX` and `FOREIGN_MIN_AMOUNT_PER_TX` is same as in your .env deployment contract


### Erc to Erc mode

On `.env` file set `BRIDGE_MODE=ERC_TO_ERC`

  - `npm run watcher:signature-request`
  - `npm run watcher:collected-signatures`
  - `npm run watcher:affirmation-request`
  - `npm run sender:home`
  - `npm run sender:foreign`

To deposit from Foreign to Home contract run `node scripts/sendUserTxToErcForeign.js 10` where `10` is how many tx you would like to send out

To withdrawal to Home to Foreign contract run `node scripts/sendUserTxToErcHome.js 10` where `10` is how many tx you would like to send out

### Run with Docker

  - Start RabbitMQ and Redis: `docker-compose up -d`
  - `docker-compose run bridge npm run watcher:signature-request`
  - `docker-compose run bridge npm run watcher:collected-signatures`
  - `docker-compose run bridge npm run watcher:affirmation-request`
  - `docker-compose run bridge npm run sender:home`
  - `docker-compose run bridge npm run sender:foreign`


To use the bridge UI, clone [the repo](https://github.com/poanetwork/bridge-ui/),
create a `.env` using the same values as before, and run `npm start`.

### Useful commands for development

#### RabbitMQ
Command | Description
--- | ---
`rabbitmqctl list_queues` | List all queues
`rabbitmqctl purge_queue home` | Remove all messages from `home` queue
`rabbitmqctl status` | check if rabbitmq server is currently running  
`rabbitmq-server`    | start rabbitMQ server  

#### Redis
Use `redis-cli`

Command | Description
--- | ---
`KEYS *` | Returns all keys
`SET signature-request:lastProcessedBlock 1234` | Set key to hold the string value.
`GET signature-request:lastProcessedBlock` | Get the value of key.
`DEL signature-request:lastProcessedBlock` | Removes the specified key.
`FLUSHALL` | Delete all the keys of all the existing databases
`redis-cli ping`     | check if redis is running  
`redis-server`       | starts redis server  


### Env Variables
Variable | Description | Values
--- | --- | ---
`ALLOW_HTTP` | Explicitly allows the usage of `http` connections instead of `https`. | `yes` / `no`
`HOME_GAS_PRICE_FALLBACK` | The gas price value (in Wei) which will be used for transactions in Home network if neither the gas price oracle nor gas price set in the HomeBridge contract are available. | integer
`FOREIGN_GAS_PRICE_FALLBACK` | The gs price value (in Wei) which will be used for transactions in Foreign network if neither the gas price oracle nor gas price set in the ForeignBridge contract are available. | integer
`FORCE_NONCE_UPDATE` | Forces to update the nonce by requesting it from the RPC node rather than reading it from Redis DB`. It is possible to specify this only for one sender by executing `env FORCE_NONCE_UPDATE=true npm run sender:home` | `true` / `false`
`LOG_RPC_CALLS` | Outputs every RPC call on the console. It makes sense to specify this variable only for senders or for scripts generating transactions. E.g. `env LOG_RPC_CALLS=true node scripts/sendUserTxToErcHome.js 10`  | `true` / `false` 
