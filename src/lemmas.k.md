Copyright (C) 2019 Maker Ecosystem Growth Holdings, INC.

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published
by the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.

# Rule for setting addresses (magic nubmer is bitwise not of 2^160).
```k
rule 115792089237316195423570985007226406215939081747436879206741300988257197096960 &Int X => 0
  requires #rangeAddress(X)
```

# Lemmas for bytes4 support
```k
syntax TypedArg ::= #bytes4( Int )

syntax String ::= #typeName ( TypedArg ) [function]
rule #typeName(    #bytes4( _ )) => "bytes4"

syntax Int ::= #lenOfHead ( TypedArg ) [function]
rule #lenOfHead(#bytes4( _ )) => 32

syntax Bool ::= #isStaticType ( TypedArg ) [function]
rule #isStaticType(   #bytes4( _ )) => true

syntax WordStack ::= #enc ( TypedArg ) [function]
rule #enc( #bytes4( DATA )) => #buf(32, #getValue( #bytes4( DATA )))

syntax Int ::= "maxBytes4"
  rule maxBytes4 => 115792089210356248756420345214020892766250353992003419616917011526809519390720 [macro] /* (2^32 - 1) * 2^224 */

syntax Int ::= #getValue ( TypedArg ) [function]
rule #getValue(#bytes4( DATA )) => DATA
  requires minUInt256 <=Int DATA andBool DATA <=Int maxBytes4

rule maxBytes4 &Int X => X
  requires minUInt256 <=Int X andBool X <=Int maxBytes4

rule X &Int maxBytes4 => X
  requires minUInt256 <=Int X andBool X <=Int maxBytes4
```

# Extracting msg.sig of `stop` from calldata
```k
rule maxBytes4 &Int #asWord(99 : 196 : 240 : 49 : nthbyteof(X, 0, 32) : nthbyteof(X, 1, 32) : nthbyteof(X, 2, 32) : nthbyteof(X, 3, 32) : nthbyteof(X, 4, 32) : nthbyteof(X, 5, 32) : nthbyteof(X, 6, 32) : nthbyteof(X, 7, 32) : nthbyteof(X, 8, 32) : nthbyteof(X, 9, 32) : nthbyteof(X, 10, 32) : nthbyteof(X, 11, 32) : nthbyteof(X, 12, 32) : nthbyteof(X, 13, 32) : nthbyteof(X, 14, 32) : nthbyteof(X, 15, 32) : nthbyteof(X, 16, 32) : nthbyteof(X, 17, 32) : nthbyteof(X, 18, 32) : nthbyteof(X, 19, 32) : nthbyteof(X, 20, 32) : nthbyteof(X, 21, 32) : nthbyteof(X, 22, 32) : nthbyteof(X, 23, 32) : nthbyteof(X, 24, 32) : nthbyteof(X, 25, 32) : nthbyteof(X, 26, 32) : nthbyteof(X, 27, 32) : .WordStack) => 45126931774600552203247752449064824644304430000576705787687112960904123121664
  requires X <=Int maxUInt256 andBool X >=Int 0
```
