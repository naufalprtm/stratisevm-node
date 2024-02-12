# StratisEVM Validator Node Guide for Auroria Testnet

## Introduction

This guide provides step-by-step instructions to set up a validator node for the StratisEVM Auroria Testnet. Make sure to follow each step carefully.

## Prerequisites

## Ubuntu-based server
## System Requirements

- **OS:** 64-bit Linux, Mac OS X 10.14+, Windows 10+ 64-bit
- **CPU:** 4+ cores @ 2.8+ GHz
- **Memory:** 8GB+ RAM
- **Storage:** SSD with at least 1TB free space
- **Network:** 8 MBit/sec broadband

## Port Requirements

- **GETH:**
  - Allow: 30303/UDP+TCP in+out
  - Block: 8545/TCP all

- **Prysm beacon-chain:**
  - Allow: */TCP+UDP out, 13000/TCP in+out, 12000/UDP in+out
  - Block: 3500/TCP all, 8551/TCP all, 4000/TCP all

# Setting Firewall

```

sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
sudo ufw deny 8545/tcp
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow out 13000/tcp
sudo ufw allow out 13000/udp
sudo ufw allow out 12000/tcp
sudo ufw allow out 12000/udp
sudo ufw allow 13000/tcp
sudo ufw allow 13000/udp
sudo ufw allow 12000/tcp
sudo ufw allow 12000/udp
sudo ufw deny 3500/tcp
sudo ufw deny 8551/tcp
sudo ufw deny 4000/tcp
sudo ufw enable

```
# Setup
Create a directory for StratisEVM and navigate into it:

```
mkdir StratisEVM
cd StratisEVM

```

# This command recursively changes the ownership of all files and directories under /root/StratisEVM/ to the specified user and group.
Please replace "yourusername" with the actual username you want to grant ownership and permissions to. Keep in mind that modifying the permissions and ownership of system directories may have security implications, so proceed with caution.
```

sudo chown -R yourusername:yourusername /root/StratisEVM/

```
- This command recursively changes the ownership of all files and directories under /root/StratisEVM/ to the specified user and group.
- After changing the ownership, make sure the user has the necessary permissions to access and modify the files. You can also grant read, write, and execute permissions using the chmod command if needed:
```

sudo chmod -R 755 /root/StratisEVM/

```

#  Download required binaries:

```
wget https://github.com/stratisproject/go-stratis/releases/download/0.1.0/geth-linux-amd64-fabe6d6.tar.gz
wget https://github.com/stratisproject/prysm-stratis/releases/download/0.1.0/beacon-chain-linux-amd64-1ffa421.tar.gz
wget https://github.com/stratisproject/prysm-stratis/releases/download/0.1.0/validator-linux-amd64-1ffa421.tar.gz
wget https://github.com/stratisproject/staking-deposit-cli/releases/download/0.1.0/staking-deposit-cli-linux-amd64.zip

```

#  Extract and move binaries to /usr/local/bin:

```

apt install unzip
tar -zxvf geth-linux-amd64-fabe6d6.tar.gz
rm geth-linux-amd64-fabe6d6.tar.gz
tar -zxvf beacon-chain-linux-amd64-1ffa421.tar.gz
rm beacon-chain-linux-amd64-1ffa421.tar.gz 
tar -zxvf validator-linux-amd64-1ffa421.tar.gz
rm validator-linux-amd64-1ffa421.tar.gz
unzip staking-deposit-cli-linux-amd64.zip
rm staking-deposit-cli-linux-amd64.zip
mv /root/StratisEVM/staking-deposit-cli-linux-amd64/staking-deposit-cli-linux-amd64.tar.gz /root/StratisEVM/
rm -rf staking-deposit-cli-linux-amd64
tar -zxvf staking-deposit-cli-linux-amd64.tar.gz
rm staking-deposit-cli-linux-amd64.tar.gz

```

#  Create Geth systemd service:
- If you experience an error, pay attention to the variables
- User=root
- adjust it to your VPS in a way
- example output:root
```

whoami

```

##  Create Geth systemd service:

```

sudo tee /etc/systemd/system/geth-auroria.service > /dev/null <<EOF
[Unit]
Description=Geth Node for Auroria Testnet
After=network.target

[Service]
ExecStart=/root/StratisEVM/geth --auroria --http --http.api eth,net,engine,admin --datadir=data/testnet/geth --authrpc.addr=127.0.0.1 --authrpc.jwtsecret=jwtsecret --syncmode=full
Restart=always
User=root

[Install]
WantedBy=default.target
EOF

```


#  Start and enable Geth service:

```
sudo systemctl daemon-reload
sudo systemctl enable geth-auroria.service
sudo systemctl start geth-auroria.service


```

#  Monitor Geth logs:

```

journalctl -u geth-auroria -f

```


#  Create Beacon systemd service:

```

sudo tee /etc/systemd/system/beacon-auroria.service > /dev/null <<EOF
[Unit]
Description=Beacon Node for Auroria Testnet
After=network.target

[Service]
ExecStart=/bin/bash -c 'echo "accept" | /root/StratisEVM/beacon-chain --auroria --datadir=data/testnet/beacon --execution-endpoint=http://localhost:8551 --jwt-secret=jwtsecret'
Restart=always
User=root

[Install]
WantedBy=default.target
EOF

```

#  Reload systemd configuration:

```

sudo systemctl daemon-reload

```


#  Start Beacon service:

```
sudo systemctl enable beacon-auroria.service
sudo systemctl start beacon-auroria.service 

```


#  Monitor Beacon logs:

```

journalctl -u beacon-auroria -f

```


#  Download Staking Deposit CLI:
Visit Stratis Staking Launchpad to generate mnemonic and keys.https://auroria.launchpad.stratisevm.com/en/generate-keys

```

./deposit new-mnemonic --num_validators 1 --chain auroria --eth1_withdrawal_address <WALLET_ADDRESS>

```

#  Go to the faucet and perform the staking deposit.
https://auroria.faucet.stratisevm.com/

## Run the validator:

```

./validator accounts import --keys-dir=/root/StratisEVM/validator_keys  --auroria

```



#  Install Screen (on Ubuntu/Debian):

```
sudo apt-get update
sudo apt-get install screen
```

#  Start a New Screen Session:

```
screen -S validator

```
#  Run a Command Inside the Screen Session:
#### CHANGE
- --suggested-fee-recipient=YOURWALLETADDRESS 
```
./validator --wallet-dir=/root/.eth2validators/prysm-wallet-v2 --auroria --suggested-fee-recipient=YOURWALLETADDRESSHERE
```
## Detach from the Screen Session:
## Press Ctrl + A, then D.

#  List All Screen Sessions:

```
screen -ls

```
#  Reattach to a Screen Session:(FOR CHECK IF YOU WANT TO QUIT FROM SCREEN SESSION JUST CTRL+A+D)

```
screen -r validator
```
#  USEFULL COMMAND
-  List active screen sessions:
```
screen -ls
```
-  Terminate a screen session (when attached):
-  Press Ctrl + A, then :quit or :exit.

Terminate a screen session (from outside):
```
screen -X -S session_id quit
```
-  Replace session_id with the actual session ID.

-  Check the running validator process:
```
ps aux | grep 'validator'
```
-  Stop the validator process:
-  Find the screen session ID using screen -ls, then use screen -X -S session_id quit.

-  View logs (replace validator_logs.txt with your actual log file):
```
tail -f validator_logs.txt 
```



    
#  Create validator systemd service (OPTION 2)


#  Create a password file and set the desired password:
 echo "yourpassword"  example:12345
 
```
    
    echo "yourpassword" > /root/StratisEVM/password.txt
    
```
    
#  Update system packages and install the 'expect' tool:

```
    
    sudo apt-get update
    sudo apt-get install expect
    
```
#  Secure the password file by adjusting permissions:

```
    
    sudo chmod 400 /root/StratisEVM/password.txt
    sudo chown root:root /root/StratisEVM/password.txt
  
```    


Now, you can proceed to create the systemd service for the StratisEVM validator:
change this
 - suggested-fee-recipient=your_wallet_here example:0x1010101010

 
```
sudo tee /etc/systemd/system/validator-auroria.service > /dev/null <<EOF
[Unit]
Description=Validator Node for Auroria Testnet
After=network.target

[Service]
ExecStart=/usr/bin/expect -c "spawn /root/StratisEVM/validator --wallet-dir=/root/.eth2validators/prysm-wallet-v2 --auroria --suggested-fee-recipient=YOUADDRESSHERE; sleep 2; set password [exec cat /root/StratisEVM/password.txt]; send \"\$password\r\"; expect \"Wallet password:\"; send \"\$password\r\"; interact"
Restart=always
User=root

[Install]
WantedBy=default.target
EOF



```
#  If got error try with this before change 
#### Change to your address at=suggested-fee-recipient=0x000000;
```
/usr/bin/expect -c 'spawn /root/StratisEVM/validator --wallet-dir=/root/.eth2validators/prysm-wallet-v2 --auroria --suggested-fee-recipient=YOURADDRESSHERE; sleep 2; set password [exec cat /root/StratisEVM/password.txt]; send "$password\r"; expect "Wallet password:"; send "$password\r"; interact'
```
# then you can try it, If you still get error, change to option 2
#### CHANGE THIS YOURPASSWORD and 0x1010101010
- password at here send \"YOURPASSWORD\r\"; interact"
- suggested-fee-recipient=your_wallet_here example:0x1010101010
```
ExecStart=/usr/bin/expect -c "spawn /root/StratisEVM/validator --wallet-dir=/root/.eth2validators/prysm-wallet-v2 --auroria --suggested-fee-recipient=YOURWALLETHERE; sleep 2; set password [exec cat /root/StratisEVM/password.txt]; send \"\$password\r\"; expect \"Wallet password:\"; send \"\$password\r\"; interact"password:\"; send \"YOURPASSWORD\r\"; interact"
```

After that, try again, you just need to change the exec section and copy and paste it again in the terminal
Once finished, continue and repeat the step "sudo systemctl daemon-reload"
#  Reload systemd configuration and start validator service:

```

sudo systemctl daemon-reload
sudo systemctl enable validator-auroria.service
sudo systemctl start validator-auroria.service

```

#  Check validator logs:

```

journalctl -u validator-auroria -f


```


#  Congratulations! Your StratisEVM validator node for Auroria Testnet is now set up and running.

























