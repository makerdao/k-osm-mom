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

# OsmMom Formal Specification

## OsmMom behaviours

### Accessors

#### owner

```act
behaviour owner of OsmMom
interface owner()

for all
    Owner : address

storage
    owner |-> Owner

iff
    VCallValue == 0

returns Owner
```

#### authority

```act
behaviour authority of OsmMom
interface authority()

for all
    Authority : address

storage
    authority |-> Authority

iff
    VCallValue == 0

returns Authority
```

#### osms

```act
behaviour osms of OsmMom
interface osms(bytes32 ilk)

for all
    Osm : address

storage
    osms[ilk] |-> Osm

iff
    VCallValue == 0

returns Osm
```

### Mutators

#### change owner address

```act
behaviour setOwner of OsmMom
interface setOwner(address owner)

for all
    OldOwner : address

storage
    owner |-> OldOwner => owner

iff
    VCallValue == 0
    CALLER_ID == OldOwner
```

#### change authority address

```act
behaviour setAuthority of OsmMom
interface setAuthority(address authority)

for all
    Owner        : address
    OldAuthority : address

storage
    owner     |-> Owner
    authority |-> OldAuthority => authority

iff
    VCallValue == 0
    CALLER_ID == Owner
```

#### set OSM address for particular ilk

```act
behaviour setOsm of OsmMom
interface setOsm(bytes32 ilk, address osm)

for all
    Owner   : address
    PrevOsm : address

storage
    owner     |-> Owner
    osms[ilk] |-> PrevOsm => osm

iff
    VCallValue == 0
    CALLER_ID == Owner
```

### OSM-stopping functionality

#### stop OSM corresponding to specified ilk

```act
behaviour stop-canCall-true of OsmMom
interface stop(bytes32 ilk)

for all
    Owner         : address
    Authority     : address SimpleAuthority
    Osm           : address OSM
    AllowedCaller : address
    May           : uint256

storage
    owner     |-> Owner
    authority |-> Authority
    osms[ilk] |-> Osm

storage Authority
    0 |-> AllowedCaller
    
storage Osm
    wards[ACCT_ID] |-> May
    stopped        |-> _ => 1

iff
    VCallValue == 0
    VCallDepth < 1024
    (CALLER_ID == ACCT_ID) or (CALLER_ID == Owner) or (Authority =/= 0)
    May == 1

if
    CALLER_ID == AllowedCaller

calls
    OSM.stop
    SimpleAuthority.canCall-true
```

```act
behaviour stop-canCall-false of OsmMom
interface stop(bytes32 ilk)

for all
    Owner         : address
    Authority     : address SimpleAuthority
    Osm           : address OSM
    AllowedCaller : address
    May           : uint256
    Stopped       : uint256

storage
    owner     |-> Owner
    authority |-> Authority
    osms[ilk] |-> Osm

storage Authority
    0 |-> AllowedCaller
    
storage Osm
    wards[ACCT_ID] |-> May
    stopped |-> Stopped => #if ((CALLER_ID == ACCT_ID) or (CALLER_ID == Owner)) #then 1 #else Stopped #fi

iff
    VCallValue == 0
    VCallDepth < 1024
    (CALLER_ID == ACCT_ID) or (CALLER_ID == Owner)
    May == 1

if
    CALLER_ID =/= AllowedCaller

calls
    OSM.stop
    SimpleAuthority.canCall-false
```

## OSM behaviours (used as lemmas)

### Mutators

#### stop contract

```act
behaviour stop of OSM
interface stop()

for all
    May : uint256

storage
    wards[CALLER_ID] |-> May
    stopped          |-> _ => 1

iff
    VCallValue == 0
    May == 1
```

## SimpleAuthority behaviours (used as lemmas)

### authorization logic

#### Check whether `src` is authorized to access function with signature `sig` on contract `dst`

```act
behaviour canCall-true of SimpleAuthority
interface canCall(address src, address dst, bytes4 sig)

for all
    AllowedCaller : address

storage
    0 |-> AllowedCaller

iff
    VCallValue == 0

if
  src == AllowedCaller
  sig <= maxBytes4 
  sig >= 0

returns 1
```

```act
behaviour canCall-false of SimpleAuthority
interface canCall(address src, address dst, bytes4 sig)

for all
    AllowedCaller : address

storage
    0 |-> AllowedCaller

iff
    VCallValue == 0

if
  src =/= AllowedCaller
  sig <= maxBytes4 
  sig >= 0

returns 0
```
