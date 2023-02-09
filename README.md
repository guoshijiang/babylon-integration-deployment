## Babylon local deployment

This repository hosts a local deployment of the entire Babylon stack for
testing purposes. It involves:
- 2 validator nodes
- 1 vigilante submitter
- 1 vigilante reporter
- 1 bitcoin instance (currently btcd, in the future there is going to be
  support for bitcoind)
- 1 explorer
- 1 [gaiad](https://github.com/cosmos/gaia) instance connected to the validator
  node throuth an IBC [relayer](https://github.com/cosmos/relayer).

### Prerequisites

- Docker
- A github access token. The [vigilante](https://github.com/babylonchain/vigilante) repository
  depends on the [Babylon](https://github.com/babylonchain/babylon) repository which is currently private.
  In order for `go build` to succeed, you need a token that has access to the
  Babylon repository. Instructions on how one can be created can be found
  [here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

### Dependencies

- [Babylon v0.5.0](https://github.com/babylonchain/babylon/tree/v0.5.0).
- [Vigilante commit 4f97912eb5a80f8f7cc80fb3f9a1b9bf62f24495](https://github.com/babylonchain/vigilante/tree/4f97912eb5a80f8f7cc80fb3f9a1b9bf62f24495).
  This commit additionally includes `monitor` deployment to [v0.5.0](https://github.com/babylonchain/vigilante/tree/v0.5.0).
  Will be updated to `v0.6.0` when there is one for the vigilante.
- [Explorer commit a0878479581216107ae21fd6368f806a5be2a16f](https://github.com/babylonchain/babylon-explorer/tree/a0878479581216107ae21fd6368f806a5be2a16f/).
  This will be updated to a stable version when there is one for the explorer.
- [Faucet commit 5b6d9559c4b3d3a0b9fd2c745f3dd9d53ad398d4](https://github.com/babylonchain/faucet/tree/5b6d9559c4b3d3a0b9fd2c745f3dd9d53ad398d4).
  This will be updated to a stable version when there is one for the faucet.

### Deploying

1. Retrieve the underlying repositories for Babylon, vigilante, and explorer:
```shell
git submodule init && git submodule update
```
2. Deploy the system
```shell
make start-deployment-btcd
```

*There is also the possibility to deploy the system along with a
Prometheus/Grafana monitoring stack (Grafana UI under port `3000`)*:
```shell
make start-monitored-deployment-btcd
```
3. Stop the system
```shell
make stop-deployment-btcd
```
4. Deploy the faucet (the faucet frontend under port `3000` and the backend under port `3001`)
```shell
make start-deployment-faucet
```
5. Stop the faucet
```shell
make stop-deployment-faucet
```

### System parameters

The Babylon nodes are deployed with the following parameters:
- 2 validators
- `chain-test` as the chain ID
- `bbt0` as the checkpoint tags
- 10 blocks epoch interval
- Validators are accessible through the `192.168.10.[2-5]` IP addresses.
- The explorer is accessible at port `localhost:26661`. This can be changed by
  modifying `LISTPORT` on the `docker-compose.yml` file.
- Bitcoin uses a block generation time of 30 seconds. This can be changed by
  adding an environment variable `GENERATE_INTERVAL_SECS` on the
  `docker-compose.yml` file.


The explorer is accessible at `localhost:26661`.

### Configuration files

The corresponding node directories, Bitcoin configuration, and
vigilante configuration can be found under `.testnets`
```console
$ ls .testnets
gentxs node0 node1 node2 node3 vigilante bitcoin
```

### Testing

To test a new feature in any of the dependent repositories:
1. Push your branch into the respective repository
2. Change to the directory of the repository maintained by this repository and
   change the branch to the one that contains your new changes.
3. Deploy the system and check that everything works properly.

#### Testing without commiting
For local development without pushing commits,
one can create the Docker image corresponding to the service that they're
testing, with the following names:
- `babylonchain/babylond` for Babylon nodes
- `babylonchain/vigilante-reporter` for the reporter
- `babylonchain/vigilante-submitter` for the submitter
- `babylonchain/explorer` for the explorer
- `babylonchain/nginx-proxy` for the explorer
- `babylonchain/faucet` for the faucet

#### Logs

In order to view the logs of a particular service, one can execute the command
```
docker logs <service_name>
```

This deployment uses the following service names:
- `babylondnode[0-3]` for the Babylon nodes
- `vigilante-reporter` for the reporter
- `vigilante-submitter` for the submitter
- `btcdsim` for the btcd service
- `explorer` for the explorer
- `nginx-proxy` for the nginx proxy
- `faucet-frontend` for the frontend of the faucet
- `faucet-backend` for the backend of the faucet

#### Sanity checks

Here are some checks that one can use to quickly verify if some basic things
are working properly (by viewing the explorer):
- On the first minutes, at least 100 blocks get reported
- Checkpoints become sealed
- Sealed checkpoints become submitted relativelly quickly when new Bitcoin
  blocks get generated
- Checkpoints become confirmed and finalized

If something goes wrong, viewing the logs for services can help. 
