# Go-Splunk-HTTP
A simple and lightweight HTTP Splunk logging package for Go. Instantiates a logging connection object to your Splunk server and allows you to submit log events as desired. [Uses HTTP event collection on a Splunk server](http://docs.splunk.com/Documentation/Splunk/latest/Data/UsetheHTTPEventCollector).

## Table of Contents ##

* [Installation](#installation)
* [Usage](#usage)

## Installation ##

```bash
go get "github.com/cirrus-logic/splunk-hec-logging"
```

## Usage ##

Construct a new Splunk HTTP client, then send log events as desired. 

For example:

```go
package main

import "github.com/cirrus-logic/splunk-hec-logging"

func main() {

	// Create new Splunk client
	splunk := logging.NewClient(
		nil,
		"https://{your-splunk-URL}:8088/services/collector",
		"{your-token}",
		"{your-source}",
		"{your-sourcetype}",
		"{your-index}"
	)
		
	// Use the client to send a log with the go host's current time
	err := logging.Log(
		interface{"msg": "send key/val pairs or json objects here", "msg2": "anything that is useful to you in the log event"}
	)
	if err != nil {
        	return err
        }
	
	// Use the client to send a log with a provided timestamp
	err = logging.LogWithTime(
		time.Now(),
		interface{"msg": "send key/val pairs or json objects here", "msg2": "anything that is useful to you in the log event"}
	)
	if err != nil {
		return err
	}
	
	// Use the client to send a batch of log events
	var events []logging.Event
	events = append(
		events,
		logging.NewEvent(
			interface{"msg": "event1"},
			"{desired-source}",
			"{desired-sourcetype}",
			"{desired-index}"
		)
	)
	events = append(
		events,
		logging.NewEvent(
			interface{"msg": "event2"},
			"{desired-source}",
			"{desired-sourcetype}",
			"{desired-index}"
		)
	)
	err = logging.LogEvents(events)
	if err != nil {
		return err
	}
}

```

## Splunk Writer  ##
To support logging libraries, and other output, we've added an asynchronous Writer. It supports retries, and different intervals for flushing messages & max log messages in its buffer

The easiest way to get access to the writer with an existing client is to do:

```go
writer := splunkClient.Writer()
```

This will give you an io.Writer you can use to direct output to splunk. However, since the io.Writer() is asynchronous, it will never return an error from its Write() function. To access errors generated from the Client,
Instantiate your Writer this way:

```go
logging.Writer{
  Client: splunkClient
}
```
Since the type will now be logging.Writer(), you can access the `Errors()` function, which returns a channel of errors. You can then spin up a goroutine to listen on this channel and report errors, or you can handle however you like. 

Optionally, you can add more configuration to the writer.

```go
logging.Writer {
  Client: splunkClient,
  FlushInterval: 10 *time.Second, // How often we'll flush our buffer
  FlushThreshold: 25, // Max messages we'll keep in our buffer, regardless of FlushInterval
  MaxRetries: 2, // Number of times we'll retry a failed send
}
```

