# S2-testnet-2
   ```sh
   SIDE_PORT="CUSTOM PORT"
   ```
## Cloning SIDE Repository and Setup
1. Clone the SIDE repository:
   ```sh
   git clone https://github.com/sideprotocol/side.git
   ```

2. Checkout to the desired version:
   ```sh
   git checkout v0.8.1
   ```

3. Move to the SIDE directory:
   ```sh
   cd side
   ```

4. Install SIDE:
   ```sh
   make install
   ```

# Launch
Here's your tutorial with the commands formatted for clarity:

1. Download the genesis file:
```sh
wget $HOME/.side/config/genesis.json https://github.com/sideprotocol/testnet/raw/main/S2-testnet-2/genesis.json
```

2. Verify the SHA256 hash of the downloaded genesis file:
```sh
shasum -a 256 ~/.side/config/genesis.json
```
Expected output:
```
2c6a6044de04cfdd466c9b114f0082af46c18519dc0ee413d2250de435b5d669  genesis.json
```
# config and init app
   ```sh
   sided config node tcp://localhost:${SIDE_PORT}657
   sided config keyring-backend test
   sided config chain-id S2-testnet-2
   sided init "test" --chain-id S2-testnet-2
   ```

3. Set up seeds:
   ```sh
   SEEDS="582dedd866dd77f25ac0575118cf32df1ee50f98@202.182.119.24:26656"
   sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" $HOME/.side/config/config.toml
   ```
# set custom ports in app.toml
   ```sh
   sed -i.bak -e "s%:1317%:${SIDE_PORT}317%g;
   s%:8080%:${SIDE_PORT}080%g;
   s%:9090%:${SIDE_PORT}090%g;
   s%:9091%:${SIDE_PORT}091%g;
   s%:8545%:${SIDE_PORT}545%g;
   s%:8546%:${SIDE_PORT}546%g;
   s%:6065%:${SIDE_PORT}065%g" $HOME/.side/config/app.toml
   ```
# set custom ports in config.toml file
   ```sh
   sed -i.bak -e "s%:26658%:${SIDE_PORT}658%g;
   s%:26657%:${SIDE_PORT}657%g;
   s%:6060%:${SIDE_PORT}060%g;
   s%:26656%:${SIDE_PORT}656%g;
   s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SIDE_PORT}656\"%;
   s%:26660%:${SIDE_PORT}660%g" $HOME/.side/config/config.toml
   ```
# config pruning
   ```sh
   sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.side/config/app.toml
   sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.side/config/app.toml
   sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.side/config/app.toml
   ```
# set minimum gas price, enable prometheus and disable indexing
   ```sh   
   sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.005uside"|g' $HOME/.side/config/app.toml
   sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.side/config/config.toml
   sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.side/config/config.toml
   ```
# create service file
   ```sh  
   sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
   [Unit]
   Description=Side node
   After=network-online.target
   [Service]
   User=$USER
   WorkingDirectory=$HOME/.side
   ExecStart=$(which sided) start --home $HOME/.side
   Restart=on-failure
   RestartSec=5
   LimitNOFILE=65535
   [Install]
   WantedBy=multi-user.target
   EOF
   ```
# enable and start service
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl enable sided
   sudo systemctl restart sided && sudo journalctl -u sided -f
   ```
# Validating

1. Add a **Bitcoin Segwit Address**
```sh
 sided keys add test --key-type="segwit"

- address: bc1q0xm60dd99hucpkux7rq6vr57g7k479nlw0xapt
  name: test
  pubkey: '{"@type":"/cosmos.crypto.segwit.PubKey","key":"A6gxg+M4sEu0MBFiYlj4r2fEaz/ueeaNE7ymf8Zx+Tqq"}'
  type: local
```

**Note:**
Please ensure that you use a Segwit address; otherwise, you will not be able to claim your rewards.

2. Create Validator
```sh
sided tx staking create-validator \
--from="test" \
--amount="10000000uside" \
--pubkey=$(sided tendermint show-validator)  \
--moniker="side_node" \
--security-contact="contact@side.one" \
--chain-id="S2-testnet-1" \
--commission-rate="0.1" \
--commission-max-rate="0.2" \
--commission-max-change-rate="0.05" \
--min-self-delegation="10000000" \
```

