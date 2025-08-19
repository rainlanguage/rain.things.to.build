# Rain options

## Context

We want a way for Rain users to set future prices for tokens.

Alice would ask Bob to lock in the right to buy/sell some TKN at a predetermined
price at some point in the future.

Bob bears the risk that Alice may walk away, most likely in the case that the
market is offering a better price than the predetermined price when the trade is
scheduled.

For this reason, Bob would expect a premium in the predetermined price relative
to the market at the moment the agreement is made.

For example, say ETH is trading at $4k today.

Alice might want to buy some ETH in 1 month for $4k if the market has moved above
$4k, presumably to take her locked in price and go flip it on the markets.

Bob wouldn't accept these terms as-is, but he might be willing to if Alice was to
pay him, say $100 per ETH up front to lock in the price.

This would make Alice's effective final price $4.1k with a minimum up-front
commitment.

The final result is that if ETH is trading above $4.1k in 1 month, then Alice
will profit. If it trades less than this, she walks away from the trade, and the
cost to her is $100 per ETH.

Bob either makes $100 for committing his ETH to Alice for 1 month, or he incurs
an _opportunity cost_ by selling ETH to Alice in 1 month for a worse price than
he would have got on that future market (today's price).

### Onchain

Because all of this is happening onchain, the only way that we can enforce any of
the above is via. some kind of escrow process handled by a smart contract.

We have to actually take Bob's ETH, and Alice's premium up front. Bob can claim
his premium whenever, but his ETH is released conditionally to either himself or
Alice only after the final conditions are met.

Either Alice finalises the payment to Bob at the correct time, and so receives
the ETH at the predetermined price, or she does nothing and Bob can retrieve his
ETH once the contract times out.

### Practical example: Cyclo insurance

Cyclo is a leverage protocol that issues cy* tokens when users deposit
collateral. They sell their cy* tokens to gain leverage on the collateral, at
whatever price they can sell the tokens for.

To retrieve their collateral they later buyback the same amount of tokens that
they sold, to burn and unlock the collateral.

The sale and buyback is entirely a free market, the underlying mechanisms mean
that cy* token supply will increase when leverage is in demand, pushing price
down, and subsequently incentivising deleveraging which involves burning and
supply decrease.

At the base layer, this means the protocol can offer no liquidations and no
interest on leverage positions, by leaving all of that risk to the free market.
This is an important mechanic and UVP, but doesn't mean we can't add tokenomics
layers on top of this to achieve specific goals
(similar to how nexus offer defi smart contract insurance direct to users who
feel they need it, and will pay a premium for it).

However, many people will only accept leveraged positions if they have more price
certainty on their future unlocks.

In this case, an options layer as described above on Rain would allow users to
opt-in to a free market of price-insured positions on Cyclo.

For example, Alice could deposit 1 ETH at $4k, minting 4k cyWETH. Let's say the
market for cyWETH is $0.5, so she sells her cyWETH for $2k.

She then negotiates an agreement with Bob for an effective future rate for $0.52
per cyWETH. Bob also locks 1 ETH to put up 4k cyWETH for Alice, and receives the
$0.02 premium.

When the agreement finalises, Alice will either see the market is above $0.52 and
so pays Bob $0.52 (total, including the up front premium) to take his cyWETH to
unlock her position and receive her 1 ETH. In this case Bob has been paid the
premium, but has to wait for the market to return below $0.52 for him to unwind
his own position profitably, using the full payment he received from Alice.

The alternative is that the market is below $0.52 so Alice buys back her cyWETH
on the open market and allows Bob's lock to timeout. In this case Bob can simply
take his cyWETH back and unlock his 1 ETH collateral, and keep the premium paid
to him. Alice may even profit if the market is sufficiently cheap relative to her
entry to offset the cost of the premium, e.g. if the market is trading at $0.45
then the $0.05 difference vs. her $0.5 initial sale offsets the $0.02 premium she
paid to Bob.

For Cyclo, the market is designed to natively incentivise returning to the long
term range. Therefore, the higher the market is in the above example, the less
premium Bob should need to charge, because it is increasingly unlikely that he
will be in a position where he cannot immediately retrieve his collateral
+ premium paid to him.

For example, if cyWETH was trading at $0.1, which would be a historically low
price based on market data, Bob would have to charge a very high premium. Most
likely what would happen is the market recovers to say $0.2-0.3+ and so Bob needs
the premium he charges to reflect that he'll be forced to buyback somewhere in
that range to retrieve his ETH collateral.

### Rainflow

Rainflow is a smart contract that enables programmatic escrow type token
movements controlled by Rainlang.

The above system can be modelled as an escrow where:

1. Bob proposes some Rainlang that controls the finalisation of the agreement
2. Bob approves the tokens that Alice will pay a premium to lock
3. Alice reviews Bob's proposal and accepts it by paying the premium
    - At this point Bob's tokens are pulled into the escrow from his wallet
5. When Bob's agreement finalisation logic returns without error the agreement
   can finalise
6. Either Alice will make final payment and receive tokens or Bob will receive
   the return of his tokens

Note that because Bob's tokens are pulled lazily upon acceptance by Alice, he
could have several different agreements proposed at the same time, and only the
first one to be accepted by Alice will pull and lock his tokens.

Everything can be standardised in the Solidity as a factory except for Rainlang
expressions that determine:

- Logic that restricts Alice from starting the lock on Bob's tokens
    - E.g. premium X must be paid for Y tokens at price Z
- Logic that restricts Alice from finalising payment and taking Bob's locked tokens
    - E.g. T time has passed since the lock started
- Logic that restricts Bob from exiting his tokens without payment from Alice
    - E.g. T + t time has passed since the lock started

By implementing the conditions as Rainlang expressions, Bob is free to propose
different conditions under which the agreement is finalized.

For example, Bob could dynamically calculate the premium that Alice must pay, so
that the exact price of the agreement is set in the moment that Alice pays for
and locks Bob's tokens.

Another example is that Bob could allow Alice to make payment at any time before
the expiration time, converting the European style approach described above
(contract finalised at specific time) into an American style
(Alice can pay and exit any time until the specific time).

As the logic is onchain and agreements can be bytecode equivalent, the GUI can
define standard agreements and make an optimized UX for these.

As the logic is Rainlang, it does not need to be compiled ahead of time with
specialised tools (e.g. unlike Solidity). The onchain Rainlang interpreter is
self contained tooling that the SDK uses under the hood in the GUI.