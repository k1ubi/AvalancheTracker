<div align="center">

```
  ___  _   _  ___   _       ___   _   _ _____  _   _  _____ 
 / _ \| | | |/ _ \ | |     / _ \ | \ | /  __ \| | | ||  ___|
/ /_\ \ | | / /_\ \| |    / /_\ \|  \| | /  \/| |_| || |__  
|  _  | | | |  _  || |    |  _  || . ` | |    |  _  ||  __| 
| | | \ \_/ / | | || |____| | | || |\  | \__/\| | | || |___ 
\_| |_/\___/\_| |_/\_____/\_| |_/\_| \_/\____/\_| |_/\____/ 
```

`[ tracking every AVAX the C-Chain burns, block by block ]`

![nodejs](https://img.shields.io/badge/NODE.JS-ff00c8?style=for-the-badge&logo=nodedotjs&logoColor=00fff9&labelColor=0a0014)
![express](https://img.shields.io/badge/EXPRESS-00fff9?style=for-the-badge&logo=express&logoColor=0a0014&labelColor=0a0014)
![mongodb](https://img.shields.io/badge/MONGODB-ff00c8?style=for-the-badge&logo=mongodb&logoColor=00fff9&labelColor=0a0014)
![ethereum](https://img.shields.io/badge/WEB3-00fff9?style=for-the-badge&logo=ethereum&logoColor=0a0014&labelColor=0a0014)

</div>

<br>

```
▓▒░ 0x00 // SITREP ░▒▓
```

Avalanche's C-Chain burns AVAX on every transaction (EIP-1559-style base fee burn). This tracker
polls the public C-Chain RPC for the latest block, computes the AVAX burned in it, and persists
every block it sees to MongoDB so the running total survives restarts — served through a small
Express + EJS front end.

<br>

```
▓▒░ 0x01 // DATA FLOW ░▒▓
```

```
 https://api.avax.network/ext/bc/C/rpc
                 │
                 │  web3.eth.getBlock("latest")   (every 2s)
                 ▼
     ┌───────────────────────┐
     │  blockController.js   │ ── hash, blockNumber, gasUsed
     └───────────┬───────────┘
                 │  feesCalculator(gasUsed)
                 │  burnedAvax = gasUsed × 470 gwei × 1e-12
                 ▼
     ┌───────────────────────┐        every 5s        ┌─────────────────┐
     │   MongoDB: Blocks     │ ──────────────────────▶│  countFees()    │
     └───────────┬───────────┘                         │  running total  │
                 │                                      └─────────────────┘
                 ▼
        GET /  →  views/index.ejs   (blocks + total burned)
        GET /about → views/about.ejs
```

<br>

```
▓▒░ 0x02 // LOADOUT ░▒▓
```

| dependency | role |
|---|---|
| `express` | HTTP server + routing |
| `ejs` | server-rendered views (`index`, `about`) |
| `mongodb` | persists every polled block in a `Blocks` collection |
| `web3` | talks to the Avalanche C-Chain JSON-RPC endpoint |
| `nodemon` *(dev)* | autoreload during development |

You'll also need a MongoDB connection URI configured in `server/database/db_connector.js` —
cloud-hosted (e.g. Atlas) or a local instance both work.

<br>

```
▓▒░ 0x03 // RUN IT ░▒▓
```

```console
root@node:~/AvalancheTracker# npm install
root@node:~/AvalancheTracker# npx nodemon server.js
Successfully connected to server.
Server is running.
```

Then open `http://localhost:3000` — the block feed starts polling immediately, and the running
burned-AVAX total updates on its own 5-second cycle.

<br>

<div align="center">

`.: . . : <[ every burned block, logged ]> : . : .`

</div>
