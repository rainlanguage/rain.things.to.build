# Rain cotrade

## Context

Broadly in the Rain ecosystem, and wider industry, we note two high level audiences.

- People who want to write trading strategies
- People who have capital and want to put it to work on Rain

These two groups of people have some overlap, but not as much as you might think.

The former group largely consists of auto-didactic highly motivated individuals, such as developers, analytical minds and other enthusiasts.

The second group is more purely focussed on finance, which makes sense in "decentralized finance". They like that defi provides access, but they don't necessarily want the responsibility and overhead that comes with the unforgiving autonomy of decentralization.

Both groups have sub-optimal natural incentives.

- Strat writers often have no clear path to scale up any success, and so often stop short of moving past tinkering to battle testing their work, as there's no real carrot for them, other than intellectual curiosity
- Individuals and institutions with capital that would like to deploy on Rain have no clear financial "product" to deploy into, and lack the skills/confidence/etc. to create something themselves

To put it another way, one group has time and motivation, but no capital, while the other group has enough capital to scale, but not enough time and motivation to make use of it.

## Solution

A free market that pairs strat writers with capital providers.

The foundational principle of the market is that:

- The above incentive issues are addressed
- The new market incentives align the needs of both groups

The simplest possible implementation of the marketplace would simply be some kind of "copy trading" system where users can propose strategies and other users can deploy capital into those.

The problem with this simple model is that it fails to incentivise matchmaking in the market. This is because the strat writer still doesn't benefit from the useage of their strategies, in fact they may _suffer_ from too much capital being deployed into their strategy.

For example, consider a peg inefficiency harvesting strategy. A modest amount of capital can make decent profit buying every dip of the peg, but if more and more capital is deployed against the peg chasing the depegs, eventually the token itself will stop depegging and the profit opportunities will become increasingly scarce.

In such an example, if a trader identifies an inefficient peg that they are automatically trading with a strategy, why would they ever share this strategy with others, only to be diluted and have their revenue evaporate?

### Profit sharing

To address the incentive issue above for the strat writers, we can implement a contract that mediates the strat sharing onchain.

The contract would take the funds from the capital providers and deploy the strategy onchain _with the contract as the strategy owner_. This means that neither the strat writer nor the capital provider can directly modify the strategy on the DEX, they have to coordinate all further actions via the contract.

At some point (conditions likely encoded in Rainlang) the strategy winds up, and the final capital would be returned to the capital providers, with profits split between the proposer and the provider.

In this way, the strat writer is incentivised to share their strategies because they will get a smaller share of profit than deploying it themselves, but on a much larger scale.

For example, let's say the strat writer and capital providers agree to split profit 50/50. If a strategy makes 10% profit, then the writer and backers each earn 5%. For the strat writer, this means that as long as the backers can provide at least 2x the capital that the writer could have deployed themselves, they are better off using the backer's money than their own.

This setup solves the incentives for the strat writers, potentially giving them access to millions of $ in capital.

However, the other half of the market still remains unincentivised.

The backers have two key problems here:

- Quality control, there's nothing preventing low effort, high risk strategies being proposed, as the writer only benefits from profits and never experiences any consequences if the strategy trades poorly
- Reason to participate, as the backer could simply look at the strategy being proposed, and copy+paste it into raindex directly and deploy it themselves, retaining 100% of the profits, bypassing paying the strat author

### Writer deposits

To address the incentive gap on the backer side, we add an additional mechanism.

The writer, when seeking backing capital, must deposit some of their own collateral into the strategy themselves.

When the strategy is wound down, assuming it finalizes at breakeven or profit, the deposit is returned to the proposer and the profits are split as above.

However, if the strategy is wound down at a loss, the deposit is partially or fully consumed to cover the losses of the backers _first_ before anything is returned to the proposer. Obviously there are no profits, so the proposer receives nothing from the backer's contribution to the strategy.

This introduces a new mechanic, that backers may want the strategy to wind down earlier than it otherwise would have, in order to ensure that the strategy losses do not exceed the size of the proposer's deposit.

The "can wind down" logic can be expressed in Rainlang, leaving all possible reasons for an exit to be define programatically.

A clear example would be much like how lending platforms such as Aave already behave. If the value of the collateral deposited by a borrower decreases in value to some ratio of their loan, according to some well known oracle, then the protocol liquidates the collateral to internally force clear the loan. The liquidations work effectively like a rain strategy does, as a discounted auction/limit order that solvers connect to external liquidity or take directly.

With this new mechanic, both parties are incentivised to make the strat perform as effectively as possible.

Strat writer:

- Has access to capital that they wouldn't otherwise
- Backers are incentivised to collaborate with them rather than simply copy and compete
- Potential to make much more profit than trading solo
- Maximum downside limited to their deposit

Backer:

- Losses on backed strategies are partially or fully insured by the proposer
- Capture a significant % of the performance of backed strategies
- Early wind down/liquidation handled onchain trustlessly
- Deposits from strat writers passively filter out lazy and high risk strategies automatically
- Free market of strategies means the fees paid for access to insured strategies becomes a commoditized race to the bottom for well known strategies