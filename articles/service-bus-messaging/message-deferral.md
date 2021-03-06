---
title: Azure Service Bus - message deferral
description: This article explains how to defer delivery of Azure Service Bus messages. The message remains in the queue or subscription, but it is set aside.
ms.topic: article
ms.date: 06/23/2020
ms.custom: fasttrack-edit
---

# Message deferral

When a queue or subscription client receives a message that it is willing to process, but for which processing is not currently possible due to special circumstances inside of the application, it has the option of "deferring" retrieval of the message to a later point. The message remains in the queue or subscription, but it is set aside.

Deferral is a feature specifically created for workflow processing scenarios. Workflow frameworks may require certain operations to be processed in a particular order, and may have to postpone processing of some received messages until prescribed prior work that is informed by other messages has been completed.

A simple illustrative example is an order processing sequence in which a payment notification from an external payment provider appears in a system before the matching purchase order has been propagated from the store front to the fulfillment system. In that case, the fulfillment system might defer processing the payment notification until there is an order with which to associate it. In rendezvous scenarios, where messages from different sources drive a workflow forward, the real-time execution order may indeed be correct, but the messages reflecting the outcomes may arrive out of order.

Ultimately, deferral aids in reordering messages from the arrival order into an order in which they can be processed, while leaving those messages safely in the message store for which processing needs to be postponed.

> [!NOTE]
> Deferred messages will not be automatically moved to the dead-letter queue [after they expire](./service-bus-dead-letter-queues.md#exceeding-timetolive). This behaviour is by design.

## Message deferral APIs

The API is [BrokeredMessage.Defer](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.defer#Microsoft_ServiceBus_Messaging_BrokeredMessage_Defer) or [BrokeredMessage.DeferAsync](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deferasync#Microsoft_ServiceBus_Messaging_BrokeredMessage_DeferAsync) in the .NET Framework client, [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync) in the .NET Standard client, and [IMessageReceiver.defer](/java/api/com.microsoft.azure.servicebus.imessagereceiver.defer) or [IMessageReceiver.deferAsync](/java/api/com.microsoft.azure.servicebus.imessagereceiver.deferasync) in the Java client. 

Deferred messages remain in the main queue along with all other active messages (unlike dead-letter messages that live in a subqueue), but they can no longer be received using the regular Receive/ReceiveAsync functions. Deferred messages can be discovered via [message browsing](message-browsing.md) if an application loses track of them.

To retrieve a deferred message, its owner is responsible for remembering the [SequenceNumber](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.sequencenumber#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_SequenceNumber) as it defers it. Any receiver that knows the sequence number of a deferred message can later receive the message explicitly with `Receive(sequenceNumber)`.

If a message cannot be processed because a particular resource for handling that message is temporarily unavailable but message processing should not be summarily suspended, a way to put that message on the side for a few minutes is to remember the **SequenceNumber** in a [scheduled message](message-sequencing.md) to be posted in a few minutes, and re-retrieve the deferred message when the scheduled message arrives. If a message handler depends on a database for all operations and that database is temporarily unavailable, it should not use deferral, but rather suspend receiving messages altogether until the database is available again.


## Next steps

To learn more about Service Bus messaging, see the following topics:

* [Service Bus queues, topics, and subscriptions](service-bus-queues-topics-subscriptions.md)
* [Get started with Service Bus queues](service-bus-dotnet-get-started-with-queues.md)
* [How to use Service Bus topics and subscriptions](service-bus-dotnet-how-to-use-topics-subscriptions.md)
