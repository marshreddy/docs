# Webhooks API

## Introduction

Webhooks are becoming a common means of integrating disparate web services. Our implementation of the concept allows developers to subscribe to various events in the WhereIsMyTransport domain.
Currently this only extends to Alerts; human (and machine) readable messages describing changes to the service of a transit network. However other events types are in the pipeline.


# API Reference

[Messenger](https://messenger.whereismytransport.com) is an API and webapp which allows users to broadcast changes to the transit network to other commuters.
Messages are delivered via "channels" and [Transport API](https://platform.whereismytransport.com/api).

If you have access to Messenger API, you can create your own channels. Channels initially start off inactive. They will 
not appear in the channel list when sending messages. Only once a channel has been connected to at least one webhook, 
will it appear to messenger users. To create a channel webhook, you need the name of the target channel - this channel has
to already exist.  

To create a channel webhook, `POST` the following body to `api/messenger/channels/{channelName}/`:
```JSON
{
    "callbackUrl":"{callbackUrl}",
    "description":"{webhookDescription}",
    "handshakeKey":"{keyOfYourChoice}",
    "format":"JSON|proto"
}
```    
All webhooks have the choice to receive posts in one of two formats: JSON or the [protocol buffer format](https://github.com/google/protobuf/). The JSON format is 
simply an alternative serialization of the .proto file. 

Messages from channel webhooks follow the GTFS-R specificiation, however the schema file has been slightly modified to support
proto3. You can download the proto schema in question [here](./gtfs-realtime.proto). The root element is the `FeedMessage`

The authentication scheme used is standard WhereIsMyTransport authentication. Scopes required are `transitapi:all`, `messenger:messagechannels:write`, `messenger:messagechannels:read`
and `messenger:messagechannels:delete`.    

To complete creation of a webhook, several things need to happen. First WhereIsMyTransport's servers send a `POST`
request to the callbackUrl. This initial request is important. It contains the `x-hook-secret` and `x-hook-handshake` headers.
The former header is a key used to validate the authenticity of subsequent requests, whilst the latter is the handshakeKey
specified in the initial request. Naturally if the `x-hook-handshake` value differs from the secret specified, the handshake 
request should be ignored. The webhooks API expects a 2XX response back containing the `x-hook-secret` header so as to acknowledge secret receipt.

Once this handshake is complete, a `201 CREATED` response is returned to the client requesting webhook creation. The body 
contains the webhook ID, which is necessary to delete the resource (Performed through a `DELETE` request, the path being the original endpoint with the webhook ID tacked onto the end; `{originalPath}/{webhookId}`)


Now that the webhook is created, whenever an alert is received which should be sent to the target channel, the webhook API 
makes a `POST` request to the callbackUrl, the body is the message formatted as either a JSON string or a protocol buffer byte stream.
The header contains the `x-hook-signature` field. This field contains a base64 encoded `HMAC-SHA256` hash of the body bytes (the JSON body is UTF8 encoded).
The key of this hash is the `x-hook-secret`. Only if the hash matches, should the user accept the message. 

If you do not have privileged access to an agency's messenger account, or if you wish to target more specific entities in 
the transit network, `api/alerts` allows one to subscribe to alerts in a more general context than simply channels. The webhook 
creation body inherits from the body of `api/messenger/channels`, with an additional field: `informedEntity`. This field emulates
the GTFS-R informed entities field. One can choose to filter by one or more agencies, stop, route or route types. The decision of whether
to send an alert to this webhook, is done on a sum of products basis. For example if you wish to subscribe to all alerts about *agency A's* *North Stop*,
or route 2 on agency B, you would use the following body



```JSON
{
    "callbackUrl":"{callbackUrl}",
    "description":"{webhookDescription}",
    "handshakeKey":"{keyOfYourChoice}",
    "format":"JSON|proto"
    informedEntity:[
        {
            agencyId: "{agencyA ID}",
            stopId: "{northStop ID}"            
        },
        {
            agencyId: "{agencyB ID}",
            routeId: "{route2 ID}"            
        }
    ]
}
```    

The routeType field is represented by an integer corresponding to the GTFS-R types. When an alert is posted, it is sent out in the same manner as the channel endpoint.

