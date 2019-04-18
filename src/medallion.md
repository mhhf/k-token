What follows is an executable K specification of the smart contracts for the Centrifuge Medallion Token. Thanks goes to DappHub for their work on multicollateral dai.

# Medallion

The `Medallion` contract is the user facing ERC20 contract maintaining the accounting for Medallion balances. Most functions are standard for a token with changing supply, but it also notably features the ability to issue approvals for transferFroms based on signed messages, called `Permit`s.

## Specification of behaviours

### Accessors

<-- TODO: Name, symbol, domain separator (requires dynamic types) -->
```act
behaviour wards of Medallion
interface wards(address usr)

for all

    May : uint256

storage

    wards[usr] |-> May

iff

    VCallValue == 0

returns May
```

```act
behaviour allowance of Medallion
interface allowance(address holder, address spender)

types

    Allowed : uint256

storage

    allowance[holder][spender] |-> Allowed

iff

    VCallValue == 0

returns Allowed
```

```act
behaviour balanceOf of Medallion
interface balanceOf(address who)

types

    Balance : uint256

storage

    balanceOf[who] |-> Balance

iff

    VCallValue == 0

returns Balance
```

```act
behaviour totalSupply of Medallion
interface totalSupply()

types

    Supply : uint256

storage

    totalSupply |-> Supply

iff

    VCallValue == 0

returns Supply
```

```act
behaviour nonces of Medallion
interface nonces(address who)

types

    Nonce : uint256

storage

    nonces[who] |-> Nonce

iff

    VCallValue == 0

returns Nonce
```

```act
behaviour decimals of Medallion
interface decimals()

storage

    decimals |-> 18

iff

    VCallValue == 0

returns 18
```

```act
behaviour permit_TYPEHASH of Medallion
interface permit_TYPEHASH()

storage

    permit_TYPEHASH |-> keccak(#parseBytesRaw("Permit(address holder,address spender,uint256 nonce,uint256 deadline,bool allowed)")

iff

    VCallValue == 0

returns keccak(#parseBytesRaw("Permit(address holder,address spender,uint256 nonce,uint256 deadline,bool allowed)")
```

### Mutators


#### adding and removing owners

Any owner can add and remove owners.

```act
behaviour rely-diff of Medallion
interface rely(address usr)

for all

    May   : uint256
    Could : uint256

storage

    wards[CALLER_ID] |-> May
    wards[usr]       |-> Could => 1

iff

    // act: caller is `. ? : not` authorised
    May == 1
    VCallValue == 0

if

    CALLER_ID =/= usr
```

```act
behaviour rely-same of Medallion
interface rely(address usr)

for all

    May   : uint256

storage

    wards[CALLER_ID] |-> May => 1

iff

    // act: caller is `. ? : not` authorised
    May == 1
    VCallValue == 0

if
    usr == CALLER_ID
```

```act
behaviour deny-diff of Medallion
interface deny(address usr)

for all

    May   : uint256
    Could : uint256

storage

    wards[CALLER_ID] |-> May
    wards[usr]       |-> Could => 0

iff

    // act: caller is `. ? : not` authorised
    May == 1
    VCallValue == 0

if

    CALLER_ID =/= usr
```

```act
behaviour deny-same of Medallion
interface deny(address usr)

for all

    Could : uint256

storage

    wards[CALLER_ID] |-> Could => 0

iff

    // act: caller is `. ? : not` authorised
    May == 1
    VCallValue == 0

if

    CALLER_ID == usr
```


```act
behaviour transfer-diff of Medallion
interface transfer(address dst, uint wad)

types

    SrcBal : uint256
    DstBal : uint256

storage

    balanceOf[CALLER_ID] |-> SrcBal => SrcBal - wad
    balanceOf[dst]        |-> DstBal => DstBal + wad

iff in range uint256

    SrcBal - wad
    DstBal + wad

iff

    VCallValue == 0

if
    CALLER_ID =/= dst

returns 1
```

```act
behaviour transfer-same of Medallion
interface transfer(address dst, uint wad)

types

    SrcBal : uint256

storage

    balanceOf[CALLER_ID] |-> SrcBal => SrcBal

iff in range uint256

    SrcBal - wad

iff

    VCallValue == 0

if

    CALLER_ID == dst

returns 1
```


```act
behaviour transferFrom-diff of Medallion
interface transferFrom(address src, address dst, uint wad)

types

    SrcBal  : uint256
    DstBal  : uint256
    Allowed : uint256

storage

    allowance[src][CALLER_ID] |-> Allowed => #if (src == CALLER_ID or Allowed == maxUInt256) #then Allowed #else Allowed - wad #fi
    balanceOf[src]            |-> SrcBal  => SrcBal  - wad
    balanceOf[dst]            |-> DstBal  => DstBal  + wad

iff in range uint256

    SrcBal - wad
    DstBal + wad

iff
    (#rangeUint(256, Allowed - wad) or src == CALLER_ID)
    VCallValue == 0

if
    src =/= dst

returns 1
```

```act
behaviour transferFrom-same of Medallion
interface transferFrom(address src, address dst, uint wad)

types

    SrcBal  : uint256
    Allowed : uint256

storage

    allowance[src][CALLER_ID] |-> Allowed => #if (src == CALLER_ID or Allowed == maxUInt256) #then Allowed #else Allowed - wad #fi
    balanceOf[src]            |-> SrcBal  => SrcBal

iff in range uint256

    SrcBal - wad

iff
    (#rangeUint(256, Allowed - wad) or src == CALLER_ID)
    VCallValue == 0

if
    src == dst

returns 1
```

```act
behaviour mint of Medallion
interface mint(address dst, uint wad)

types

    DstBal      : uint256
    TotalSupply : uint256

storage

    wards[CALLER_ID] |-> May
    balanceOf[dst]   |-> DstBal => DstBal + wad
    totalSupply      |-> TotalSupply => TotalSupply + wad

iff in range uint256

    DstBal + wad
    TotalSupply + wad

iff

    May == 1
    VCallValue == 0
```
```act
behaviour burn of Medallion
interface burn(address src, uint wad)

types

    SrcBal      : uint256
    TotalSupply : uint256

storage

    allowance[src][CALLER_ID] |-> Allowed => (#if src == CALLER_ID #then Allowed #else Allowed - wad #fi)
    balanceOf[src]            |-> DstBal => DstBal - wad
    totalSupply               |-> TotalSupply => TotalSupply - wad

iff in range uint256

    SrcBal - wad
    DstBal + wad

iff

    (#rangeUint(256, Allowed - wad) or src == CALLER_ID)
    VCallValue == 0
```


```act
behaviour approve of Medallion
interface approve(address usr, uint wad)

types

    Allowed : uint256

storage

    allowance[CALLER_ID][usr] |-> Allowed => wad

iff
    VCallValue == 0

returns 1
```

```act
behaviour permit of Medallion
interface permit(address holder, address spender, uint256 nonce, uint256 deadline, bool allowed, uint8 v, bytes32 r, bytes32 s)

types

    Nonce : uint256

storage

    nonces[holder]             |-> Nonce => Nonce + 1
    allowance[holder][spender] |-> Allowed => (#if allowed == 1 #then maxUInt256 #else 0 #fi)

iff

    holder == #sender(#unparseByteStack(#padToWidth(32, #asByteStack(keccak(#encodePacked(STUFF))), v,#unparseByteStack(#padToWidth(32, #asByteStack(r))), #unparseByteStack(#padToWidth(32, #asByteStack(s))))))
    deadline == 0 or TIME < deadline
    VCallValue == 0
    nonce == Nonce

iff in range uint256
    Nonce + 1

```


