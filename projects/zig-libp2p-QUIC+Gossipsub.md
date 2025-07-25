# Implement QUIC+Gossipsub in zig-libp2p

## Motivation

### Zig Advantage

Zig offers several benefits, including a focus on simplicity, explicit control over memory management, and strong C interoperability, making it a compelling alternative to languages like C, C++, and Rust, particularly for systems programming and performance-critical applications.

There are many blockchains that are developed using Zig, including [Solana](https://blog.syndica.io/introducing-sig-by-syndica-an-rps-focused-solana-validator-client-written-in-zig/), [Polkdot jam]( https://jamzig.dev/).The Ethereum client [Lodestar](https://blog.chainsafe.io/lodestar-zig-and-javascript/) is also preparing to migrate to the TS+Zig hybrid client. Migrating the networking module to Zig was a critical task to support the Lodestar migration, which is the core purpose of this project. The implementation of the QUIC transport protocol and the Gossipsub application protocol allows lodestar to use the zig-libp2p network library to implement the functions of the network layer.

### QUIC Advantage

QUIC (Quick UDP Internet Connections) offers several advantages over traditional protocols like TCP, primarily focusing on speed and security. It achieves this by reducing connection establishment time, improving resilience to packet loss, and enhancing multiplexing capabilities. QUIC also integrates encryption, providing a secure connection by default.

## Project description

The lodestar team wanted to rewrite some of its components with Zig to improve performance, and rewriting the p2p module with Zig was one of the key steps. At the same time, the consensus client teams have basically reached an agreement to jointly promote the use of the QUIC transport protocol. In order to achieve this, we need to implement the new QUIC transport in zig-libp2p, as well as the Gossipsub protocol currently used by consensus clients.

1. Utilize existing or create a lsquic-based zig binding for QUIC transport. This is a [library](https://github.com/zen-eth/lsquic.zig) under development.
2. Implement QUIC transport follow by the [libp2p doc](https://docs.libp2p.io/concepts/transports/listen-and-dial/).
3. Implement QUIC transport [interop test](https://github.com/libp2p/test-plans/tree/master/transport-interop).
4. Implement Gossipsub protocol [V1.0](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md).([V1.1](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.1.md) and [V1.2](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.2.md) may be continued after EPF since it may be not enough time.)
5. Implement Gossipsub [interop test](https://github.com/libp2p/test-plans/tree/master/gossipsub-interop).

## Specification

The project consists of two key elements: the QUIC transport protocol and the Gossipsub application protocol. Both components can be subjected to interop testing, which allows the work to be split into three distinct segments.

### QUIC Transport

#### Lsquic Library

Because there is currently no native zig QUIC implementation, in order to quickly support QUIC transport, it is necessary to use the [lsquic library](https://github.com/litespeedtech/lsquic) which has been used for QUIC implementations in many projects. Right now we use [lsquic zig package](https://github.com/zen-eth/lsquic) as a dependency of [zig-libp2p](https://github.com/zen-eth/zig-libp2p) project.

#### Implement QUIC Transport

In libp2p there is a concept of [transport](https://docs.libp2p.io/concepts/transports/overview/), which is mainly responsible for [dialing and listening](https://docs.libp2p.io/concepts/transports/listen-and-dial/).

Follow the design document and other implementations, defined a `Transport` interface in the zig-libp2p. `QuicTransport` could implement this interface using the lsquic library.  

```zig
const std = @import("std");
const conn = @import("conn.zig");

/// Listener interface for accepting incoming connections.
/// This is a type-erased interface that allows
/// different implementations of listeners to be used interchangeably.
/// It uses the VTable pattern to provide a consistent interface for accepting
/// incoming connections. The `acceptFn` function pointer is used to call the
/// appropriate implementation of the accept function for the specific listener
/// instance. The `callback_instance` parameter is an optional user-defined data pointer
/// that can be passed to the callback function. The `callback` function is
/// called when a new connection is accepted. It takes a user-defined data pointer
/// and a result of type `anyerror!conn.AnyRxConn`, which represents the accepted
/// connection. The `anyerror` type is used to represent any error that may occur
/// during the acceptance of a connection.
pub const ListenerVTable = struct {
    acceptFn: *const fn (instance: *anyopaque, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!conn.AnyConn) void) void,
};

/// AnyListener is a struct that uses the VTable pattern to provide a type-erased
/// interface for accepting incoming connections. It contains a pointer to the
/// underlying listener implementation and a pointer to the VTable that defines
/// the interface for that implementation. The `instance` field is a pointer to
/// the underlying listener instance, and the `vtable` field is a pointer to the
/// VTable that defines the interface for that instance. The `accept` function
/// is used to accept incoming connections. It takes an optional user-defined
/// data pointer and a callback function that is called when a new connection
/// is accepted. The callback function takes a user-defined data pointer and a
/// result of type `anyerror!conn.AnyRxConn`, which represents the accepted
/// connection. The `anyerror` type is used to represent any error that may occur
/// during the acceptance of a connection.
pub const AnyListener = struct {
    instance: *anyopaque,
    vtable: *const ListenerVTable,

    const Self = @This();
    pub const Error = anyerror;

    pub fn accept(self: Self, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!conn.AnyConn) void) void {
        self.vtable.acceptFn(self.instance, callback_instance, callback);
    }
};

/// Transport interface for dialing and listening on network addresses.
/// This is a type-erased interface that allows different implementations of
/// transports to be used interchangeably. It uses the VTable pattern to provide
/// a consistent interface for dialing and listening on network addresses.
/// The `dialFn` function pointer is used to call the appropriate implementation
/// of the dial function for the specific transport instance.
/// The `listenFn` function pointer is used to call the appropriate implementation
/// of the listen function for the specific transport instance.
pub const TransportVTable = struct {
    dialFn: *const fn (instance: *anyopaque, addr: std.net.Address, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!conn.AnyConn) void) void,
    listenFn: *const fn (instance: *anyopaque, addr: std.net.Address) anyerror!AnyListener,
};

/// AnyTransport is a struct that uses the VTable pattern to provide a type-erased
/// interface for dialing and listening on network addresses. It contains a
/// pointer to the underlying transport implementation and a pointer to the
/// VTable that defines the interface for that implementation. The `instance`
/// field is a pointer to the underlying transport instance, and the `vtable`
/// field is a pointer to the VTable that defines the interface for that instance.
/// The `dial` function is used to dial a remote address. It takes an address,
/// an optional user-defined data pointer, and a callback function that is
/// called when the dialing operation is complete. The callback function takes
/// a user-defined data pointer and a result of type `anyerror!conn.AnyRxConn`,
/// which represents the established connection. The `listen` function is used
/// to start listening on a local address. It takes an address and returns an
/// `anyerror!AnyListener`, which represents the listener that will accept
/// incoming connections.
pub const AnyTransport = struct {
    instance: *anyopaque,
    vtable: *const TransportVTable,

    const Self = @This();
    pub const Error = anyerror;

    /// Dials a remote address via the underlying transport implementation.
    pub fn dial(self: Self, addr: std.net.Address, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!conn.AnyConn) void) void {
        self.vtable.dialFn(self.instance, addr, callback_instance, callback);
    }

    /// Starts listening on a local address via the underlying transport implementation.
    pub fn listen(self: Self, addr: std.net.Address) Error!AnyListener {
        return self.vtable.listenFn(self.instance, addr);
    }
};

```

> **Note:** In addition to using vtable, other methods such as tagged union can be used in zig to express interfaces. In the future, a thread is specially opened in github discussion to discuss this design.

### Gossipsub Protocol

#### Ambient Peer Discovery

This is out of the scope for this propossal, from the [spec](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#ambient-peer-discovery), for testing purpose, we can use a known list peers as bootstrap peers.

#### Wire Protocol Message

Use a [zig protobuf library](https://github.com/Arwalk/zig-protobuf) to implement codec for messages defined below.

- Common Wire Format
The common wire format which used protobuf is defined [the-rpc section](https://github.com/libp2p/specs/tree/master/pubsub#the-rpc).
- Gossipsub Control Message
The specific control message is defined [control-messages section](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#control-messages).

#### Gossipsub Parameters

It need maintance all these configurable [parameters](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#parameters) in a `GossipsubParams` struct.

#### Core Gossip Router

It need implement the core `GossipsubRouter` follow the [spec](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#router-state), it is the most important component which keeps track all the states.

- Peer State
It need maintain the [core state](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#peering-state) specific for the Gossipsub and common state for the common pubsub framework.

- Message Cache
Implement a [cache](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#message-cache) for messages during the history windows.

#### Heartbeat

Implement the [heartbeat mechanism](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#heartbeat) follow the spec.

#### Topic Membership

It need implement the two topic membership operations JOIN(topic) and LEAVE(topic) follow the [spec](https://github.com/libp2p/specs/blob/master/pubsub/gossipsub/gossipsub-v1.0.md#topic-membership).

#### How To Integrate Transport

zig-libp2p defines a `ConnHandler` interface below, and it also need define a similar `StreamHandler` but which focus on high level protocol message.

```zig
/// Handler interface for handling events in the pipeline.
/// Implementations of this interface should handle the errors and close the connection. And propagate the errors to the pipeline in `onReadFn` and `onActiveFn`.
pub const ConnHandlerVTable = struct {
    /// onActiveFn is called when connection established after dialing or listening. Implementations should close the connection when
    /// it came across an error or when it is no longer needed. Also the error should be propagated and can aware by the pipeline.
    onActiveFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext) anyerror!void,
    /// onInactiveFn is called when the connection is closed and before memory is freed. Implementations should clean up any resources they hold.
    onInactiveFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext) void,
    /// onReadFn is called when a message is received on the connection. Implementations should process the message, and close the connection
    /// if an error occurs or the message is not valid. It should also propagate the error to the pipeline.
    onReadFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext, msg: []const u8) anyerror!void,
    /// onReadCompleteFn is called when the read operation is complete. Implementations close the connection when this is called.
    /// The TailHandlerImpl of pipeline has a default implementation that closes the connection. You could just propagate the call to it.
    /// But you can also implement your own logic.
    onReadCompleteFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext) void,
    /// onErrorCaughtFn is called when an error is caught during the processing of the connection. Implementations should handle the error and close the connection if necessary.
    /// The TailHandlerImpl of pipeline has a default implementation that logs the error and closes the connection. You could just propagate the call to it.
    /// But you can implement your own logic.
    onErrorCaughtFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext, err: anyerror) void,
    /// writeFn is called when a message is to be sent on the connection. Implementations should send the message and close the connection if an error occurs.
    /// The HeadHandlerImpl of pipeline has a default implementation that writes the message to the connection. You could just propagate the call to it.
    /// But you can also implement your own logic.
    writeFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext, buffer: []const u8, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!usize) void) void,
    /// closeFn is called when the connection is to be closed. Implementations should close the connection and clean up any resources they hold.
    /// The HeadHandlerImpl of pipeline has a default implementation that closes the connection. You could just propagate the call to it.
    /// But you can also implement your own logic.
    closeFn: *const fn (instance: *anyopaque, ctx: *ConnHandlerContext, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!void) void) void,
};

/// AnyConnHandler is a struct that holds the instance and vtable for the Handler interface.
pub const AnyConnHandler = struct {
    instance: *anyopaque,
    vtable: *const ConnHandlerVTable,

    const Self = @This();

    pub fn onActive(self: Self, ctx: *ConnHandlerContext) !void {
        return self.vtable.onActiveFn(self.instance, ctx);
    }
    pub fn onInactive(self: Self, ctx: *ConnHandlerContext) void {
        return self.vtable.onInactiveFn(self.instance, ctx);
    }
    pub fn onRead(self: Self, ctx: *ConnHandlerContext, msg: []const u8) !void {
        return self.vtable.onReadFn(self.instance, ctx, msg);
    }
    pub fn onReadComplete(self: Self, ctx: *ConnHandlerContext) void {
        return self.vtable.onReadCompleteFn(self.instance, ctx);
    }
    pub fn onErrorCaught(self: Self, ctx: *ConnHandlerContext, err: anyerror) void {
        return self.vtable.onErrorCaughtFn(self.instance, ctx, err);
    }
    pub fn write(self: Self, ctx: *ConnHandlerContext, buffer: []const u8, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!usize) void) void {
        return self.vtable.writeFn(self.instance, ctx, buffer, callback_instance, callback);
    }
    pub fn close(self: Self, ctx: *ConnHandlerContext, callback_instance: ?*anyopaque, callback: *const fn (instance: ?*anyopaque, res: anyerror!void) void) void {
        return self.vtable.closeFn(self.instance, ctx, callback_instance, callback);
    }
};
```

Follow the [spec](https://github.com/libp2p/specs/tree/master/pubsub#stream-management), it need use two unidirectional streams for reading and writing data.

### Tests

- Interop Test
  - Transport Interop Test: follow the [spec](https://github.com/libp2p/test-plans/tree/master/transport-interop) to do the test.
  - Gossipsub Interop Test: follow the [spec](https://github.com/libp2p/test-plans/tree/master/gossipsub-interop) to do the test.
- Bench Test

## Roadmap

- **Milestone 1(week 5 - 8)** - Implement the `QuicTransport`.
  - epic 1: Implement `listen` feature of the `QuicTransport`.
  - epic 2: Implement `dial` feature of the `QuicTransport`.

- **Milestone 2(week 9 - 12)** - Implement the common pubsub wire message publishing and subscription.
  - epic 1: Implement common wire message protobuf codec.
  - epic 2: Implement the publishing and subscription interfaces.

- **Milestone 3(week 13 - 18)** - Implement the gossipsub route algorithm
  - epic 1: Implement gossipsub control message protobuf codec.
  - epic 2: Implement core router.
  - epic 3: Implement heartbeat mechanism.
  - epic 4: Implement topic membership operations.

- **Milestone 4(week 19 - 21)** - Implement tests
  - epic 1: Implement transport interop test.
  - epic 2: Implement gossipsub interop test.
  - epic 3: Implement bench test.

## Possible challenges

- Currently Zig has no native async/await support, zig-libp2p use the libxev library which use eventloop + callback, the code is less readable. The new [async/await](https://github.com/ziglang/zig/tree/async-await-demo) feature is under development, once it release, it need a huge API refactor for zig-libp2p.
- QUIC is built in security and multiplex which is different from TCP transport, it may need redesign for zig-libp2p.
- The Gossipsub protocol has three versions, but due to EPF timeframe constraints, V1.0 is prioritized for implementation, with efforts focused on passing the interop test.
- How to integrate zig-libp2p to lodestar through FFI.

## Goal of the project

The project seeks to create a P2P network layer entirely in Zig, with the possibility of integrating it into the Lodestar client after it successfully meets performance and stability benchmarks. The initial phase focuses on establishing interoperability of QUIC transport and Gossipsub within libp2p, followed by performance benchmarking to ensure requirements are met. Upon validation, this foundation can support the development of innovative research protocols, such as LEAN Ethereum, tailored to address further needs.

## Collaborators

[@spiral-ladder](https://github.com/spiral-ladder) from the Lodestar team.
[@FoodChain1028](https://github.com/FoodChain1028)  

### Fellows

[@grapebaba](https://github.com/GrapeBaBa)

### Mentors

[@wemeetagain](https://github.com/wemeetagain) from the Lodestar team.

## Resources

- [QUIC](https://quicwg.org/)
- [LSQUIC](https://lsquic.readthedocs.io/en/latest/)
- [Libp2p spec](https://github.com/libp2p/specs)
- [Libp2p interop test](https://github.com/libp2p/test-plans)
- [Protobuf](https://protobuf.dev/)
