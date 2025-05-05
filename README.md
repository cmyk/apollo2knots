# apollo2knots

## Upgrading FutureBit Apollo II to Bitcoin Knots

This guide walks you through **safely replacing Bitcoin Core with Bitcoin Knots** on the FutureBit Apollo II (Orange Pi-based) miner without reflashing or breaking mining functionality.

---

## ⚠️ Warning

This process **replaces a production node binary**. You **must** stop the node and miner before replacing any files. Failure to do so could lead to corruption or node failure.

---

## ✅ Requirements

- Apollo II with SSH access
- Basic CLI familiarity
- Verified Bitcoin Knots release (see below)

---

## 🔒 Step 1: Download & Verify Bitcoin Knots

SSH into your apollo.

Adjust URLs to what version you want, obvioulsy.
You'll need the `aarch64-linux-gnu` tarball:
```sh
wget https://bitcoinknots.org/files/28.x/28.1.knots20250305/bitcoin-28.1.knots20250305-aarch64-linux-gnu.tar.gz
```

2. Also download these:
```sh
wget https://bitcoinknots.org/files/28.x/28.1.knots20250305/SHA256SUMS.asc
wget https://bitcoinknots.org/files/28.x/28.1.knots20250305/SHA256SUMS
```

2.1 Download developer signatures and import them:
```sh
wget https://raw.githubusercontent.com/bitcoinknots/guix.sigs/knots/builder-keys/chrisguida.gpg
gpg --import chrisguida.gpg

wget https://raw.githubusercontent.com/bitcoinknots/guix.sigs/knots/builder-keys/luke-jr.gpg
gpg --import luke-jr.gpg
```

Refresh the signatures:
```sh
gpg --keyserver hkps://keys.openpgp.org --refresh-keys A291A2C45D0C504A
```

Check fingerprints like this:
```sh
gpg --fingerprint "Luke"
```
Expected (for Luke Dashjr):
```sh
pub   ed25519 2023-02-25 [SC] [expired: 2025-03-25]
      1A3E 761F 19D2 CC77 85C5  502E A291 A2C4 5D0C 504A
uid           [ expired] Luke Dashjr (Codesigning) <luke-jr+git@utopios.org>
```

2.2 This ensures the SHA256SUMS file hasn’t been tampered with:
```sh
gpg --verify SHA256SUMS.asc SHA256SUMS
```
2.3 Verify the download:
This checks the actual file (bitcoin-28.1.knots20250305-aarch64-linux-gnu.tar.gz) against the signed hash — this is the most important step to prevent supply chain attacks.
```sh
sha256sum -c --ignore-missing SHA256SUMS
```
Should output:
```sh
bitcoin-28.1.knots20250305-aarch64-linux-gnu.tar.gz: OK
```

## 🛑 Step 2: Stop Miner and Node

Use the Apollo II web interface to stop both:
- The Bitcoin Node
- The Miner

Wait for full shutdown before continuing.

Make sure, the processes are not running:
```sh
ps aux | grep -i bitcoind
ps aux | grep -i miner
```

## 💾 Step 3: Backup Existing Binary

Backup you existing bitcoind:
```sh
sudo  cp /opt/apolloapi/backend/node/bitcoind ~/bitcoind.backup
```

## 💾 Step 4: Replace Core binary with Knots

Unpack the tar.gz, replace core bitcoind with knots bitcoind and set x bit:

```sh
tar -xf bitcoin-28.1.knots20250305-aarch64-linux-gnu.tar.gz
sudo cp bitcoin-28.1.knots20250305/bin/bitcoind /opt/apolloapi/backend/node/bitcoind
sudo chmod +x /opt/apolloapi/backend/node/bitcoind
```

## 🏁 Step 5: Start Node via GUI

Use the Apollo II web interface to start the node.

You should now see the new version reported in the GUI as:
/Satoshi:28.1.0(FutureBit–Apollo–Node)/Knots:20250305/

![image](https://github.com/user-attachments/assets/55d412d3-0fbf-4fdc-ba92-c7aae33870ad)

## 🛠️ Step 6: Tweak your bitcoind.conf with knots options:

```sh
sudo vim /opt/apolloapi/backend/node/bitcoin.conf
```

```sh
# Block NFT/Ordinal-style junk
rejectnonstdoutputs=1
datacarrier=0

# Optional: block ancient multisig scripts
permitbaremultisig=0
```

Restart node via web gui for changes to take effect.

This config:
- Lets you stay fully on-chain
- Ensures your mempool won’t relay **garbage**
- Prevents your miner from including these TXs
- Keeps you in consensus with rest of network

## 🍺 Have a Beer.

Cheers!
You’ve now got a filtered, principled Knots-based miner.




