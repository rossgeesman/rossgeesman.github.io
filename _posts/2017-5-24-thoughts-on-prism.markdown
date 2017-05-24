---
layout: post
title: "My Thoughts on ShapeShift's Prism"
date:   2017-5-24
categories: "ethereum, investing, cryptocurrency"
---

Two days ago, [ShapeShift.io][shapeshift] released [Prism][prism], which they've billed as a "trustless asset portfolio platform" built on Ethereum smart contracts. In a [blog post][prism-blog], ShapeShift founder Erik Voorhees gives an overview of Prism so read that and I'll skip the explanation.

Since I've been following cryptocurrencies for a while and have been working on a project to make it easier to invest in them, I was naturally interested in the announcement. The following are a few thoughts I have about products of this nature and Prism in particular.

ShapeShift hasn't to my knowledge made any of the code public, so I could be totally off base here but it seems like a stretch to claim that Prism is completely trustless. Execution of the Prism contract depends on the prices of the underlying assets that comprise it. Those prices have to be provided from some off-chain entity [(often referred to as an 'oracle')][oracle] and this is a huge point of centralization. I did see several references to an oracle that ShapeShift has developed to handle this but they haven't explained it in any detail. There are projects like [Town Crier][towncrier] that are working on developing trustworthy sources of data for smart contracts, and maybe ShapeShift has collaborated with one of them for this. We'll have to wait until ShapeShift reveals more about the project.

The second major criticism I have is that Prism seems like a poor substitute for holding tokens directly. Lets say I wish to invest $100 in Bitcoin, ZCash, and Ripple and choose to do so through Prism. When setting up the Prism, I will fund the investment by sending $100 worth of Ether to the Prism contract, and this will be matched by a counter party, which for now is ShapeShift. If the underlying assets increase in value by 50%, I could close the contract and would receive an automatic payout of $150 ($50 of the counterparty's collateral + my original $100) minus fees. If the assets were to lose 50% of their value, the contract would give the counterparty $50 of my collateral + $100 and return the remaining $50 to me. So far so good, but what if the price of the assets were to increase by more than 2x? Will the contract automatically close and pay out? If the counterparty holds the underlying assets directly, they could sell them to settle the amount in excess of $200 and pay me out. However, as far as I can tell there isn't a way of proving that they own the assets and I would just have to trust they would pay out. If I was holding Bitcoin, ZCash, and Ripple in my own wallet, I wouldn't have these problems. Also, since a Prism contract is all or nothing, I lose the flexibility to sell part of my stake in order to rebalnce or taker advantage of other opportunities.

By creating another instrument in the form of an Ethereum smart contract, Prism introduces an additional level of abstraction that has some significant downsides. "But, managing cryptographic keys is hard!", you say. I do have sympathy for this, but I don't believe products like Prism are the solution. Isn't cryptocurrency supposed to be "programmable money"? Why not take advantage of this and create softeare to make managing portfolios of multiple cryptocurrencies easier. Multi-currency wallets like Jaxx and Exodus are starting to do this but the experience is lacking for this usecase. When the assets can be moved around programmatically, there shouldn't be any need to introduce more layers of inderiction and pay significant fees to own a basket of digitial assets.

Don't take my criticism as me saying that Prism is totally useless. I'm sure there are many investors who will find use for this product. I just believe that it's much more accurate to compare Prism a financial derivative than an ETF or index fund for blockchain assets. I'm sure ShapeShift will continue to iterate and improve on Prism until some of the criticisms in this post are no longer valid. I definitely look forward to seeing where they go with the project. 

[towncrier]: http://www.town-crier.org
[prism-blog]: https://blog.prism.exchange/blog/introducing-prism-the-worlds-first-trustless-asset-portfolio-platform/
[prism]: https://prism.exchange
[shapeshift]: https://shapeshift.io
[cryptodex]: http://blog.prettymuchallofthetime.com/cryptodex/
[oracle]: https://blockchainhub.net/blockchain-oracles/
