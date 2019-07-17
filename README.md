# Redriver

[![CircleCI](https://circleci.com/gh/forsam-education/redriver.svg?style=svg)](https://circleci.com/gh/forsam-education/redriver)
[![GoDoc](https://godoc.org/github.com/forsam-education/redriver?status.svg)](https://godoc.org/github.com/forsam-education/redriver)
[![Go Report Card](https://goreportcard.com/badge/github.com/forsam-education/redriver)](https://goreportcard.com/report/github.com/forsam-education/redriver)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## When should I use this ?

When you process messages from an SQS Queue, and your processing is not idempotent (processing multiple times the same message would have a negative impact).

You could use the redrive policy to a Dead Letter Queue alone, but what if you only have 1 out of 10 messages that fails in your messages batch ? If you return no errors, the SQS will delete the message. If you return one, your entire batch will go to the DLQ and therefor be re-processed when you will replay the queue.

## How does it work ?

You'll have to define the number of retries and the consumed queue URL.

Then Redriver will use the processing function you'll provide on each event, inside goroutines if you opted for parallelization, and retry them the amount of times you specified.

If everything works well, Redriver will return `nil` when all messages have been processed and you should make your handler return a non-error type so all messages will be deleted.

If some or every messages failed even after retrying them, Redriver will delete the processed messages from the queue and return an error. You should return any error in your handler in this case so every unprocessed messages will be sent to the DLQ specified in your AWS SQS Redrive Policy.

## SQS Redrive Policy

Since this module allows you to do in-code retries, you should set the lambda `maxReceiveCount` parameter to 1 if you use retries in this module.

If you don't do so, the amount of retries done will be `maxReceiveCount * redriverRetries`. It could also be a strategy with "quick" retries effectued by this module in-code, and delayed replay using `maxReceiveCount`.
