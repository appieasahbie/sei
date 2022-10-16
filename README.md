# 𝚂𝚎𝚒𝚗𝚊𝚖𝚒 𝚃𝚎𝚜𝚗𝚎𝚝 𝚃𝚞𝚝𝚘𝚛𝚒𝚊𝚕


![see](https://user-images.githubusercontent.com/108979536/196036172-30688c84-2ecc-4dab-bd2b-4e40aacd40c6.jpeg)



# Hardware Requirements

 + Minimum Hardware Requirements
 
    3x CPUs; the faster clock speed the better
    
    4GB RAM
    
    80GB Disk
    
 
 + Recommended Hardware Requirements
 
    4x CPUs; the faster clock speed the better
    
    8GB RAM
    
    200GB of storage (SSD or NVME)

# Use our script for quick installation

      wget -O sei.sh https://raw.githubusercontent.com/appieasahbie/sei/main/sei.sh && chmod +x sei.sh && ./sei.sh

After installation we gonna install the post

      source $HOME/.bash_profile
      
  + check synchronization status of the node
  
      seid status 2>&1 | jq .SyncInfo
      
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
      
# Create wallet (save everything after enter the command on your notepad )
  
      seid keys add $WALLET
      
  ( if you have already old wallet you can recover it with this command)
  
       seid keys add $WALLET --recover
       
# Add wallet and valoper address and load variables into the system

          SEI_WALLET_ADDRESS=$(seid keys show $WALLET -a)
          SEI_VALOPER_ADDRESS=$(seid keys show $WALLET --bech val -a)
          echo 'export SEI_WALLET_ADDRESS='${SEI_WALLET_ADDRESS} >> $HOME/.bash_profile
          echo 'export SEI_VALOPER_ADDRESS='${SEI_VALOPER_ADDRESS} >> $HOME/.bash_profile
          source $HOME/.bash_profile    
          
# Fund your wallet
  ( go to faucet channel on sei discord and put this command bellow )
  
          !faucet <YOUR_WALLET_ADDRESS>
          
 After you recive the tokens we gonna create a validator
 
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
       
           
  # Basic Firewall security

     ( ufw status it`s inactive open this ports and active firewall)
     
     
           sudo ufw default allow outgoing
           sudo ufw default deny incoming
           sudo ufw allow ssh/tcp
           sudo ufw limit ssh/tcp
           sudo ufw allow ${SEI_PORT}656,${SEI_PORT}660/tcp
           sudo ufw enable
           
           
  # Unjail Validator

           seid tx slashing unjail --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y 
           
  #   Jail Reason

           seid query slashing signing-info $(seid tendermint show-validator)

  #   List All Active Validators

           seid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl

  #   List All Inactive Validators

           seid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl

  #   View Validator Details

           seid q staking validator $(seid keys show wallet --bech val -a)      
           
  #   Withdraw Rewards From All Validators

           seid tx distribution withdraw-all-rewards --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

  #   Withdraw Commission And Rewards From Your Validator

           seid tx distribution withdraw-rewards $(seid keys show wallet --bech val -a) --commission --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

  
  #   Delegate to yourself

           seid tx staking delegate $(seid keys show wallet --bech val -a) 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

  #   Delegate

           seid tx staking delegate <TO_VALOPER_ADDRESS> 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

  #   Redelegate

           seid tx staking redelegate $(seid keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000usei -         
           
  #   Unbond

           seid tx staking unbond $(seid keys show wallet --bech val -a) 1000000usei --from wallet --chain-id atlantic-1 --gas-prices 0.1usei --gas-adjustment 1.5 --gas auto -y

  #   Send

           seid tx bank send wallet <TO_WALLET_ADDRESS> 1000000usei --from wallet --chain-id atlantic-1 --gas-price
        
        
        
  #  Service Management

   + Reload Services

          sudo systemctl daemon-reload

   + Enable Service

          sudo systemctl enable seid

   + Disable Service

          sudo systemctl disable seid

   + Run Service

          sudo systemctl start seid

   + Stop Service

          sudo systemctl stop seid

   + Restart Service

          sudo systemctl restart seid

   + Check Service Status

          sudo systemctl status seid

   + Check Service Logs

          sudo journalctl -u seid -f --no-hostname -o cat      
        
    
