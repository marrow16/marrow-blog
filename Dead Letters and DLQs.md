# Dead Letters & DLQs
_Originally posted 02/09/2018_

## What is a DLQ (Dead Letter Queue)
In any architecture, mircoservices in particular, message queues are used for various reason (e.g. handling high latency tasks, high scalable asynchronous handling of medium latency tasks etc.).  Whatever the reason for using a queue, one overriding principle will have been considered – the requirement that each message has a guaranteed “at least once” delivery.  With a standard FIFO (first in, first out) queue this guaranteed delivery can become problematic – if the current first message out cannot be handled but isn’t removed from the queue, then the entire queue becomes blocked and the system grinds to a halt.  This is the underlying reason for having a DLQ – a place to put messages that simply cannot be handled.

## When to DLQ a message?
In principle, it’s quite an easy decision of when to DLQ a particular message – if the message, no matter how many times redelivered, cannot be processed then it should be DLQ’d.  Some specific examples of when to DLQ a message:

- Bad message – the message payload cannot be unmarshalled (e.g. non well-formed JSON or XML)
- Incoherent message – the message payload could be unmarshalled but didn’t make ‘sense’ to the consumer (e.g. a JSON property value was expected to be a numeric but was actually a string)
- Wrong message – the message payload could be unmarshalled but the ‘instruction’ contained in it is not understood by the consumer
- Invalid message – the message payload could be unmarshalled and was correct and coherent but caused some violation when being processed (e.g. saving the information within the message caused some foreign key violation in a database)

## When not to DLQ a message?
Again, in principle, it should be an easy decision of when not to DLQ a message and let it retry for delivery – if the failure to process the message was transient.  Some specific examples of when not to DLQ a message:

- Resource unavailable – attempting to process the message required access to a resource that was currently unavailable (e.g. required saving to a database but the database connection is lost).  This assumes that the consumer is able, at some point, to regain access to the resource (which should be designed into the consumer).
- Resource deadlock – attempting to process the message encountered some deadlock condition (e.g. saving to a database but there was a transient lock conflict)

 

## Database Transient vs Non-Transient Errors
Determining the difference between transient and non-transient errors in a consumer acting on a database can be tricky.  Fortunately, in Java, most consumers dealing with SQL databases will be going through JDBC – and JDBC conveniently does a pretty good job of categorizing exceptions (SQLNonTransientException, SQLTransientException, SQLRecoverableException) in a way that makes it fairly easy to differentiate between transient and non-transient errors.

## Testing
It is imperative that any DLQ strategy implemented in a consumer is thoroughly tested – specifically forcing error conditions that will show when and when not messages are sent to DLQ.  To facilitate this testing, it is important to modularize the consumer code in such a way that the different aspects of processing a message can be tested in isolation.

## Idempotency
With queues there is a guaranteed “at least once” delivery – and with a system designed to be robust, we should also guarantee that a message may be delivered more than once!  A misbehaving message producer may send the same the same message more than once.  Therefore, message consumers must be designed to be idempotent – so that receiving a repeated message is neither sent to a DLQ nor left on the queue.  The repeated message should either be consumed and ignored – or the processing of it is guaranteed idempotent.

It may be that we want to know about repeated messages – so if repeats can be detected they should be logged.

## What to put in the DLQ message?
Obviously, the most important information to be placed into the DQL message is the original message itself – so that, in theory, the DLQ message could be re-sent (re-routed) to the consumer and processed.  For example, in condition (b), (c) or (d) it may be that the consumer is ‘fixed’ to cope with the messages and we now want to retry the messages.  If the DLQ messages were not original messages, this retrying would be difficult.

It is likely that a consumer, when DLQ-ing a message, may also log a reason.  However, it would be a painful task to correlate these logged reasons back to the DLQ’d message (even if the message correlation ids were also logged).  For this reason, it’s also useful and prudent to add a ‘reason’ property into the DLQ’d message.  Though adding a property to a bad message (i.e. invalid JSON or XM) is not possible – in which case the original message can be ‘stringified’ into a property along with the reason.

## Automatic DLQ-ing?
Many queue providers, such as Amazon SQS, provide mechanisms and settings to allow failed messages to be automatically sent to a DLQ after a given number of retries.  Whilst this is a useful for feature, it should be considered a failsafe mechanism rather than the underpin of a DLQ strategy.  Given adequate monitoring, it should be possible to set this retry threshold extremely high – if we DLQ too easily (a low number of retries) we can be making work for ourselves to recover from a slightly longer than expected transient failure condition.

Be wary of ‘suggested’ automatic DLQ thresholds (e.g. Amazon SQS Dead Letter Queues) as they tend to be far too low.  Automatically sending to DLQ after 5 retries would, in most scenarios, cause numerous messages to be sent to DLQ with a simple, temporary database outage.

## Monitoring
With any system that relies on queues and DLQs, it is imperative that adequate monitoring is in place.  There are  specific things we should be monitoring:

+ Queues filling up – if messages are being sent to a queue but not being dequeued (at all) it is indicative of some long running transient failure condition
+ Queues backing up – if messages are being sent to a queue but not being dequeued at a sufficient rate then it’s likely we have a scaling issue to address (we may need more consumers to cope with the throughput)
+ DLQs filling up – we have a producer that is creating bad messages or consumers that are not capable of processing messages

## What to do with DLQs?
If we go to the trouble of designing consumers that will send messages to a DLQ then we are expecting, at some point, to see messages in that DLQ.  We will have also gone to the trouble of monitoring those DLQs to be notified when they are filling.  It would, therefore, be a little imprudent not go to the trouble of knowing how to handle those DLQ’d messages.  Assuming the information in a message required some action be carried by our system on behalf of our customers – whilst the customer may be tolerant of high latency of those actions, they might be slightly less happy with unreasonable delays in processing those actions.  For example, if a change in the overall system caused a sudden flurry of DLQ messages but the work required to reprocess those messages (after deploying fixes) took days or weeks to happen.  There are ways to mitigate this in advance – by creating tooling that allows for easy re-routing and re-processing of DLQs.

