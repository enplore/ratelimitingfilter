# RateLimitingFilter

The `RateLimitingFilter` is a filter for the Python logging system that allows you to restrict the rate at which messages can pass through your logging handlers.

The filter can be useful if you're using a handler such as Python's `logging.handlers.SMTPHandler` to send error notification emails. Error notification emails provide a useful means of keeping an eye on the health of a running system, but these emails have the potential to overload a mailbox if they start arriving in quick succession due to some kind of critical failure.

The `RateLimitingFilter` can help prevent mailbox overload by throttling messages based on a configurable rate, whilst allowing for initial bursts of messages which can be a useful indicator that something somewhere has broken.

## Basic Usage

A typical use case may be to throttle error notification emails sent by the `logging.handlers.SMTPHandler`. Here's an example of how you might set it up:

```
import logging.handlers
import time

from ratelimitingfilter import RateLimitingFilter

logger = logging.getLogger('throttled_smtp_example')

# Create an SMTPHandler
smtp = logging.handlers.SMTPHandler(mailhost='smtp.example.com',
                                    fromaddr='from@example.com',
                                    toaddrs='to@example.com',
                                    subject='An error has occurred')
smtp.setLevel(logging.ERROR)

# Create an instance of the RateLimitingFilter, and add it to the handler
throttle = RateLimitingFilter()
smtp.addFilter(throttle)

# Create a formatter and set it on the handler
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
smtp.setFormatter(formatter)

# Add the handler to the logger
logger.addHandler(smtp)

# Logged errors will now be restricted to 1 every 30 seconds
while True:
    logger.error('An error message')
    time.sleep(2)
```

Creating an instance of the `RateLimitingFilter` without any arguments like in the example above will restrict the flow of messages to 1 every 30 seconds. 

You can customize the flow rate by supplying your own values for the `rate`, `per` and `burst` attributes. For example, to allow a rate of 1 message every 2 minutes with a periodic burst of up to 5 messages:

```
...

throttle = RateLimitingFilter(rate=1, per=120, burst=5)
smtp.addFilter(throttle)

...
```


## Advanced Usage

It is possible to pass some additional configuration options to the `RateLimitingFilter` when it is initialized for further control over message message throttling.

Perhaps you want to selectively throttle particular error messages whilst allowing other messages to pass through freely. This might be the case if there is part of the application which you know can generate large volumes of errors, whilst the rest of the application is unlikely to.

One way to achieve this might be to use separate loggers, one configured with rate limiting, one without, for the different parts of the application. Alternatively, you can use a single logger and configure the `RateLimitingFilter` to match those messages that should be rate limited.

Applying selective rate limiting like this permits constant visbility of lower volume errors whilst keeping the higher volume errors in check.

### Substring based message throttling
 
You can pass a list of substrings to the `RateLimitingFilter` which it will use to match messages to apply to.
 
```
config = {'content': {'contains': ['some error', 
                                   'a different error']}}

throttle = RateLimitingFilter(rate=1, per=60, burst=1, config)
smtp.addFilter(throttle)

# Can be rate limited
logger.error('some error occurred')
# Can be rate limited
logger.error('a different error occurred')
# Will be not be rate limited
logger.error('something completely different happened')
```

### Automatic message throttling

License
--------

Contributing
-------------


