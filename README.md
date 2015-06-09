# sqs-consumer-attributes

Major portion taken from https://www.npmjs.com/package/sqs-consumer

Build SQS-based applications without the boilerplate. Just define a function that receives an SQS message and call a callback when the message has been processed.

## Usage

```js

var Consumer = require('sqs-consumer');

var app = Consumer.create({
  queueUrl: 'https://sqs.eu-west-1.amazonaws.com/account-id/queue-name',
messageAttributeNames = [
                'user_id'
              ],  
handleMessage: function (message, done) {
    // do some work with `message`
    done();
  }
});

app.on('error', function (err) {
  console.log(err.message);
});

app.start();
```

* The queue is polled continuously for messages using [long polling](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-long-polling.html).
* Messages are deleted from the queue once `done()` is called.
* Calling `done(err)` with an error object will cause the message to be left on the queue. An [SQS redrive policy](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/SQSDeadLetterQueue.html) can be used to move messages that cannot be processed to a dead letter queue.
* By default messages are processed one at a time – a new message won't be received until the first one has been processed. To process messages in parallel, use the `batchSize` option [detailed below](#options).

## API

### `Consumer.create(options)`

Creates a new SQS consumer.

#### Options

* `queueUrl` - _String_ - The SQS queue URL
* `region` - _String_ - The AWS region (default `eu-west-1`)
* `handleMessage` - _Function_ - A function to be called whenever a message is receieved. Receives an SQS message object as its first argument and a function to call when the message has been handled as its second argument (i.e. `handleMessage(message, done)`).
* `batchSize` - _Number_ - The number of messages to request from SQS when polling (default `1`). This cannot be higher than the AWS limit of 10.
* `sqs` - _Object_ - An optional [AWS SQS](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/SQS.html) object to use if you need to configure the client manually

### `consumer.start()`

Start polling the queue for messages.

### `consumer.stop()`

Stop polling the queue for messages.

### Events

Each consumer is an [`EventEmitter`](http://nodejs.org/api/events.html) and emits the following events:

|Event|Params|Description|
|-----|------|-----------|
|`error`|`err`|Fired when an error occurs interacting with the queue or processing the message.|
|`message_received`|`message`|Fired when a message is received.|
|`message_processed`|`message`|Fired when a message is successfully processed and removed from the queue.|
