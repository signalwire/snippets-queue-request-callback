# Snippets Queue Request Callback
This snippets will show you how to request a call back from a voice call or text message.
## About Queue Request Callback
Have your customers keep their sanity, while avoiding long hold times by offering a callback option.  Customers can call and request a call back by pressing a digit, or they can text message the number and request a call back when the next agent is availible.
## Getting Started
You will need a machine with Python installed, the SignalWire SDK, a provisioned SignalWire phone number, and optionaly Docker if you decide to run it in a container.

For this demo we will be using Python, but more languages may become available.

- [x] Python
- [x] SignalWire SDK
- [x] SignalWire Phone Number
- [x] Docker (Optional)
----
## Running Queue Request Callback - How It Works
## Methods and Endpoints

```
Endpoint: /text_request_call
Methods: GET OR POST
Endpoint to accept incoming text messages from SignalWire space.
```

```
Endpoint: /enter_queue
Methods: GET OR POST
Endpoint to place callers into queue.
```

```
Endpoint: /wait_url
Methods: GET OR POST
Endpoint to handle people on hold in queue.
```

```
Endpoint: /request_callback
Methods: GET OR POST
Endpoint to process request call back.
```

```
Endpoint: /connect_agent
Methods: GET OR POST
Endpoint to connect agent to caller.
```

## Setup Your Enviroment File

1. Copy from example.env and fill in your values
2. Save new file callled .env

Your file should look something like this
```
## This is the full name of your SignalWire Space. e.g.: example.signalwire.com
SIGNALWIRE_SPACE=
# Your Project ID - you can find it on the `API` page in your Dashboard.
SIGNALWIRE_PROJECT=
#Your API token - you can generate one on the `API` page in your Dashboard
SIGNALWIRE_TOKEN=
# The phone number you'll be using for this Snippets. Must include the `+1` , e.g.: +15551234567
SIGNALWIRE_NUMBER=
```

## Build and Run on Docker
Lets get started!
1. Use our pre-built image from Docker Hub 
```
For Python:
docker pull signalwire/snippets-queue-request-callback:python
```
(or build your own image)

1. Build your image
```
docker build -t snippets-queue-request-callback .
```
2. Run your image
```
docker run --publish 5000:5000 --env-file .env snippets-queue-request-callback
```
3. The application will run on port 5000

## Build and Run Natively
For Python
```
1. Replace enviroment variables
2. From command line run, python3 app.py
```

----
# More Documentation
You can find more documentation on LaML, Relay, and all Signalwire APIs at:
- [SignalWire Python SDK](https://github.com/signalwire/signalwire-python)
- [SignalWire API Docs](https://docs.signalwire.com)
- [SignalWire Github](https://gituhb.com/signalwire)
- [Docker - Getting Started](https://docs.docker.com/get-started/)
- [Python - Gettings Started](https://docs.python.org/3/using/index.html)

# Support
If you have any issues or want to engage further about this Signal, please [open an issue on this repo](../../issues) or join our fantastic [Slack community](https://signalwire.community) and chat with others in the SignalWire community!

If you need assistance or support with your SignalWire services please file a support ticket from your Dashboard. 
