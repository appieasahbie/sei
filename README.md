# Seinami Testnet Tutorial

![see](https://user-images.githubusercontent.com/108979536/196041590-623aece2-4d67-46e9-a113-7c7286eadc13.jpeg)


# How do I get involved?

  + Register here to join the testnet.  [Link](https://docs.google.com/forms/d/e/1FAIpQLSfD-FWT3VrxtYAAmUiwwX5Zbw3mzkZoT6pV0ZAXYqu1yUNtEw/viewform)

  + Review Sei testnet mission docs.    [Link](https://3pgv.notion.site/Sei-Network-Incentivized-Testnet-Seinami-1f3de71c76c24d4f862af936f0a5fe04)
  
  + Join Discord server to stay up to date on announcements.   [Link](https://discord.gg/T34hks4Y)

  + Join Telegram channel to talk to the team and fellow users [Link](https://t.me/seinetwork)
  
  
 # Hardware Requirements

  + Minimum Hardware Requirements
  
       3x CPUs; the faster clock speed the better
       
       4GB RAM
       
       80GB Disk
       
  + Recommended Hardware Requirements
  
       4x CPUs; the faster clock speed the better
       
       8GB RAM
       
       200GB of storage (SSD or NVME)
       
 ### For quick installation use our script (enter your wallet name)      
 
 
     wget -O sei.sh https://raw.githubusercontent.com/appieasahbie/sei/main/sei.sh && chmod +x sei.sh && ./sei.sh

 ### Post installation

      source $HOME/.bash_profile
      
(Check the status of your node)

      seid status 2>&1 | jq .SyncInfo

# open ports and active the firewall

      sudo ufw default allow outgoing
      sudo ufw default deny incoming
      sudo ufw allow ssh/tcp
      sudo ufw limit ssh/tcp
      sudo ufw allow ${SEI_PORT}656,${SEI_PORT}660/tcp
      sudo ufw enable

# Data snapshot

      sudo apt update
      sudo apt install lz4 -y
      sudo systemctl stop seid
      seid tendermint unsafe-reset-all --home $HOME/.sei --keep-addr-book

      cd $HOME/.sei
      rm -rf data

     SNAP_NAME=$(curl -s https://snapshots1-testnet.nodejumper.io/sei-testnet/ | egrep -o ">atlantic-1.*\.tar.lz4" | tr -d ">")
     curl https://snapshots1-testnet.nodejumper.io/sei-testnet/${SNAP_NAME} | lz4 -dc - | tar -xf -

     sudo systemctl restart seid
     sudo journalctl -u seid -f --no-hostname -o cat
     
     
# Create wallet
 + (Please save all keys on your notepad)
 
     seid keys add $WALLET
     
# To recover your old wallet use this command

     seid keys add $WALLET --recover


# Add wallet and valoper address and load variables into the system

     SEI_WALLET_ADDRESS=$(seid keys show $WALLET -a)
     SEI_VALOPER_ADDRESS=$(seid keys show $WALLET --bech val -a)
     echo 'export SEI_WALLET_ADDRESS='${SEI_WALLET_ADDRESS} >> $HOME/.bash_profile
     echo 'export SEI_VALOPER_ADDRESS='${SEI_VALOPER_ADDRESS} >> $HOME/.bash_profile
     source $HOME/.bash_profile
     
# Fund your wallet
  (go to sei discord channel and use faucet chain to fund your wallet)
    
     !faucet <YOUR_WALLET_ADDRESS>
     
     
# Create validator
   (first check your bank balance )
    
      seid query bank balances $SEI_WALLET_ADDRESS
      
   if you recive the tokens let`s create validator
   
      seid tx staking create-validator \
      --amount 1000000usei \
      --from $WALLET \
      --commission-max-change-rate "0.01" \
      --commission-max-rate "0.2" \
      --commission-rate "0.07" \
      --min-self-delegation "1" \
      --pubkey  $(seid tendermint show-validator) \
      --moniker $NODENAME \
      --chain-id $SEI_CHAIN_ID
      
      
 # Delegate to yourself

       seid tx staking delegate $(seid keys show wallet --bech val -a) 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

 # Delegate

       seid tx staking delegate <TO_VALOPER_ADDRESS> 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

 # Redelegate

       seid tx staking redelegate $(seid keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

 # Unbond

       seid tx staking unbond $(seid keys show wallet --bech val -a) 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

 # Send

       seid tx bank send wallet <TO_WALLET_ADDRESS> 1000000usei --from wallet --chain-id atlantic-1   
       
 #  Validator management
    +Edit validator

       seid tx staking edit-validator \
       --moniker=$NODENAME \
       --identity=<your_keybase_id> \
       --website="<your_website>" \
       --details="<your_validator_description>" \
       --chain-id=$SEI_CHAIN_ID \
       --from=$WALLET


#  Get Validator Info

      seid status 2>&1 | jq .ValidatorInfo

#  Get Catching Up

      seid status 2>&1 | jq .SyncInfo.catching_up

#  Get Latest Height

      seid status 2>&1 | jq .SyncInfo.latest_block_height

#  Get Peer

      echo $(seid tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.sei/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')

#  Reset Node

      seid tendermint unsafe-reset-all --home $HOME/.sei --keep-addr-book

#  Remove Node

      sudo systemctl stop seid && sudo systemctl disable seid && sudo rm /etc/systemd/system/seid.servi     
      
#  Service Management

### Reload Services

     sudo systemctl daemon-reload

### Enable Service

     sudo systemctl enable seid

### Disable Service

     sudo systemctl disable seid

### Run Service

     sudo systemctl start seid

### Stop Service

     sudo systemctl stop seid

### Restart Service

     sudo systemctl restart seid

### Check Service Status

     sudo systemctl status seid

### Check Service Logs

     sudo journalctl -u seid -f --no-hostname -o cat      
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
     
   
