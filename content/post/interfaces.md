+++
title = "Do not write misusable APIs"
description = "The art of writing APIs properly"
tags = [
    "software-engineer",
    "development",
]
date = 2020-02-28T20:13:50Z
author = "Lorenzo Peppoloni"
+++
> APIs should be easy to use and hard to misuse.
>
> — Josh Bloch

Today I found this quote on Twitter and for a moment I thought: I'm gonna print it and frame it right now!!

This is something I see a lot in my day to day life as a Software Engineer. Spending days and days looking for nasty bugs resulting in realising that I was misusing an API, taught me that a rule to live by it's:

*Make getting your APIs wrong really...really...really hard*

There should be no doubt in the usage of an API, nothing should be left for the user to guess, everyone should be able to read your API and understands immediately how to use it with no doubts.

Let's write a bad API. 

Imagine we are writing a proto message for an API we are implementing, we want to model a shop transaction:

```proto
message Transaction {
    int64 timestamp = 1;
    string product_code = 2;
    float price = 3;
    string address = 4;
  }
```

This message it's not the most usable, you look at it and question arise:

1) mmmm `int64 timestamp`, wait should I put there an epoch timestamp?
2) `price` should it be in the local currency? 
3) `address` of what? presumably of the shop...maybe?

In this message, for at least 3 fields, it is not immediately and unmistakenly clear how to use them.

Let's try and improve our interface, we can achieve this in multiple ways.

### Good documentation
That's a solution you see quite often and it's a good solution. If your API is well documented, people will know how to use it (presumably).

```proto
// Transaction models a shop transaction. Each transaction involves only one product.
// Each product can be purchased in a shop.
message Transaction {
    // Unix epoch with nanoseconds indicating when the transaction was completed.
    int64 timestamp = 1;
    // Unique product code of the product involved in the transaction.
    string product_code = 2;
    // Price in the local currency of the product involved in the transaction.
    // If the address is not specified, then the currency must be in USD.
    float price = 3;
    // Address of the shop where the transaction happened.
    string address = 4;
  }
```

Ok good, now as a user, I have way more information, I know that a transaction it's a shop transaction, that the timestamp is an epoch timestamp with nanoseconds, that the price is in the local currency of the store and that the address it's the address of the shop.

There is still something bugging, right? The price can be either in local currency if the address is specified, or in USD if no address is provided. Mmmmm despite the comments that document the behaviour that does sound quite right.

Imagine you are a new hire, you don't know anything about this message, you have to return all the transaction happened in the US, you notice that some of the transactions are missing the address, but hey all of them have the price. Let's just fetch all the transactions in USD. That would return the wrong set of transactions...this could lead to bugs that are quite hard to find.

The problem here is one of cognitive load, still, the API doesn't document itself fully, you still need to have some previous knowledge to use it (in this case that in the case of missing address the price is in USD).

As an additional point, comments will not be available in the classes generated for this message. A developer will always have to dig up the proto definition and read the comments.

### Good naming

If the names of the fields speak for themselves and make themselves unmistakable, well...then it's really hard to get it wrong.

```proto
// ShopTransaction defines a transaction happened in a shop.
message ShopTransaction {
    int64 epoch_timestamp_ns = 1;
    string purchased_product_code = 2;
    // Price of the transaction, always in USD despite the location
    // where the transaction happened.
    float price_usd = 3;
    string shop_address = 4;
  }
```

As you can clearly see at this point comments are basically superfluous. Each field tells exhaustively what it contains and what should be put in it. The currency it's always in USD, so no assumption can be made on the purchase location using the price, and it's clear from the name, the address it's the address of the shop (clear from the name), the timestamp is epoch nanoseconds, clear from the name.

Good. We don't need much commenting at this point, but we are still free to add them. We can still drop a line saying that the price is always in USD, no matter the address, but it's not strictly necessary.

As a bonus, to feel fully happy about the message, I would probably change the `int64` to a [google Timestamp](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto) and maybe deal a bit better with the address field. Defining an `address` message might be a good idea but it really depends on the use case.
