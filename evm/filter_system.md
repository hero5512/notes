# Filter: Filtering and Pushing Service

In Ethereum, smart contract events include the contract address that triggered the event and a list of topics (up to 4). RPC provides interfaces for external users to query events within a block range and filter out the events of interest by specifying the address and topics. These events are then pushed to the user in ascending order of blocks.

Note: It is necessary to understand core/bloombits before reading this module.

## Supported Filtering Conditions

The supported filtering conditions are specified by the following struct:

```go
type FilterQuery struct {
   BlockHash *common.Hash     // If set, represents a single block query
   FromBlock *big.Int         // Starting block height of the range (inclusive), nil represents the genesis block
   ToBlock   *big.Int         // Ending block height of the range (inclusive), nil represents the latest block
   Addresses []common.Address // Only matches events emitted by the specified contract addresses
   // Examples:
   // {} or nil          matches any topic list
   // {{A}}              matches topic A in the first position
   // {{}, {B}}          matches any topic in the first position AND B in the second position
   // {{A}, {B}}         matches topic A in the first position AND B in the second position
   // {{A, B}, {C, D}}   matches topic (A OR B) in the first position AND (C OR D) in the second position
   Topics [][]common.Hash
}
```

The rules are as follows:

1. Supports matching a single block by specifying BlockHash, or specifying FromBlock and ToBlock to define a closed range of block heights. Setting FromBlock as nil represents starting from the genesis block, and setting ToBlock as nil represents the latest block.
2. Addresses specify the contract addresses to filter events emitted by.
3. Topics is a two-dimensional array where the first dimension represents the position of the event topic, and the second dimension contains the optional matching items for that position. The filtering condition is that each position must have a match. An empty optional item at a position indicates a match at that position.

Based on the description, the Addresses list can be treated as a special position for topic matching, allowing the two to be unified.

## Filter

This module relies on the following backend interface:

```go
type Backend interface {
	ChainDb() ethdb.Database
	HeaderByNumber(ctx context.Context, blockNr rpc.BlockNumber) (*types.Header, error)
	HeaderByHash(ctx context.Context, blockHash common.Hash) (*types.Header, error)
	GetReceipts(ctx context.Context, blockHash common.Hash) (types.Receipts, error)
	GetLogs(ctx context.Context, blockHash common.Hash) ([][]*types.Log, error)

	SubscribeNewTxsEvent(chan<- core.NewTxsEvent) event.Subscription
	SubscribeChainEvent(ch chan<- core.ChainEvent) event.Subscription
	SubscribeRemovedLogsEvent(ch chan<- core.RemovedLogsEvent) event.Subscription
	SubscribeLogsEvent(ch chan<- []*types.Log) event.Subscription
	SubscribePendingLogsEvent(ch chan<- []*types.Log) event.Subscription

    BloomStatus() (uint64, uint64) // sectionSize(4096), totalSection
	ServiceFilter(ctx context.Context, session *bloombits.MatcherSession)
}
```

The backend provides functions to retrieve blockchain data and subscribe to various events.

The Filter is defined as follows:

```go
type Filter struct {
	backend Backend
	addresses []common.Address
	topics    [][]common.Hash

	block      common.Hash // Query by a single block
	begin, end int64       // Query by a block range
	matcher *

bloombits.Matcher

	mu        sync.RWMutex // Protects the following fields
	logs      []*types.Log
	head      common.Hash
	changes   *changes
	installCh chan struct{} // SubscribeChainEvent installation synchronization
}
```

1. `backend` is the blockchain backend.
2. `addresses` is a list of contract addresses to filter events emitted by.
3. `topics` is a two-dimensional array that defines the topic matching conditions.
4. `block` represents the block hash for querying a single block.
5. `begin` and `end` represent the block range for querying events.
6. `matcher` is the bloombits matcher.
7. `logs` is the filtered event log list.
8. `head` is the latest block hash.
9. `changes` is used to track changes in the blockchain and update the filter accordingly.
10. `installCh` is used for synchronization during installation of the `SubscribeChainEvent` event.

## Interface

The module provides the following interfaces:

- `NewFilter` creates a new Filter instance.
- `GetLogs` retrieves the filtered event logs.
- `MatchTopics` matches the event topics.
- `RetrieveLogs` retrieves the logs from the blockchain backend.
- `PushLogs` pushes the filtered logs to the user.
- `HandleEvent` handles the blockchain events and updates the filter.

## Usage

The module can be used to filter and push Ethereum smart contract events based on specified conditions. It allows users to query events within a block range and filter out events emitted by specific contract addresses or matching specific topics. The filtered events can then be pushed to the user for further processing.

