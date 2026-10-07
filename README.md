# Pinocchio (PNH)

A CPU mineable Scrypt cryptocurrency with its own blockchain, forked from
Litecoin 0.18.1. No ICO, no presale, 1.2% project allocation disclosed below.

PNH is the coin behind the Planet Pinocchio Football Universe, a fictional
football world running since 1983. Every result in the Universe League pays
PNH into the winning club's wallet, on-chain and publicly verifiable.

| | |
|---|---|
| Coin name | Pinocchio |
| Ticker | PNH |
| Algorithm | Scrypt |
| Block reward | 50 PNH |
| Block time | 2.5 minutes |
| Halving | Every 840,000 blocks |
| Max supply | 85,000,000 PNH |
| Difficulty retarget | DarkGravityWave, every block |
| Address prefix | `P` (legacy P2PKH) |
| P2P port | 9777 |
| RPC port | 9779 |
| Based on | Litecoin 0.18.1 |

**Genesis:** `f1bd5b30b65b5334c29b1551dbeebc8549459e441bba6f69a1f4bc8629dbca73`

- Block explorer: https://planetpinocchio.com/explorer.html
- Live results and club prize fund: https://planetpinocchio.com/league.html
- Website: https://planetpinocchio.com

---

## Get an address without building anything

The quickest way to hold PNH is the browser address generator:

**https://planetpinocchio.com/address.html**

It derives a keypair entirely client side. Nothing is sent to any server.
You get a `P` address for receiving and a WIF private key to keep. That is
enough to receive club prize money or mining rewards. To spend what you
receive, import the key into the wallet below.

---

## Club prize money

Twelve clubs each hold a PNH address. Every competitive result pays into
the winning club's wallet.

| Result | Pays |
|---|---|
| League win | 600 PNH |
| League draw | 200 PNH |
| Per goal scored | 100 PNH |
| Universe Cup round | x2 |
| Universe Cup Final | x4 |
| Tea Cup group or QF | x3 |
| Tea Cup semi-final | x5 |
| Tea Cup Final | x10 |

Payments are made weekly in a single transaction. Running totals and the
twelve club addresses are published at
https://planetpinocchio.com/league.html and every payment can be checked
on the block explorer.

---

## Mining

Difficulty is low enough that a normal desktop CPU finds blocks in under a
minute.

### 1. Get an address

Either use the [browser generator](https://planetpinocchio.com/address.html),
or build the wallet below and run `getnewaddress`.

Your address must begin with `P`. If you build the wallet, put
`addresstype=legacy` in your config. Without it the node produces P2SH
addresses beginning with `Q`, and cpuminer cannot build P2SH outputs, so
rewards mined to a `Q` address are permanently unspendable.

### 2. Build cpuminer

```
sudo apt install -y build-essential libssl-dev \
  libcurl4-openssl-dev libjansson-dev automake

git clone https://github.com/pooler/cpuminer
cd cpuminer && ./autogen.sh && ./configure CFLAGS="-O3" && make
```

### 3. Mine

```
./minerd -a scrypt \
  -o http://13.60.252.130:9779 \
  -u admin -p pnh_seed_2026 \
  --coinbase-addr=YOUR_P_ADDRESS -t 4
```

No node needed. You mine directly against the seed node.

### 4. Check it worked

If you built the wallet:

```
./src/litecoin-cli -datadir=$HOME/.pinocchio getwalletinfo
```

`immature_balance` should show `50.00000000` after your first accepted
block. Rewards mature after 100 confirmations.

If you used the browser generator, look the address up on the
[block explorer](https://planetpinocchio.com/explorer.html).

---

## Building the wallet

Needed if you want to spend coins, run a node, or generate addresses
locally.

```
sudo apt install -y build-essential libtool autotools-dev automake \
  pkg-config libssl-dev libevent-dev bsdmainutils libboost-all-dev \
  libdb-dev libdb++-dev

git clone https://github.com/pinocchiocoin/pinocchiocoin
cd pinocchiocoin
./autogen.sh
./configure --with-incompatible-bdb
make
```

Create `~/.pinocchio/pinocchio.conf`:

```
rpcuser=YOUR_USERNAME
rpcpassword=YOUR_STRONG_PASSWORD
rpcport=9779
rpcbind=127.0.0.1
rpcallowip=127.0.0.1
daemon=1
server=1
listen=1
port=9777
txindex=1
addresstype=legacy
addnode=13.60.252.130:9777
```

Start it:

```
./src/litecoind -datadir=$HOME/.pinocchio -daemon
./src/litecoin-cli -datadir=$HOME/.pinocchio getblockcount
./src/litecoin-cli -datadir=$HOME/.pinocchio getnewaddress "mining"
```

To import a key from the browser generator:

```
./src/litecoin-cli -datadir=$HOME/.pinocchio importprivkey YOUR_WIF_KEY
```

Back up `~/.pinocchio/wallet.dat`. It is the only copy of your keys.

---

## Network

Seed node:

```
addnode=13.60.252.130:9777
```

**WSL2:** mining to the seed node works from WSL2, which only needs
outbound connections. A full node under WSL2 cannot accept inbound peers.
Use native Linux or a VPS to run a node.

---

## Project allocation

Block 1 pays **1,000,000 PNH** (1.2% of max supply) to
`PVZnbusbn3c5hyaVifn3whc3gxrSedLJjv`. Blocks 2 onward pay 50 PNH on the
standard schedule.

The allocation funds club prize money, OTC purchases from miners, and
airdrops. It is visible on the block explorer and in `GetBlockSubsidy`
in the source.

---

## Relaunch, August 2026

The chain was rebuilt from a new genesis after a review found several
parameters inherited unchanged from Litecoin that would have caused
problems later:

- `PUBKEY_ADDRESS` was 48, identical to Litecoin, so addresses were
  indistinguishable between the two chains and cross-chain sends would
  have been accepted and lost. Now 55.
- `SCRIPT_ADDRESS2` and `SECRET_KEY` likewise changed, so PNH private
  keys no longer import into Litecoin Core.
- `bech32_hrp` was `ltc`, now `pnh`.
- `vFixedSeeds` still carried Litecoin's peer list, so nodes could never
  discover each other.
- Difficulty used Litecoin's 2016-block retarget, which would have
  frozen the chain if a miner joined and left. Replaced with
  DarkGravityWave, retargeting every block.
- BIP16 (P2SH) was unenforced until height 218,579.
- CSV and SegWit had BIP9 windows that expired in 2018 and could never
  have activated. Both are now active from height 0.
- `nMinimumChainWork` and `defaultAssumeValid` were non-zero, leaving
  `IsInitialBlockDownload()` permanently true so `getblocktemplate`
  refused every mining request.
- `MAX_MONEY` was 100,000,000 against an 84,000,000 issuance schedule.

The original chain was stopped and archived. No PNH was held by anyone
outside the project at the time.

---

## Selling mined PNH

PNH is not listed on any exchange. Applications are pending and there is
no market price.

In the meantime the project will buy mined PNH for BTC or ETH. Email
**info@planetpinocchio.com** with the amount and your receiving address.
The rate is negotiable because there is nothing to price against yet.

---

## Other tokens

Separate ERC-20 tokens on Base, not convertible to or from PNH coin:

| Token | Contract |
|---|---|
| PINO | 0x579da34BE72f48328eB5efD1B311e5D3cD1B7129 |
| Winners Coin | 0x5C201703E40491ad5c7163266fe3A9D6631ab416 |
| PP Planet Pinocchio | 0x79334c3E64D85173Bd875dc72843F2DF7387cd16 |
| BP Bad Planet | 0x44B8AaD8F9424e78eaD423F4524Af52569281f78 |
| FP Fort Province | 0x47DeB209ee8A1F62EA680d4C3Bf9B7189bf08840 |

---

## Links

- https://planetpinocchio.com
- https://planetpinocchio.com/explorer.html
- https://planetpinocchio.com/league.html
- https://planetpinocchio.com/address.html
- https://bitcointalk.org/index.php?topic=5585138
- https://x.com/PinocchioPNH
- info@planetpinocchio.com

## License

MIT
