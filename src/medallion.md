What follows is an executable K specification of the smart contracts for the Centrifuge Medallion Token. Thanks goes to DappHub for their work on multicollateral dai.

# Ceiling
The ceiling contract is a ward on a token contract and limits minting to never succeed a set limit on totalSupply.

## Specification of behaviours

### Accessors

```act
behaviour wards of Ceiling 
interface wards(address usr)

types

    May : uint256

storage

    wards[usr] |-> May

iff

    VCallValue == 0

returns May
```


```act
behaviour roof of Ceiling
interface roof()

types

    Roof : uint256

storage

    roof |-> Supply

iff

    VCallValue == 0

returns Roof
```


### Mutators


#### adding and removing owners

Any owner can add and remove owners.

```act
behaviour rely-diff of Ceiling 
interface rely(address usr)

types

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
behaviour rely-same of Ceiling
interface rely(address usr)

types

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
behaviour deny-diff of Ceiling
interface deny(address usr)

types

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
behaviour deny-same of Ceiling
interface deny(address usr)

types

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

```
behaviour roof of Ceiling 
interface roof()

types

    Roof : uint256

storage

    roof |-> Roof 

iff

    VCallValue == 0

returns May
```


```act
behaviour mint of Ceiling
interface mint(address usr, uint wad)

types
     
    Medallion : address MintLike
    Roof      : uint256
    Wad       : uint256
    User      : uint256
    Supply    : uint256

storage
    tkn          |-> Medallion
    roof         |-> Roof

storage Medallion 
    totalSupply  |-> Supply
    

iff 
    
    totalSupply 

    VCallValue == 0

```

