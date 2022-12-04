+++ 
draft = false
date = 2022-12-04T13:32:00+02:00
title = "5 things I learned from 18 months with OPC UA"
description = ""
slug = "2022-12-04-5-things-from-18-months-opcua"
authors = ["Marcus Ilgner <mail@marcusilgner.com>"]
tags = ["opcua", "industry"]
categories = []
externalLink = ""
series = []
+++
# 5 Things I learned from 18 months with OPC UA

Although it feels like a much longer time, it was in May 2021 that I joined [the team at RE:](https://www.re-digital.io/). Now, 18 months later, it feels like a good time for a recap. I'll start with one of the most prominent topics that I encountered: OPC UA. This job is my first within the context of mechanical engineering and while I have worked with many different technologies and built a large variety of systems over the past two decades, this was a novel experience (and also part of the reasons why I decided to join). Overall it is probably fair to say that software development in this industry works differently from "regular", or "digital-only" software. 

## A short primer: what is OPC UA?

The electronical part of industrial machinery consists of a number of so-called [PLCs](https://en.wikipedia.org/wiki/Programmable_logic_controller) which provide real-time I/O to the sensors, motors and other parts of the machine. They have very specialised responsibilities and in order to keep them affordable, usually don't have that many resources beyond what they need for their core I/O control capabilities. This part is often called "OT" (Operational Technology) to differentiate it from traditional IT.

Communication with these PLCs used to happen via very specialised protocols optimised for bandwidth and low processing complexity. Some of them don't even use TCP but lower levels of the ISO layers; even going so far as to require dedicated wiring for their message bus (i.e. CAN). Some of them are proprietary (like the S7comm protocol used by Siemens), some are open (like Modbus and its Modbus over TCP variant).

One thing most (or all?) of them lack, is an *information model*: even though it's possible to read a variable on the PLC if one knows its address and datatype, there is no built-in mechanism to structure the variable data and one requires external, out-of-band information to access these systems.

Although it also touches on a plethora of different aspects - more on this later -, the primary drivers for adoption of OPC UA in the industry are the open availability of its specification and  its capability to provide both modelling and reflection for the data in the PLC.

## What I learned #1: OPC UA is highly complex and few people understand it

Besides providing an information model, OPC UA also tries to address a couple of other aspects. Unfortunately, its high flexibility and aspirations also make it hard to grasp. Even after reading most and re-reading some of the very extensive specification and documentation, fixing bugs in both client and server libraries and even starting to write my own client library, there are still many important details that I find myself having to look up on demand.

And yes, all the OPC UA implementations that I have worked with so far had bugs. Some very prominent, some which could be worked-around; but overall it doesn't feel like a solid foundation when one pulls in a supposedly widely-used library just to stumble over an issue after only two days of working with it. Unfortunately for people coming from "classical" software development, there's no official BNF notation of the UA protocol and implementors have to rely on interpreting a lot of prose text, jumping back and forth between different parts to get things right, leading to aforementioned issues.

Furthermore, most PLCs are extremely taxed when running a protocol which seems to have been designed with desktop and server systems in mind (i.e. PLCs usually doesn't have AES NI in their CPUs). This reduces throughput and renders the task of consistently reading the state of the PLC into a digital twin very challenging.

{{< notice question "Quo Vadis, OPC UA FX" >}}
Supposedly there is a variant of OPC UA, called OPC UA Field eXchange, which has less overhead and is targeted towards better performance for running on PLCs. It was [announced last year](https://opcconnect.opcfoundation.org/2021/06/opc-ua-fx-field-exchange-release-candidate-1-completed/) to have reached release candidate status. However, when speaking with colleagues in the industry at [K 2022](https://www.k-online.com/) it looked like no one was actually aware of it.
{{< /notice >}}

Also, I often feel like having read the actual specification is quite rare, so when using native terminology like *ExtensionObject*, *DataValue* and *ReadValueId*, I often get the feeling that others don't really know what I'm talking about 🙈.


## What I learned #2: no one uses namespace modelling

As mentioned, the biggest feature of OPC UA is its capability to provide an information model: similar to [JSON Schema](http://json-schema.org/), one can define the structure of ones data, including data types, and share that definition across many PLCs. These definitions are grouped into namespaces, allowing OPC UA servers to pull in schema definitions from multiple locations.

There are valiant efforts underway at [EUROMAP](https://www.euromap.org) to standardize the schemas used in various industries. But unfortunately, due to multiple factors, adoption is still in its infancy. For one, there are usually some vendor- or even machine-specific attributes that, when missing, require additional effort to integrate with the standard. And secondly, with only a small number of applications that can make use of a unified information model, the considerable overhead required for its implementation isn't bound to yield any ROI in the short and mid-term.

For the most part, OPC UA nodes with machine data are just put into the namespace `ns=1` which contains the local namespace of each UA server.

{{< notice tip "Clearly communicating node identifiers" >}}
When using namespaces, make sure to communicate node identifiers using URLs instead of numeric namespace identifiers. The node identifier `ns=2;i=5001` could refer to any number of things - but when specifying it as `nsu=http://opcfoundation.org/UA/DI/;i=5001`, it is clear that it's meant to refer to the [DeviceSet](https://reference.opcfoundation.org/nodesets/3/17529) node of the Device Model specification. The numeric namespace identifier is allowed to change and vary across systems, but the URL is guaranteed to be stable.
{{< /notice >}} 

All in all, the UA server's information model is mostly used for reflection, i.e. inspecting (a.k.a. "browsing") the list of available nodes after the fact.

## What I learned #3: no one uses operation limits & getting message sizes right is hard

Another concept which would be very useful when used correctly, are *operation limits*. These describe [a list of restrictions](https://reference.opcfoundation.org/v104/Core/docs/Part5/6.3.11/) that clients should adhere to when "talking to" the UA server. Unfortunately, most UA servers either don't even include the corresponding node in their schema or provide bogus values like `65536` for all of these items - only to croak when the number of `ReadValueId`s goes beyond ~90.

What's more: these parameters are tied into the maximum message size which is negotiated when establishing the UA connection. Unfortunately, it is not unusual to encounter setups where one sends a request to the `Read` service with 200 `ReadValueId`s, only to receive an error `Bad_ResponseTooLarge` because the server finds itself unable to encode the response into the negotiated message size, even if the operation limits are not violated.

## What I learned #4: subscriptions are usually just RPC

As it turns out, OPC UA has [two concepts called "PubSub"](https://reference.opcfoundation.org/Core/Part14/v105/docs/B): in the regular client-server constellation which most setups use, the client will create the subscription on the server and then regularly poll the server for changes using the ID of the subscription.

{{< notice info >}}
Even though client and server do a handshake on the polling intervals when setting up the subscription, actual PLCs have to cope with varying workloads and should not be expected to adhere to the parameters exchanged in the initial setup phase. Instead, clients should expect to receive `BadTooManyPublishRequests`, `GoodOverload` and/or `BadTcpNotEnoughResources` in their responses and dynamically adjust their polling intervals. 
{{< /notice >}}

Only the second mode follows "traditional" pub-sub data flows, where the server independently announces changes in data through a messaging middleware. While this would be highly advantegous to use, I unfortunately have yet to encounter it in an actual system.

## What I learned #5: OPC UA gives you a big gun and does not care whether you shoot your own feet

Continuing with the information model: OPC UA client UIs usually display the available nodes in the form of a tree. This easy but also very misleading: although nodes and node references make up a directed graph, this graph not specified as being acyclic. It would be a perfectly valid UA setup to have cyclic references in your graph. So any client library will have to take care not to run into infinite loops when programmatically discovering the nodeset. Also, nodes may be linked into the graph any number of times (including 0).

Another tripwire is the type system. Besides allowing custom data structures through ExtensionObjects, which provide challenges to clients that use typed programming languages, OPC UA also allows subclassing other, built-in data types. This can lead to confusing scenarios like [Siemens defining their own numeric and string types](https://support.industry.siemens.com/cs/document/109780313/) as subclasses of the original, built-in datatypes.

Without going into more detail, there is a plethora of things to configure and as a consequence, many things to get wrong. Often, changes made to get something working quickly have potential to get bite ones behind a few months down the line.

## In summary

All in all it can probably be said that OPC UA provides enough functionality that it's the future of industry communication protocols. But it's an arduous journey which will continue to provide hardship to developers and implementors for a number of years to come.
