## OP_RETURN

```
NOTE: Enriching an inscription with OP_RETURN is completely optional.
```

BSV has several other protocols for tagging on-chain data that can be used in conjunction with these tokens. 1Sat Ordinals can be extended using many existing protocols during inscription. This enables things like minting large videos (> 10MB) and attaching them to a real world geolocation.

In this example, we geotag an ordinal with a location using Magic Attribute Protocol.

```
1SAT_P2PKH INSCRIPTION OP_RETURN
```

Notice we do not use `OP_FALSE OP_RETURN` as is typical in BSV. This is important as ommitting OP_FALSE allows us to spend the output.

```
OP_DUP OP_HASH160 <PUBKEY> OP_EQUALVERIFY OP_CHECKSIG OP_FALSE OP_IF 6f7264 OP_1 <content-type> OP_0 <data> OP_ENDIF OP_RETURN 3150755161374b36324d694b43747373534c4b79316b683536575755374d74555235 534554 617070 6f72642d64656d6f 74797065 706f7374 636f6e74657874 67656f68617368 67656f68617368 6468786e643170776e
```

## Handling Large Files - BCAT

If you were to inscribe files larger than 10MB (at the time of writing this) the ransaction would generally not be accepted by miners. To work around this, a large data file can be spread across multiple transactions. We can achieve this using the BCAT protocol. BCAT lists, in order, the transaction ids required to assemble the data file. It is a Bitcom protocol that uses the prefix "15DHFxWZJT58f9nhyGnsRBqrgwK4W6h4Up". You can find more information about BCAT protocol [here](https://bcat.bico.media/).

While you cannot add the bcat data directly into the inscription fields (since ordinals protocol does not support concatenating external transactions), you can inscribe a thumbnail as the ordinal image, and attach the video using BCAT in OP_RETURN as follows.

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <BCAT fields...> tx2 tx3 tx4...
```

## Adding Metadata

You can use metadata to add context to ordinals. For example, you can tag an ordinal with a geohash to place it on a physical location. Metadata is written using "Magic Attribute Protocol" which has a prefix of "1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5".

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "geohash" geohash "dhmgdqvr7"
```

Similarly, you can attach an ordinal to a particular url:

```bash
1SAT_P2PKH <INSCRIPTION> OP_RETURN <B fields...> | MAP SET app <platform_name> type "post" context "url" url "https://google.com"
```

## Location Tagging

Create with initial location

```
1SAT_P2PKH <INSCRIPTION> OP_RETURN MAP app <appname> type "ordinal" context geohash geohash <geohash> | AIP <sig...>
```

Spend the ordinal with MAP geohash - updates are checked by indexer

```
i1 - 1 sat o1 from inscription tx
o1 - 1SAT_P2PKH OP_RETURN MAP geohash <geohash> | AIP <sig...>
```

Complex example: To inscribe a video that exists across multiple transactions and place it
on a specific geohash with identity signature

```
tx1 - o1 - 1SAT_P2PKH OP_RETURN BCAT tx2 tx3 tx4... | MAP SET context geohash geohash <geohash> | AIP <sig...> -1

tx2 - o1 - BCAT_PART <tx2_data>
```