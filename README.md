# Send tokens to GoZ Hub 
Tips and tricks to send tokens to and from the Game of Zones Hub with Iqlusion's relayer software.

## Send tokens between zones

```
# add aliases to your preferred shell

# bash/zsh aliases
alias C_AIB='aib-goz-1'
alias C_GOZ='gameofzoneshub-1a'
alias ADDR_AIB='cosmos1dee8te2quczv4gkyckxk9nlr3aefyw77c73pqw'
alias ADDR_GOZ='cosmos1xnc4hwja48e77nms0a49eqkdw36d7dqzrj006n'
alias RPC_GOZ='http://35.233.155.199:26657'

# fish shell aliases
set -Ux C_AIB aib-goz-1
set -Ux C_GOZ gameofzoneshub-1a
set -Ux ADDR_AIB cosmos1dee8te2quczv4gkyckxk9nlr3aefyw77c73pqw
set -Ux ADDR_GOZ cosmos1xnc4hwja48e77nms0a49eqkdw36d7dqzrj006n
set -Ux RPC_GOZ http://35.233.155.199:26657

# remove your old relayer config folder
rm -rf ~/.relayer

# setup a new relayer config folder
rly cfg init

# add at least two zones via their JSON files
rly chains add-dir ./zones/

# initialize light clients for both chains
rly lite init -f $C_AIB
rly lite init -f $C_GOZ

# restore aib account key
rly keys restore $C_AIB testkey "mnemonic"
rly ch edit $C_AIB key testkey
rly q bal $C_AIB

# restore goz account key
rly keys restore $C_GOZ testkey "mnemonic"
rly ch edit $C_GOZ key testkey
rly q bal $C_GOZ

# create a new path between both zones
rly pth gen -f $C_AIB transfer $C_GOZ transfer $AIB_GOZ

# ensure path exists
rly tx link $AIB_GOZ

# try sending tokens, e.g bits from aib to goz

# for bash/zsh
rly tx transfer $C_AIB $C_GOZ 500bits true $(rly ch addr $C_GOZ)

# for fish
rly tx transfer $C_AIB $C_GOZ 500bits true (rly ch addr $C_GOZ)
```

## Debugging relayer issues
```
# General notice on errors
# you need to make sure when you run `rly tx link $AIB_GOZ`
# you get a checkmark next to "channel created" e.g.
Channel created: [aib-goz-1]chan{oigbfuzeen}port{transfer}

# Error: verify non adjacent from # to # failed:
# if you created a path over 90 minutes ago, you will need 
# to re-init the lite clients and create a new path
rly paths delete $AIB_GOZ
rly lite init -f $C_AIB
rly lite init -f $C_GOZ
rly pth gen -f $C_AIB transfer $C_GOZ transfer $AIB_GOZ
rly tx link $AIB_GOZ

# err client: packet commitment verification failed
# err channels: invalid channel state
# if you get these errors you may have unsent tx in the
# relayer queue. View a chain's unrelayed txs with:
rly q unrelayed $AIB_GOZ

# it should show some unrelayed txs:
{"src":["1","2"]}

# you can try to send unsent txs again:
rly tx rly $AIB_GOZ

#----------
# err(sdk: out of gas) 
# if you get this error you may have to increase the gas amount
rly ch edit $C_GOZ gas {{ higher_amount }}

# then relay unsent again with 
rly tx rly $AIB_GOZ

#----------
# err(sdk: insufficient funds)
# if you get this error you need to add more tokens to your account
# or alternatively increase the gas amount
# see how much gas a transaction would cost by appending -d 

# for example to see how much sending all unrelayed txs would cost is 
rly tx rly $AIB_GOZ -d
```

## Useful gaiacli commands
```
# query your balance on the local chain
gaiacli q bank balances $ADDR_AIB

# how to query your balance on another blockchain
gaiacli q bank balances $ADDR_GOZ --node=$RPC_GOZ --chain-id=gameofzoneshub-1a

# send tokens via local RPC
gaiacli tx send $ADDR_AIB $YOUR_AIB_ADDR 1000bits

# how to send tokens via remote RPC
gaiacli tx send $ADDR_GOZ $YOUR_GOZ_ADDR 100000doubloons --node=$RPC_GOZ --chain-id=gameofzoneshub-1a
```
