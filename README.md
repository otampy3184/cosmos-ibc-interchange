# Cosmos IBC advanced


## Create the Blockchain

```:
% ignite scaffold chain interchange --no-module
```

```:
% cd interchange
```

## Create the module

```:
% ignite scaffold module dex --ibc --ordering unordered --dep bank
```