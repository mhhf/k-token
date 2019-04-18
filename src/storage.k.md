# Medallion storage model
### Ceiling 

```k
syntax Int ::= "#Ceiling.wards" "[" Int "]" [function]
// -----------------------------------------------
// doc: whether `$0` is an owner of `Vat`
// act: address `$0` is `. == 1 ? authorised : unauthorised`
rule #Ceiling.wards[A] => #hashedLocation("Solidity", 0, A)

syntax Int ::= "#Ceiling.roof" [function]
// -----------------------------------------------
// doc: maxium value of tokenSupply that can be reached 
// act: Ceiling ensures the token has no more than .` supply 
rule #Ceiling.roof => 1

syntax Int ::= "#Ceiling.tkn" [function] 
// ----------------------------------------------
// doc: address of the Medallion token contract
// act: 
rule #Ceiling.tkn => 2

syntax Int ::= "#MintLike.totalSupply" [function]
// ----------------------------------------------
// doc: total supply of a token contract
// act:
rule #MintLike.totalSupply => 0

```
