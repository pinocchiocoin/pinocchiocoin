# PNH — Pinocchio Coin

A CPU-mineable Scrypt cryptocurrency forked from Litecoin 0.18.1.
No ICO. No presale. 1.2% project allocation, disclosed below.
| | |
|---|---|
| Coin name | Pinocchio |
| Ticker | PNH |
| Algorithm | Scrypt |
| Block reward | 50 PNH |
| Block time | 2.5 minutes (150s) |
| Halving | Every 840,000 blocks |
| Max supply | 85,000,000 PNH |
| Difficulty retarget | DarkGravityWave, every block |
| P2P port | 9777 |
| RPC port | 9779 |
| Address prefix | `P` (legacy P2PKH) |
| Based on | Litecoin 0.18.1 |

**Genesis block:** `f1bd5b30b65b5334c29b1551dbeebc8549459e441bba6f69a1f4bc8629dbca73`

Block explorer: https://planetpinocchio.com/explorer.html

---

## ⚠ Read this before you mine

**Your mining address must begin with `P`.**

`cpuminer --coinbase-addr` builds P2PKH outputs only. If you give it a P2SH
address (beginning with `Q`) or a bech32 address (`pnh1...`), the miner will
accept it, blocks will be found, and **every reward will be permanently
unspendable by anyone, including you.**

Set `addresstype=legacy` in your config, generate your address, and confirm
it starts with `P` before you mine anything. Then mine one block and check
your balance before running for any length of time.

---

## Chain

PNH mainnet launched from genesis on 13 August 2026.

It is a fully independent chain: unique address and key prefixes, its own
bech32 HRP, and P2SH and SegWit active from block 0.

Difficulty uses DarkGravityWave, retargeting every block over a 24-block
window with a 3× clamp, so difficulty tracks real hashrate. The chain does
not stall when miners join or leave — a practical requirement for a
CPU-mined coin with variable participation.

Verify you are on the correct chain before mining:

```bash
./src/litecoin-cli -datadir=$HOME/.pinocchio getblockhash 0
```

This must return
`f1bd5b30b65b5334c29b1551dbeebc8549459e441bba6f69a1f4bc8629dbca73`.

---

## Project allocation

Block 1 pays **1,000,000 PNH** (1.2% of max supply) to:

```
PVZnbusbn3c5hyaVifn3whc3gxrSedLJjv
```

Blocks 2 onward pay 50 PNH on the standard schedule.

The allocation funds OTC purchases from miners, airdrops, and exchange
liquidity. It is implemented in `GetBlockSubsidy` (`src/validation.cpp`)
and is verifiable on the block explorer.

It is in block 1 rather than the genesis block because the genesis coinbase
is never added to the UTXO set and would be permanently unspendable — the
same reason Satoshi's genesis 50 BTC has never moved.

---

## Building the node

You need a node to create an address and see your balance.

```bash
sudo apt install -y build-essential libtool autotools-dev automake \
  pkg-config libssl-dev libevent-dev bsdmainutils libboost-all-dev \
  libdb-dev libdb++-dev

git clone https://github.com/pinocchiocoin/pinocchiocoin
cd pinocchiocoin
./autogen.sh
./configure --with-incompatible-bdb
make
```

**Build note for Ubuntu 25.10+ / GCC 15:** if the build fails with
`std::array ... has initializer but incomplete type`, the toolchain has
dropped a transitive include. Add `#include <array>` to the offending file,
or build with an older compiler:

```bash
sudo apt install g++-12 gcc-12
./configure CXX=g++-12 CC=gcc-12 --with-incompatible-bdb
```

---

## Configuration

Create `~/.pinocchio/pinocchio.conf`:

```ini
rpcuser=YOUR_USERNAME
rpcpassword=YOUR_STRONG_PASSWORD
rpcport=9779
daemon=1
server=1
listen=1
port=9777
txindex=1
addresstype=legacy
changetype=legacy
addnode=13.60.252.130:9777
```

`addresstype=legacy` and `changetype=legacy` are required. Without them the
wallet generates P2SH addresses and your mining rewards will be lost.

---

## Running

```bash
./src/litecoind -datadir=$HOME/.pinocchio
```

Give it a moment to sync from the seed node:

```bash
./src/litecoin-cli -datadir=$HOME/.pinocchio getblockcount
./src/litecoin-cli -datadir=$HOME/.pinocchio getconnectioncount
```

---

## Creating your mining address

```bash
./src/litecoin-cli -datadir=$HOME/.pinocchio getnewaddress "mining"
```

Confirm the type:

```bash
./src/litecoin-cli -datadir=$HOME/.pinocchio getaddressinfo YOUR_ADDRESS
```

You need `"isscript": false` and `"ismine": true`, and the address must
start with `P`. If `isscript` is true, `addresstype=legacy` is missing from
your config — fix it, restart the node, and generate another address. Do
not mine to the old one.

---

## Building cpuminer

```bash
sudo apt install -y build-essential libssl-dev \
  libcurl4-openssl-dev libjansson-dev automake

git clone https://github.com/pooler/cpuminer
cd cpuminer && ./autogen.sh && ./configure CFLAGS="-O3" && make
```

---

## Mining

```bash
./minerd -a scrypt \
  -o http://13.60.252.130:9779 \
  -u admin -p pnh_seed_2026 \
  --coinbase-addr=YOUR_P_ADDRESS -t 4
```

Use one fewer thread than you have cores so your node stays responsive.

### Verify before running longer

After your first `accepted: 1/1`, stop the miner and check:

```bash
./src/litecoin-cli -datadir=$HOME/.pinocchio getwalletinfo
```

`immature_balance` must show `50.00000000`.

If it shows `0.00000000`, stop — your address is the wrong type and that
reward is gone. Return to the configuration step.

Rewards mature after 100 confirmations before they can be spent.

---

## Back up your wallet

```bash
cp ~/.pinocchio/wallet.dat ~/wallet_backup.dat
```

This file is the only copy of your keys. Lose it and you lose your coins.

---

## Network

Seed node:

```
addnode=13.60.252.130:9777
```

**WSL2:** mining to the seed node works from WSL2 (outbound only). A node
under WSL2 cannot accept inbound peers, so use native Linux or a VPS if you
want to run a fully connected node.

---

## Selling mined PNH

PNH is not yet listed on an exchange. Applications are pending.

In the meantime the project will buy mined PNH directly for BTC or ETH.
Email info@planetpinocchio.com with the amount and your receiving address.
There is no established market price yet, so the rate is negotiable — early
trading is what will establish one.

---

## Related tokens

PINO and wPNH are ERC-20 tokens on Base. They are separate assets. Neither
is mined, and neither is convertible to or from PNH coin. wPNH is not
backed by, and carries no claim on, PNH coin despite its name.

See https://planetpinocchio.com for details.

---

## Links

- Website: https://planetpinocchio.com
- Block explorer: https://planetpinocchio.com/explorer.html
- Bitcointalk: https://bitcointalk.org/index.php?topic=5585138
- X: https://x.com/PinocchioPNH
- Contact: info@planetpinocchio.com

## License

MIT. See [COPYING](COPYING).

Forked from Litecoin Core, which is forked from Bitcoin Core. Copyright
notices of both are retained.
