---
layout: post
title:  "Response to 'Why Bitcoin is failing the Muggles'"
categories: []
tags:
- blockchain
- crypto currency
- bitcoin
- ethereum
- consensus mechanisms
- security
- trust
- privacy
- environment
status: publish
type: post
published: true
meta: {}
---
I recently read an [article][original-article] by [Florian Gamper][florian-gamper] about how he thinks blockchain and crypto currencies - Bitcoin in particular - are failing the "muggles" - a nice reference to the average Joe taken from Harry Potter. Even though he made some valid points, I feel the need to make some clarifications.

In his rant, Florian starts with a short introduction and continues by expressing major concerns about 1. the environment, 2. trust, 3. security and 4. privacy. Each of these building blocks gives the reader something to think about and I'm going to stick to the scheme and comment on them sequentially. So here we go..

### Introduction
In this part the author provides some good references for beginners and distance his writing from the current hype which led to failed ICOs and share increases through company renaming. Furthermore, he claims that *"the community"* describes the technology as magic which leads me to believe that he either went to the wrong meetups or talked to the wrong people. 

I do understand that for the bigger part of the society it's incomprehensable how the tech works and yes most of them can't tell the difference between bitcoin and blockchain. However, that doesn't hold true for people familiar with the matter and most certainly nobody of us would call it *"magic"* by any means. I must agree with his statement on the inflationary expectations though, as also [Gartner][gartner-hypecycle] confirms it.

### The Environment
I really do care about the environment and I totally agree with all that has been said in this part. Unfortunately, the author only talked about one very well-known consensus algorithm instead of providing the reader with sufficient information on alternatives. He didn't mention Proof of Stake, a secure and way more [environment-friendly][pos-environment] protocol (most notably implemented in [Ethereum's Casper Protocol][ethereum-pos]) which would completely undermine his argument. Also, he didn't take improvements to the Bitcoin network through second-layer protocols such as the [Lightening Network][bitcoin-lightening] into account which would allow vastly more transactions without the need for additional hash power. If you think that's just a glimpse in the future, you're mistaken as both solutions are either already deployed on corresponding test networks or even had a first live [debut][first-lightening-tx]. 

Another issue with this part is the made-up number of 80% of nodes being in China and the false claims about where the electricity is coming from. I haven't found any statistics on which node uses which source of energy, however, I found the acutal node distribution as of Feb, 12th 2018:

![](/assets/posts/18-02-12_Bitcoin-Node-Distribution.png)

{:.image-caption}
*Source: Bitnodes*

There is also a whole different type of blockchain which is mostly being adopted in enterprises which use different consensus mechanisms such as Proof of Elapsed Time (PoET)  or Redundant Byzantine Fault Tolerance (RBFT) as described in the [Hyperledger Architecture Overview][hyperledger-consensus]. They all consume tremendously less energy than the Proof of Work mechanism mentioned by the author.

### Trust
This was really the part when I decided to clarify things. In my opinion, the line of reasoning in this part is.. let's put gently: questionable. Blockchain enables trustless computing through it's various components with the ultimate goal of reaching consensus. The author argues that the trust isn't actually trust but faith as an algorithm determines it. Even though, technically speaking he's right, he didn't describe what the algorithm actually does in order to reach consensus which would give the reader a chance to judge for himself how much trust vs. faith there is. I'm not going to elaborate in much detail on how the transactions are being validated either, but an integral part of it is to check all transactions that led to the sender being capable of sending a transaction at a certain point in time. In other words: Before a transaction can be made, the funds necessary for this transaction are being validated. A technical description can be found in the protocol rules of the [Bitcoin Wiki][protocol-rules-bitcoin]. From my perspective, this has nothing to do with faith.

The next argument was "money -> nodes -> owning the chain" and a presumptious threat if China's 4 pools would come to an agreement. Well first of all, the referenced article about mining pool distribution is 7 months old which means it's very much outdated. Just 2 months later, exchanges and pools started to shut down or move to [foreign shores][btc-foreign-shores] due to potential [regulation][china-btc-regulation]. Second and more importantly, the author puts too much emphasis on the possibility of a 51% attack which even though theoretically possible and technically feasible, the damage caused is in fact comparably small as described [here][51-percent-attack]. From an economic perspective this attack can be considered infeasable with just a small gain compared to the effort put into it and an expected sudden decline of the token value if the network is compromised.

In the last paragraph the author refers to "other chains" and an even bigger threat for them to be exposed to the aforementioned attack. Since he's not stating any specific chain, I can only guess that he means any of the recent super ICO coins. In fact, many of the coins are based on the [Ethereum network][ethereum-tokens] and are so-called ERC-20 tokens, which would be indeed backed up by the underlying network for the exact reason of not wanting to create a new network, gain traction, build up nodes, etc.

Anyway, on to the next topic..

### Security
In this part the author describes the pitfalls of public key cryptography and how it's always been a challenge to safely store the private key in order to prove ownership. I do agree a lot with what's been said and yes, storing the key in a safe place is a non-trivial task for non-technical people but for this exact reason there are plenty of [wallet types][wallet-types] out there to increase usability without compromising security. The latest trend is the rise of so-called hardware wallets, such as [Trezor][trezor] or [Ledger Nano S][ledger-wallet]. As stated, it's not a problem with the technology itself and at some point people need to adjust to these kinds of things in order to reach mainstream adoption. From my perspective, this is mostly the same reason why PGP didn't reach mainstream adoption even though it should.

### Privacy
Florian again only focusses on Bitcoin in this section but generalizes it by saying blockchain. There are alternatives to prevent the mentioned traceability (also called linkability of transactions), such as [Monero][tx-linkability-xmr]. Also, off-chain transactions haven't been considered even though they provide enhanced privacy as they happen in private.

#### Wrapping it up
In my opinion, the author failed to be concise enough to provide some well researched information on the topic but instead leaves the reader - me - with a lot of questionmarks and unfinished thoughts.

If you have any remarks, please feel free to reach out! :wave:


[original-article]: https://www.linkedin.com/pulse/why-bitcoin-failing-muggles-florian-gamper/
[florian-gamper]: https://www.linkedin.com/in/floriangamper/
[gartner-hypecycle]: https://www.gartner.com/smarterwithgartner/top-trends-in-the-gartner-hype-cycle-for-emerging-technologies-2017/
[pos-environment]: https://coincentral.com/could-proof-of-stake-mend-bitcoins-energy-costs/
[ethereum-pos]: https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ
[bitcoin-lightening]: https://en.wikipedia.org/wiki/Lightning_Network
[first-lightening-tx]: https://www.reddit.com/r/Bitcoin/comments/7rkunw/lightning_the_future_just_arrived_at_my_doorstep/
[hyperledger-consensus]: https://www.hyperledger.org/wp-content/uploads/2017/08/Hyperledger_Arch_WG_Paper_1_Consensus.pdf
[protocol-rules-bitcoin]: https://en.bitcoin.it/wiki/Protocol_rules#.22tx.22_messages
[china-btc-regulation]: https://techcrunch.com/2017/09/14/china-bitcoin-exchange-suspended-bttc-china/
[btc-foreign-shores]: https://www.investopedia.com/news/which-countries-benefit-chinas-crackdown-bitcoin-mining/
[51-percent-attack]: https://learncryptography.com/cryptocurrency/51-attack
[ethereum-tokens]: https://etherscan.io/tokens
[trezor]: https://trezor.io/
[ledger-wallet]: https://www.ledgerwallet.com/
[wallet-types]: http://coinoutletatm.com/7-types-of-bitcoin-wallets/
[tx-linkability-xmr]: https://getmonero.org/2017/04/19/an-unofficial-response-to-an-empirical-analysis-of-linkability.html