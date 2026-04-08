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
orders (so that the back of house staff can deliver food to the
wait staff when it is ready.)

The server maintains a collection of *orders*. Each order is a collection
of *items*. Full details about orders and items are given in the
[Object Model](#object-model) section.

The *updater* client creates new orders and updates the status of existing
items and orders. The *display* client shows the current state of all active
orders and their constituent items.

The clients and server communicate with each other by sending *messages*.
Full details about messages, their encoding, and the general network protocol
are given in the [Protocol](#protocol) section.

## Restaurant Order System

This section documents the object model and network protocol to be implemented
in the restaurant order system.

### Object Model

The *object model* is the collection of data types representing orders, items, and
their statuses. All of these types are defined in the `include/model.h` header file.

`OrderStatus` is an enumeration type representing the status of an order. Its
members are `INVALID`, `NEW`, `IN_PROGRESS`, `DONE`, and `DELIVERED`. (Note that `INVALID`
is not a valid status, and is used only to represent the absence of a valid
order status.)

`ItemStatus` is an enumeration type representing the status of an item within an
order. Its members are `INVALID`, `NEW`, `IN_PROGRESS`, and `DONE`. (As with
`OrderStatus`, the `INVALID` member is not a valid status.)

The `Order` class represents an order and its constituent items. It has a unique
integer identifier, and `OrderStatus` value, and a collection of item objects.

The `Item` class represents an item within an order. It has an order id (which is
the unique id of the order the item is part of), an integer item id (which is
unique within the overall order), an `ItemStatus` value, a description string, and
an integer quantity (which must be positive).

### Protocol

TODO
