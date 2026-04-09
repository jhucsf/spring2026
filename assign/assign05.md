---
layout: default
title: "Assignment 5: Restaurant Order System"
---

*Note*: Assignment 5 is a double assignment. Each milestone (MS1 and MS2)
is worth 1/6 of the assignments grade for the course, the same as
(individually) Assignments 1–4.

**Due:**

* Milestone 1 due **Mon Apr 20th** by 11pm
* Milestone 2 due **Mon Apr 27th** by 11pm (no late hours allowed)

Note that you may **not** use late hours on Milestone 2.
Please plan accordingly.

## Quick Guide

Here are the high-level steps we recommend for completing the assignment.

For Milestone 1:

1. Implement the `Wire::encode` and `Wire::decode` functions.
   Get all of the unit tests in the `message_tests` unit test program
   working.
2. Implement the `IO::send` and `IO::receive` functions.
   Get all of the unit tests in the `io_tests` test program
   working.
3. Implement the `updater` client and test it
4. Implement the `display` client and test it

For Milestone 2:

1. Implement the `Server::server_loop` member function so that
   the server listens for TCP connections from clients. For each
   client that connects, create a `Client` object, and in a new
   detached thread, call its `chat` member function to communicate
   with the remote client.
2. Implement the `Client::chat` member function sufficiently that
   the client can log in
3. Add functionality to the `Client` and `Server` classes to implement
   the required server functionality. You'll need to add a central
   data structure to `Server` to keep track of orders, and use
   appropriate synchronization so that client threads can access the
   data concurrently.
4. Implement the protocol for communicating with an updater client.
   This will involve code in both `Server` and `Client`.
5. Implement the protocol for communicating with a display client.
   Each `Client` should have a `MessageQueue` object that the
   code in the `Server` object can use to post messages to be
   sent to the remote display client program which the state of
   any order or item changes. You'll need to implement the
   `MessageQueue::enqueue` and `MessageQueue::dequeue` member
   functions.

Milestone 2 tasks 3–5 will likely be the most challenging ones, although they
should only involve a couple hundred lines of code, which should be
fairly straightforward if you have thought about the problem and
determined how to factor the problem into helper functions.

## This Assignment Description is Complicated, Help!

Specifying the intended behavior of a networked application entails
some complexity. This assignment description aims to document everything
you need to know to implement the server and clients for the
restaurant order system.

The good news is that the assignment skeleton file (see
[Getting Started](#getting-started) includes reference executables
for all three programs. You can use them as reference for how
your programs should work. Also, you can use them to test your
programs. For example, in Milestone 1, it will make sense to use
the reference server implementation to test your client program
implementations against.

Also, the two unit test programs should make it fairly straightforward
to implement encoding, decoding, sending, and receiving of messages.
Once that code works, implementing the actual application protocol
is relatively easy, and fun!

## Grading Criteria

TODO

## Getting Started

To get started, download [csf\_assign05.zip](csf_assign05.zip) and unzip it.

In the extracted `csf_assign05` directory, the `include` directory has header
files and the `src` directory has C++ source files. The `build` directory is
where compiled object files and executable files will be generated.

## Overview

In this assignment, you will implement two network clients and a network server
which together form a restaurant order system. The general idea is that
this system could be used to keep track of current orders in a restaurant,
encompassing point of sale (order entry), display (so the workers in the
kitchen can see what needs to be prepared), and updating the status of
orders (so that the back of house staff can see which items still need to
be prepared, and deliver the food to the wait staff when the items are
ready.)

The server maintains a collection of *orders*. Each order is a collection
of *items*. Full details about orders and items are given in the
[Object Model](#object-model) section.

The *updater* client creates new orders and updates the status of existing
items and orders. The *display* client shows the current state of all active
orders and their constituent items.

The clients and server communicate with each other by sending *messages*.
Full details about messages, their encoding, and the general network protocol
are given in the [Protocol](#protocol) and [Encoding](#encoding) sections.

## Restaurant Order System

This section documents the object model and network protocol to be implemented
in the restaurant order system.

### Object Model

The *object model* is the collection of data types representing orders, items, and
their statuses. All of these types are defined in the `include/model.h` header file.

`OrderStatus` is an enumeration type representing the status of an order. Its
members are `OrderStatus::INVALID`, `OrderStatus::NEW`, `OrderStatus::IN_PROGRESS`,
`OrderStatus::DONE`, and `OrderStatus::DELIVERED`. (Note that `OrderStatus::INVALID`
is not a valid status, and is used only to represent the absence of a valid
order status.)

`ItemStatus` is an enumeration type representing the status of an item within an
order. Its members are `ItemStatus::INVALID`, `ItemStatus::NEW`, `ItemStatus::IN_PROGRESS`,
and `ItemStatus::DONE`. (As with `OrderStatus`, the `ItemStatus::INVALID` member
is not a valid status.)

The `Order` class represents an order and its constituent items. It has a unique
integer identifier, and `OrderStatus` value, and a collection of item objects.

The `Item` class represents an item within an order. It has an order id (which is
the unique id of the order the item is part of), an integer item id (which is
unique within the overall order), an `ItemStatus` value, a description string, and
an integer quantity (which must be positive).

The `Order` and `Item` classes have a variety of accessor functions for inspecting
and modifying their data.

### Messages

A *message* is a bundle of information sent from client to server (a "request")
or from server to client (a "response"). The `Message` class, defined in
`include/message.h`, represents one message.

The `MessageType` enumeration defines the various types of messages. These will
be described in more detail in the [Protocol](#protocol) section.

The `Message` class is designed to be able to represent any message.
Each message type has a specific combination of data values it contains.
So, the `Message` class's fields and accessor functions represent the
union of all data values a single message could contain.

### Protocol

The following table summarizes the message types, which program sends
messages of that type, and what information a message of that type will
contain.

Message type                      | Sent by            | Reeived by         | Contained data values
--------------------------------- | ------------------ | ------------------ | ---------------------
`MessageType::LOGIN`              | updater or display | server             | client mode, string
`MessageType::QUIT`               | updater            | server             | string
`MessageType::ORDER_NEW`          | updater            | server             | order
`MessageType::ITEM_UPDATE`        | updater            | server             | order id, item id, item status
`MessageType::ORDER_UPDATE`       | updater            | server             | order id, order status
`MessageType::OK`                 | server             | updater or display | string
`MessageType::ERROR`              | server             | updater or display | string
`MessageType::DISP_ORDER`         | server             | display            | order
`MessageType::DISP_ITEM_UPDATE`   | server             | display            | order id, item id, item status
`MessageType::DISP_ORDER_UPDATE`  | server             | display            | order id, order status
`MessageType::DISP_HEARTBEAT`     | server             | display            | *none*

The protocols implemented by the updater client, display
client, and server are described by the following state machines.
In each state machine, the nodes (circles) represent states,
and the transitions (arrows) represent events. Each transition
involves either sending or receiving a message. The "Start"
node represents the initial state, and the "Done" node indicates
that the conversation has finished and the network connection
will be terminated.

Note that the edge labels in <span style="color: #800080;">purple</span>
represent interactive commands that the user enters.

**Updater state machine**:

<a href="img/assign05/updater-sm.svg">
  <img style="width: 48em;" alt="Updater state machine diagram" src="img/assign05/updater-sm.svg">
</a>

**Display state machine**:

<a style="margin-left: 6em;" href="img/assign05/display-sm.svg">
  <img style="width: 36em;" alt="Display state machine diagram" src="img/assign05/display-sm.svg">
</a>

**Server state machine**:

<a href="img/assign05/server-sm.svg">
  <img style="width: 48em;" alt="Server state machine diagram" src="img/assign05/server-sm.svg">
</a>

The server state machine describes the protocol implementing a conversation
between the server and one client.

Note the following special cases in the server's state machines (indicated with
the \*, †, and ‡ symbols in the state diagram):

\* When a new order is created and added to the collection, the server should
enqueue a `MessageType::DISP_ORDER_NEW` message containing the order data to
the message queues of each active display client.

† When a `MessageType::ITEM_UPDATE` message is successfully processed, the server
should enqueue a `MessageType::DISP_ITEM_UPDATE` message containing the item id
and new item status to the message queues of each active display client. Also,
if as a result of applying the item update, the order status transitions from
`OrderStatus::NEW` to `OrderStatus::IN_PROGRESS`, or if the order status
transitions from `OrderStatus::IN_PROGRESS` to `OrderStatus::DONE`, the server
should enqueue a `MessageType::ORDER_UPDATE` with the order id and new order status
to the message queues of each active display client.

‡ When a `MessageType::ORDER_UPDATE` message is successfully processed, the
server should enqueue a `MessageType::DISP_ORDER_UPDATE` message with the
order id and new order status to the message queues of each active display
client.

### Encoding

In order to be sent and received via a TCP connection, a message is represented
as a string, i.e., a sequence of bytes. The `Wire::encode` and `Wire::decode`
functions implement conversion of a `Message` object to and from a string representation.

In the table of message types in the [Protocol](#protocol) section, you
will note that the last column is called "Contained data values". The
entries in this column describe the "payload" of the message. The string
representation of a message consists of the message type, followed by the
contained data values (in order), with all items separated by a single
"`|`" character.

A `MessageType` value can be converted to a string using the
`Wire::message_type_to_str` function.

Integer data values such as order id and item id are encoded as a sequence of
base 10 digits. You can use the `std::to_string` function to do this conversion.

`OrderStatus` and `ItemStatus` values can be converted to a string using
(respectively) the `Wire::order_status_to_str` and `Wire::item_status_to_str`
functions.

`MessageType::ORDER_NEW` and `MessageType::DISP_ORDER_NEW` messages contain an
`Order` as the payload value. An `Order` is converted to a string
consisting of the order id, order status, and item list, separacter by comma
("`,`") characters. The item list is a sequence of 1 or more items, separated
by semicolon (";") characters. Each item is encoded as a string consisting of
order id, item id, item status, description string, and integer quantity,
each separated by colon ("`:`") characters.

Note that you may assume that the separator characters "`|`", "`,`", "`;`", and
"`:`" will never occur in a string value within an encoded message.
