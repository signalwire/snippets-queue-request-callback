# Queue Request Callback
In a past [blog](https://signalwire.com/blogs/industry/best-phone-menu-practices), we talked about the importance of having a concise phone menu so that customers don't have to wait through your IVR in order to resolve an issue. But what happens if a lot of customers call at once, making long hold and wait times that may make your customers aggravated as they stay on the line for extended periods of time? Have your customers keep their sanity while avoiding long hold times by offering a callback option.  Using the [SignalWire Python SDK](https://developer.signalwire.com/twiml/reference/client-libraries-and-sdks#python), customers can call and request a call back by pressing a digit, or they can text message the number and request a call back when the next agent is available.

This code runs by navigating through a series of endpoints that accepts incoming text messages from your SignalWire Space in order to place and handle callers in a callback queue. Once the call-back request is processed, the agent will be connected back to the caller. 

# Setup Your Environment File

1. Copy from example.env and fill in your values
2. Save new file called .env

Your file should look something like this
```
## This is the full name of your SignalWire Space. e.g.: example.signalwire.com
SIGNALWIRE_SPACE=
# Your Project ID - you can find it on the `API` page in your Dashboard.
SIGNALWIRE_PROJECT=
#Your API token - you can generate one on the `API` page in your Dashboard
SIGNALWIRE_TOKEN=
# The phone number you'll be using for this guide. Must include the `+1` , e.g.: +15551234567
SIGNALWIRE_NUMBER=
```

# Configuring the code 

Our first route `/text_request_call` will listen for text messages to process, send a message back to anyone who texts in, and process the call when the timer (for demo purposes) is complete. In a real example, you would wait for your queue to lessen and place the call when an agent is available. 

```python
@app.route('/text_request_call', methods=['GET', 'POST'])
def text_request():
    # Setup the Signalwire client to process commands 
    client = signalwire_client(os.environ['SIGNALWIRE_PROJECT'], os.environ['SIGNALWIRE_TOKEN'], signalwire_space_url = os.environ['SIGNALWIRE_SPACE'])

    # Read the 'From' variable from the GET/POST request
    number = request.values.get('From')

    # Send a text message back to the number that sent it
    message = client.messages.create(
        from_ = os.environ['SIGNALWIRE_NUMBER'],
         body = "This is a message from Signalwire, thank you for requesting a call back.  We will call you in about 3 min.",
           to = number
    )

 # Sub routine to process the call and execute the connect agent script, for this demo, we inserted a timer for demonstration purposes
    def do_call(number):

        time.sleep(120)

        # Place the call, and on connection execute connection agent script
        call = client.calls.create(
            from_ = os.environ['SIGNALWIRE_NUMBER'],
              url = "http://" + os.environ['HOSTNAME'] + "/connect_agent",
               to = number
        )

    # Run sub on seperate thread, so call does not block REST request
    t = threading.Thread(target=do_call, args=[number,])

    # Start thread
    t.start()

    return "200"
```

Our next endpoint `/enter_queue` will serve as the entry point for putting callers in the queue. We will thank the caller for their call and place them into the `DemoQueue1` queue. We will then redirect them to the `/wait_url` route which will play music and handle the on-hold portion of the call. 

```python
# Listen on route '/enter_queue' for GET/POST requests
@app.route('/enter_queue', methods=['GET', 'POST'])
def enter_queue():

    # Make an instance of Signalwire VoiceResponse
    response = VoiceResponse()

    # Synthesize text to speech, using Say verb
    response.say('Thank you for calling SignalWire Demos.')

    # Drop caller into DemoQueue1 and send to onhold /wait_url
    response.enqueue('DemoQueue1', wait_url='/wait_url')

    # Return the VoiceResponse as a string
    return str(response)
```

In the `/wait_url` route, we will ask the user to hold and then play an audio file. After the audio file has ended, we will repeat the prompt and advise that the caller can press 1 to request a callback. If the caller presses one, they will redirect to the `/request_callback` route. If they do not, the audio file/prompts will continue to loop intermittently until the caller is next up in the queue. 

```python
# Listen on route '/wait_url' for GET/POST requests
@app.route('/wait_url', methods=['GET', 'POST'])
def wait_url():

    # Make instance of VoiceResponse
    response = VoiceResponse()

    # Make instance of Gather
    gather = Gather(action='/request_callback', input='dtmf', timeout="3", method='GET')
    # Append Say verb to Gather verb
    gather.say('Please hold for the next available representative.')
    # Append Play verb to Gather verb
    gather.play('https://sinergyds.blob.core.windows.net/signalwire/snoopclose.wav')
    # Append Say verb to Gather verb
    gather.say('Your call is very important to us, a representative will be with you shortly. To request a call back, press 1 and you will not lose your place in line.')
    # Append Play verb to Gather verb
    gather.play('https://sinergyds.blob.core.windows.net/signalwire/8d82b5_The_Muppet_Show_Theme_Song.mp3')
    # Append Gather to Response
    response.append(gather)

    # Return the VoiceResponse as a string
    return str(response)

```

If the caller presses 1 while on hold, they will be redirected to the `/request_callback` route. In this route, we will play a short message to let the caller know that they have been queued for callback and hang up the call. We will then enact the same routine as in the first route - we will use a timer to demo the caller being in a queue and then place a call. If you are using this example in practice, you can initiate the call whenever the next agent is ready to connect instead. 

```python
# Listen on route '/request_callback' for GET/POST
@app.route('/request_callback', methods=['GET', 'POST'])
def request_callback():

    # Make instance of VoiceResponse
    response = VoiceResponse()

    # Use Say verb for TTS
    response.say('OK. Your call has been queued for a call back, We will call you back shortly.  Thank you.')
    # Use Hangup verb to hang up the call
    response.hangup()

    # Read number to call back from From GET/POST request
    number = request.values.get('From')

    # Subroutine to handle call back
    def do_callback(number):

        # Setup a Signalwire Client
        client = signalwire_client(os.environ['SIGNALWIRE_PROJECT'], os.environ['SIGNALWIRE_TOKEN'], signalwire_space_url = os.environ['SIGNALWIRE_SPACE'])

        # For demo purposes, sleep for 15 seconds before sending text
        time.sleep(15)

        # Send a text message to tell user, thank you and an estimated call back time.
        text = client.messages.create(
            from_ = os.environ['SIGNALWIRE_NUMBER'],
             body = "This is a message from Signalwire, thank you for requesting a call back.  We will call you in about 3 min.",
               to = number
        )

        # For demo purposes, sleep for 2 min, then continue
        time.sleep(120)

        # Perform the call back to user, and execute connect agent script
        call = client.calls.create(
            from_ = os.environ['SIGNALWIRE_NUMBER'],
              url = "http://" + os.environ['HOSTNAME'] + "/connect_agent",
               to = number
        )
    # Create a new thread to run call back, so web request is not blocking
    t = threading.Thread(target=do_callback, args=[number,])
    # Start the thread
    t.start()

    # return the VoiceResponse as a string
    return str(response)

```

Once the agent is connected to the caller, the action `url` will redirect to `/connect_agent` where we will let the user know that we are executing the callback they requested, play a short audio file, then hang up. In practice, you can play a short message and skip the audio file as your customer/agent will now be connected, so nothing further is needed!

```python 

@app.route('/connect_agent', methods=['GET', 'POST'])
def connect_agent():

    # Create an instance of VoiceResponse
    response = VoiceResponse()
    # Use Say verb for TTS
    response.say('This is signalwire calling you back from your request earlier.')
    # Use Play verb for audio file playback
    response.play('https://sinergyds.blob.core.windows.net/signalwire/snoopclose.wav')
    # Use Say verb for TTS
    response.say('For more excellent demos and starting projects, visit signal wire dot com')
    # Hangup the call
    response.hangup()

    # Return VoiceResponse as a string
    return str(response)
```

# Build and Run on Docker


1. Use our pre-built image from Docker Hub 
```

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

# Build and Run Natively

To run the application, execute export FLASK_APP=app.py then run flask run.

You may need to use an SSH tunnel for testing this code if running on your local machine. – we recommend [ngrok](https://ngrok.com/). You can learn more about how to use ngrok [here](https://developer.signalwire.com/apis/docs/how-to-test-webhooks-with-ngrok). 

# Sign Up Here

If you would like to test this example out, you can create a SignalWire account and space [here](https://m.signalwire.com/signups/new?s=1).

Please feel free to reach out to us on our Community Slack or create a Support ticket if you need guidance!
