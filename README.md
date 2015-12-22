# Telegram Bot API helper for Golang

This package is for building a simple Telegram Bot with webhook interface.

View the [documentation here](https://godoc.org/github.com/meinside/telegram-bot-go).

## Install

```
$ go get github.com/meinside/telegram-bot-go
```

## Generate a self-signed certificate

Generate self-signed certificate & private key with following script:

```bash
#!/bin/bash
#
# Generate self-signed certificates for Telegram Bot API
# (https://core.telegram.org/bots/self-signed)
# 
# created on : 2015.10.08.
# last update: 2015.10.11.
# 
# by meinside@gmail.com

DOMAIN="my.host.com"	# XXX - edit
EXPIRE_IN=3650	# XXX - edit

NUM_BITS=2048	# 2048 bits
C="US"
ST="New York"
L="Brooklyn"
O="Example Company"

PRIVATE_KEY="cert.key"
CERT_PEM="cert.pem"

openssl req -newkey rsa:$NUM_BITS -sha256 -nodes -keyout $PRIVATE_KEY -x509 -days $EXPIRE_IN -out $CERT_PEM -subj "/C=$C/ST=$ST/L=$L/O=$O/CN=$DOMAIN"

# finished
echo "> finished - certificate: $CERT_PEM / private key: $PRIVATE_KEY"
```

Generated *cert.key* and *cert.pem* file will be used in **telegrambot.StartWebhookServerAndWait()** function.

## Usage

```go
// sample code for telegram-bot-go, last update: 2015.12.22.
package main

import (
	"fmt"
	"log"
	"time"

	bot "github.com/meinside/telegram-bot-go"
)

const (
	ApiToken     = "01234567:abcdefghijklmn_ABCDEFGHIJKLMNOPQRST"
	WebhookHost  = "my.host.com"
	WebhookPort  = 8443
	CertFilename = "cert.pem"
	KeyFilename  = "cert.key"

	TypingDelaySeconds = 3
)

func main() {
	client := bot.NewClient(ApiToken)
	client.Verbose = true

	// get info about this bot
	if me := client.GetMe(); me.Ok {
		log.Printf("Bot information: @%s (%s)\n", *me.Result.Username, *me.Result.FirstName)

		// delete webhook
		if unhooked := client.DeleteWebhook(); unhooked.Ok {
			// set webhook
			if hooked := client.SetWebhook(WebhookHost, WebhookPort, CertFilename); hooked.Ok {
				// on success, start webhook server
				client.StartWebhookServerAndWait(CertFilename, KeyFilename, func(webhook bot.Webhook, err error) {
					if err == nil {
						// 'is typing...'
						client.SendChatAction(webhook.Message.Chat.Id, bot.ChatActionTyping)
						time.Sleep(TypingDelaySeconds * time.Second)

						// send message
						message := fmt.Sprintf("I received @%s's message: %s", *webhook.Message.From.Username, *webhook.Message.Text)
						options := map[string]interface{}{
							"reply_to_message_id": webhook.Message.MessageId,
						}
						if sent := client.SendMessage(webhook.Message.Chat.Id, &message, options); !sent.Ok {
							log.Printf("*** failed to send message: %s\n", *sent.Description)
						}
					} else {
						log.Printf("*** error while receiving webhook (%s)\n", err.Error())
					}
				})
			} else {
				panic("failed to set webhook")
			}
		} else {
			panic("failed to delete webhook")
		}
	} else {
		panic("failed to get info of the bot")
	}
}
```

## License

Copyright (c) 2015 Sungjin Han

MIT License

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

