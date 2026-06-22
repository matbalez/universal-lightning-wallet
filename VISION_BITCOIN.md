# ⚡ Buzz Bitcoin — Value moves with the work

> A maintainer gives an agent a small budget to investigate a bug. The agent pays
> for a private test dataset, asks a specialist agent for a second opinion, and
> posts the result in the branch channel. The maintainer approves the fix. A
> workflow pays the contributor. A teammate tips the explanation that saved
> everyone an hour. One recipient is asleep; their wallet receives anyway.
>
> No platform credits. No custodial Buzz balance. No copy-pasted invoices. Bitcoin
> moves through the same collaboration context as messages, code, agents, and work.

Collaboration already moves information, attention, access, and effort. Value is
part of that exchange, but today's collaboration tools push it into a payment app,
billing system, marketplace, bank account, or proprietary platform ledger.

Buzz should make transmitting value a native part of building together.

Bitcoin is internet-native digital money implemented as an open protocol. It does
not require Buzz to issue a token, run a closed ledger, or make users dependent on
a payment platform. Open-source money belongs beside open-source software,
open-source communications, and open-source intelligence.

The Lightning Network makes Bitcoin useful for small, fast payments. BOLT 12 makes
Lightning a better fit for Buzz: reusable offers, payer-specific invoices,
Lightning-native negotiation, blinded paths, and less dependence on web services
and trusted intermediaries.

The goal:

**Every Buzz user should be able to hold, send, and receive Bitcoin without giving
custody to Buzz — and humans and agents should be able to use value as naturally as
they use messages, reactions, tools, and workflows.**

---

## The Core Primitive

Early Lightning applications were built around BOLT 11 invoices and LNURL. They
proved the demand, but they are a poor foundation for Buzz's native experience.

BOLT 11 invoices are single-use payment requests. LNURL and Lightning Addresses add
reusable payment discovery through an HTTP service that returns BOLT 11 invoices.
That works, but it introduces web infrastructure, domain dependencies, and another
service that observes the request.

BOLT 12 moves the negotiation into Lightning itself. A recipient publishes a
reusable **offer**. A payer uses it to request a unique invoice through Lightning
onion messages, then pays it. Blinded paths can hide the recipient's node identity.
Signed, extensible messages provide better proofs and room for protocol evolution.

For Buzz, BOLT 12 is the core primitive:

- A person, agent, project, channel, or resource can have a durable way to receive.
- A sender can include an amount and payment context without depending on a
  recipient-operated HTTP endpoint.
- Offers can be scoped and rotated for different relationships or purposes.
- The payment protocol stays independent of Buzz and of the selected wallet
  provider.

BOLT 11 remains a required compatibility rail for external Lightning protocols
that still depend on invoices, including L402/HTTP 402 paid resources and legacy
wallet connections. It should be exposed as an explicit wallet capability, while
BOLT 12 remains the durable receive target and data model for Buzz-native
payments.

BOLT 12 also does not make a powered-down wallet receivable by itself. In this
document, an **offline recipient** means the person's Buzz client or phone is
offline while their wallet infrastructure remains available to negotiate and
settle the payment.

---

## Hard Requirements of a Buzz Bitcoin Wallet

The first version has four non-negotiable requirements.

### 1. Every user has a wallet by default

Buzz must be able to programmatically create an embedded wallet during onboarding.
Self-custody is the preferred default where practical, but compatibility is judged
by control, recovery, transparency, and product safety rather than by a marketing
label:

- Buzz and the relay operator cannot custody user funds, extract wallet secrets,
  or unilaterally spend.
- Wallet secrets and spend-authority credentials are never sent to or stored by
  the Buzz relay.
- Any code that exercises the keys is explicitly authorized by the user and runs
  inside a stated security and trust model.
- Backup, recovery, withdrawal, or exit are part of onboarding.
- If the wallet exposes user-held recovery material, the user can inspect it in
  wallet settings; it is hidden by default.
- The user can restore, migrate, withdraw, close, or recover as supported by the
  provider without depending on Buzz continuing to exist.
- The wallet supports basic account operations, including reading the current
  balance and listing recent transactions.

A service may host the user's always-online Lightning node or provide other
payment infrastructure. Hosted, federated, or custodial components are compatible
with Buzz only when the trust assumptions are stated plainly, the user has a
practical exit path, and Buzz itself never becomes the custodian.

### 2. BOLT 12 is the core send and receive primitive

A wallet provider must support both sides of the BOLT 12 flow:

- create reusable offers, including amountless offers where appropriate;
- pay offers;
- receive repeated payments to an offer;
- support blinded paths;
- carry payer messages or payment context where supported;
- report payment state and fees reliably.

Buzz features should exchange BOLT 12 offers or provider-neutral references to
them. Provider-specific node IDs, credentials, and API objects stay behind the
wallet adapter.

### 3. Users can send and receive small amounts immediately

"Immediately" means useful without manual Lightning operations.

A new wallet must be able to receive a small payment without asking the user to
open a channel, find inbound liquidity, wait for manual node setup, or keep the app
open. The provider handles node provisioning, liquidity, routing, and channel
maintenance.

A wallet can send as soon as it has spendable funds. Buzz should be honest about
that boundary: a wallet with a zero balance cannot send. Funding and first receive
should still feel like one onboarding flow rather than a Lightning operations
lesson.

Small payments are the acceptance test. A wallet that works for a large deposit but
cannot economically move a few sats cannot support this vision.

### 4. Users can send to recipients whose clients are offline

The recipient must not need to keep Buzz open, keep a phone awake, approve the
incoming payment, or operate a separate signer or watchtower.

Their wallet infrastructure must remain able to respond to BOLT 12 invoice
requests, settle incoming payments, persist state safely, and notify Buzz when the
client returns.

Messages are asynchronous. Value attached to messages must be asynchronous too.

---

## Future requirement: Connecting an Existing Wallet to Buzz

The embedded wallet solves onboarding. It must not become a permanent wall around
the user.

5. **A user can connect an existing wallet and retain BOLT 12 send and receive,
   small-payment support, and receipt while the Buzz client is offline.**
   

Nostr Wallet Connect, defined by NIP-47, is the established wallet connection
protocol in the Nostr ecosystem. Its current command set centers on BOLT 11
operations such as `pay_invoice` and `make_invoice`; it does not define the BOLT 12
offer operations Buzz needs. It therefore cannot satisfy the complete Buzz wallet
contract today.

Buzz should track NWC's evolution and other work on a BOLT 12-capable wallet
connection protocol. Once a standard is specified, implemented by multiple wallets,
and proven in production, Buzz should adopt it rather than create a permanent
Buzz-specific remote wallet protocol.

The connection standard must support capability discovery, BOLT 12 offer creation
and payment, asynchronous notifications, fee limits, idempotent requests, and
revocable scoped permissions. Connection keys should remain separate from the
user's main Nostr identity.

Buzz should integrate one protocol, then gain access to every compatible wallet. It
should not write a bespoke remote integration for every wallet forever.

---

## Wallet Providers, Not a Wallet Vendor

The Buzz product experience should depend on a stable wallet capability layer, not
on one company's SDK.

```text
Humans · Agents · Workflows · Product features
                    |
                    v
          buzz-wallet capability layer
                    |
       +------------+-------------+
       |            |             |
       v            v             v
   Lexe SDK     Future SDK    Wallet connection
   adapter       adapter         protocol
       |            |             |
       +------------+-------------+
                    |
                    v
       User-selected Bitcoin wallet/provider
                    |
                    v
             Lightning Network
```

The capability layer needs provider-neutral operations for provisioning or
connecting a wallet, discovering capabilities, creating and paying offers, quoting
fees, reading balance and payment state, subscribing to payment events, exporting
recovery or exit material where applicable, and closing the wallet safely.

Product code calls those capabilities. It should not care whether the wallet runs
locally, in remotely attested secure hardware, through an embedded SDK, or behind a
future wallet connection protocol.

A relay may advertise supported providers and recommend a default. The client must
identify the provider and its security model clearly. The user can choose another
compatible provider when available, opt out, and later migrate. A relay
recommendation never gives the relay access to wallet credentials.

A default is an onboarding decision, not an architecture decision.

---

## Requirements for Embedded Wallet SDKs

Embedded wallet SDKs are evaluated against observable behavior and integration
capabilities.

| Area | Expectation |
|---|---|
| **Custody** | Buzz never custodies funds or receives spend authority; provider custody and trust assumptions are explicit. |
| **Provisioning** | A client can create a separate wallet for each user without manual operator work. |
| **BOLT 12** | Create offers, pay offers, receive repeatedly, and support blinded paths. |
| **Offline client receive** | The wallet settles while the user's Buzz client is offline. |
| **Small payments** | New users can receive small amounts without manual channel or liquidity management. |
| **Recovery** | Users have a documented recovery, withdrawal, or exit path that does not depend on Buzz. |
| **Security** | Key storage, signing, remote execution, and update trust are documented and auditable. |
| **Open implementation** | The default provider should be open source; reproducible builds and remote attestation are strongly preferred for hosted secure execution. |
| **SDK quality** | Versioned APIs, structured errors, test environments, and support for Buzz client platforms. |
| **Correctness** | Idempotency, fee limits, terminal states, and reliable payment synchronization. |
| **Privacy** | No unnecessary linkage between social identity, node identity, and payment history. |
| **Economics** | Hosting, liquidity, routing, and service fees are transparent. |
| **Portability** | Provider-specific assumptions remain behind the adapter. |

Capabilities should be discovered at runtime. Buzz should fail closed when a
provider lacks a property required by a feature. It should not silently downgrade a
BOLT 12 feature to a flow with different privacy, trust, or availability.

---

## Initial Provider: Lexe

At the time of writing, Lexe is the only known wallet SDK that appears to satisfy
all four initial requirements together while also matching the preferred
self-custody model:

- programmatic wallet creation with user-controlled recovery;
- an always-online Lightning node for each user;
- BOLT 12 offer creation and payment;
- receipt while the user's application is offline;
- managed liquidity for small incoming payments;
- open-source node and SDK code with a documented secure-enclave and recovery model.

Buzz should integrate the Lexe SDK first.

That is a starting point, not an endorsement carved into the architecture. The Lexe
adapter must sit behind the same provider-neutral layer future wallets use.
Lexe-specific credentials, types, fees, and lifecycle assumptions remain inside the
adapter.

The root seed must be created and controlled by the user's client, not a shared Buzz
backend. Buzz must never log or relay it. Recovery must be tested as a product flow
before users are encouraged to fund the wallet.

The adapter should pass a provider-independent acceptance suite covering wallet
creation, restore, small first receive, offline-client receive, BOLT 12 payment,
duplicate-request safety, fee limits, recovery, and provider failure.

---

## Future Potential Providers

Lexe is the initial provider because it is the first known SDK that satisfies the
full starting requirement set. It should not become the only path. Other providers
can be added when they satisfy the same provider-neutral capability checks and make
their custody, liquidity, availability, and recovery assumptions explicit.

### Bark

Second Tech's layer 2 Ark technology, Bark, recently launched on mainnet. Once it
adds BOLT 12 support, which Second Tech has indicated is coming, Bark could become
an embedded wallet option that ticks all of Buzz's required boxes. That would
introduce some degree of trust in the Ark server, so Buzz would need to expose that
trust model clearly rather than presenting it as equivalent to a self-custodial
Lightning node.

### Spark

Similarly, if Buzz is willing to introduce some degree of trust in Spark servers,
Lightspark's Spark could become an embedded wallet option if it ever adds BOLT 12
support. There is no known indication yet that it will do so.

### Cashu

As an ecash protocol on top of Bitcoin, Cashu could work as an embedded wallet in
Buzz. At the time of writing, there are only two known mints that support BOLT 12,
and their reliability as custodians of real user funds cannot be vouched for.

### Phoenixd

A phoenixd process running on a server the user trusts could be an option if
hosting could be made simple enough. It would not support receiving small amounts
out of the gate, so inbound liquidity and first-receive behavior would need to be
solved before it could satisfy the core wallet requirements.

### LDK

Similarly, an LDK wallet running on a server the user trusts could work, but
receiving small amounts and liquidity more generally still need to be figured out.

### MDK Agent Wallet

MoneyDevKit's Agent Wallet, built on LDK, supports BOLT 12 but needs somewhere
persistent to run.

---

## Value as a Product Primitive

The first milestone is a working wallet. The reason to build it is everything the
wallet makes possible, such as the themes and illustrative examples below.

### Appreciation and attention

- Tip a message, person, agent, project, patch, or document.
- Add sats to kudos without replacing its social meaning.
- Attach a reward to a question or request.
- Fund a bounty directly from an issue or branch channel.
- Let a user publish a price for priority attention or specialized work.
- Let an opt-in channel require a small payment for high-volume unsolicited actions.

Payment is one signal among many. Reputation, membership, context, and human
judgment still matter. Gratitude must not become a pay-to-participate rule.

### Access to resources

- Pay to unlock a private file, dataset, document, or channel.
- Pay per model inference, API request, search, build minute, or compute job.
- Let an agent buy a tool call or data source while completing a task.
- Sell access to a research feed or specialized agent.
- Pay for a temporary capability instead of creating another subscription account.

A payment receipt can satisfy an access policy. The relay coordinates the action;
it does not custody the money.

### Build and earn together

- Pay contributors when a patch is approved or a release ships.
- Split revenue among people, agents, maintainers, and project funds.
- Let workflows turn approved outcomes into payouts.
- Let projects earn from artifacts, support, data, compute, or services.
- Give humans and agents persistent economic identities and earning histories.

The long-term result is a workspace where people and agents discover work, perform
it, prove it, approve it, and get paid in the same shared context.

---

## Agents With Money

Agents make native payments more powerful and more dangerous.

An agent must never receive the user's root seed merely because it can call Buzz
tools. Agents spend through revocable capabilities with explicit policy:

- maximum amount per payment;
- maximum total amount over a time window;
- permitted recipients, projects, tools, or purposes;
- fee limits and expiration;
- human approval above a threshold;
- whether the agent may delegate authority;
- whether incoming funds belong to the agent, its owner, or a project.

Buzz already has workflows, approval gates, identities, and an audit trail. Those
primitives should govern agent payments too.

---

## Identity, Events, and Privacy

Buzz identity and Lightning identity should compose without becoming the same
thing. A user's Nostr public key is meant to be visible. Their balance,
counterparties, payment history, and Lightning node identity are not.

- Do not store seeds, balances, preimages, or complete payment histories on the
  Buzz relay.
- Support scoped and rotatable offers when correlation matters.
- Put only the minimum product-relevant payment state into Buzz events.
- Encrypt payment intents and receipts when the collaboration context is private.
- Avoid public financial graphs or leaderboards by default.

Offer discovery is a Buzz concern. Payment execution is a wallet concern.

A profile may advertise a general offer. A private channel may distribute a scoped
offer only to members. A paid resource may create a purpose-specific offer. The
exact Nostr event schema will be documented in a separate NIP.

Once the embedded wallet requirements are met, especially the requirement that
every user has a BOLT 12 wallet that can immediately receive Bitcoin, Buzz's
protocol surface stays small: broadcast and discover users' BOLT 12 offers on
Nostr as profile metadata. A forthcoming NIP will specify this receive metadata,
letting product features compose on top of the same stable receive primitive. A
forthcoming NIP will also specify a BOLT 12-based zap protocol that replaces
NIP-57's LNURL-based zap mechanism without adding new trust assumptions.

---

## Sequence

### Phase 1 — The wallet

- Define the provider-neutral capability layer and acceptance suite.
- Build the Lexe adapter.
- Ship wallet creation, backup, restore, close, and recovery.
- Create and pay BOLT 12 offers.
- Receive while the Buzz client is offline.
- Show balances, payment state, fees, and useful failures.

### Phase 2 — Value in the room

- Tip people, agents, and messages.
- Attach rewards and bounties to requests, issues, and branch channels.
- Add payment-gated artifacts, resources, and shared inference.
- Add agent budgets, approval gates, and audit traces.
- Add monetized channels.
- Add channel contributor payouts and revenue splits.

### Phase 3 — Wallet choice

- Adopt a BOLT 12-capable wallet connection standard once it is ready and adopted.
- Add more embedded providers through the same adapter boundary.
- Make provider migration and wallet exit routine.

Each phase should be useful on its own. The first shipped feature should not require
the final economy to exist.

---

## The Point

One relay. One identity model. One event log. Humans, agents, workflows, code, and
project memory already share the same space.

Bitcoin lets value share it too.

Buzz should make money part of collaboration without making Buzz the bank. The
wallet belongs to the user. BOLT 12 is the protocol boundary. Providers are
replaceable. Agents operate under explicit authority. Payments remain connected to
the work that gave them meaning.

Open-source software. Open-source communications. Open-source intelligence.
Open-source money.

Value moves with the work.

---

## FAQ

### Why Bitcoin and not stablecoins?

Bitcoin is internet-native money that introduces the least amount of trust in
third parties and the lowest cost for transfers of tiny amounts. Stablecoins
require trust in the stablecoin issuer and in the crypto rails on which the
stablecoins are sent and received. With Bitcoin, particularly through a
self-custodial wallet, users can know their funds are their own instead of a claim
on an issuer.

### Why not Lightning Address / LNURL?

LNURL gives Lightning users a Lightning Address: a stable address at which they
can receive Bitcoin. That is useful, but it requires a web server and usually
introduces another third party to operate, observe, or be trusted in the receive
flow.

BOLT 12 is a trustless way to achieve a stable receive identifier. Several Nostr
clients have already built Lightning payment experiences with LNURL. Buzz should
build around the more trustless BOLT 12 architecture from the beginning.

### Why Lexe?

Lexe was chosen as the initial embedded wallet provider because it meets the core
starting requirements: an SDK with BOLT 12 send, BOLT 12 receive, small incoming
payment support, and receive while the user's Buzz client is offline.

As more embedded wallet alternatives become feasible, they should be added as
user-selectable options in Buzz. Once a wallet connection protocol exists that
lets users connect an existing wallet with the same capabilities, Buzz should
support that too.

---

## References

- [Buzz platform vision](VISION.md)
- [Buzz sovereign vision](VISION_SOVEREIGN.md)
- [Buzz agent vision](VISION_AGENT.md)
- [BOLT 12: Negotiation Protocol for Lightning Payments](https://github.com/lightning/bolts/blob/master/12-offer-encoding.md)
- [NIP-47: Nostr Wallet Connect](https://github.com/nostr-protocol/nips/blob/master/47.md)
- [Lexe SDK](https://github.com/lexe-app/lexe-sdk)
- [Lexe public monorepo and security model](https://github.com/lexe-app/lexe-public)

---

*Buzz 🐝 — value moves with the work.*
