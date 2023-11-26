# Mantra-install-node

Update system and install build tools

    sudo apt update
    sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y

Install Go

    rm -rf $HOME/go
    sudo rm -rf /usr/local/go
    cd $HOME
    curl https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
    cat <<'EOF' >>$HOME/.profile
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export GO111MODULE=on
    export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
    EOF
    source $HOME/.profile
    go version

Install Node

    cd $HOME
    mkdir -p $HOME/go/bin/
    wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip
    unzip mantrachaind-linux-amd64.zip
    rm mantrachaind-linux-amd64.zip
    mv mantrachaind $HOME/go/bin/
    sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
    mantrachaind version

Initialize Node

    mantrachaind init NodeName --chain-id=mantrachain-testnet-1

Download Genesis

    curl -Ls https://ss-t.mantra.nodestake.top/genesis.json > $HOME/.mantrachain/config/genesis.json

Download addrbook

    curl -Ls https://ss-t.mantra.nodestake.top/addrbook.json > $HOME/.mantrachain/config/addrbook.json

Create Service

    sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
    [Unit]
    Description=mantrachaind Daemon
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which mantrachaind) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo systemctl enable mantrachaind

Download Snapshot(optional)

    SNAP_NAME=$(curl -s https://ss-t.mantra.nodestake.top/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
    curl -o - -L https://ss-t.mantra.nodestake.top/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.mantrachain

Launch Node

    sudo systemctl restart mantrachaind
    journalctl -u mantrachaind -f

        
    
    
