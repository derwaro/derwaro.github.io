---
title: "Endpoint for Twilio Status Callback in FastAPI"
date: 2023-06-26T18:21:22-06:00
lastmod: 
draft: false
tags: ["fastapi", "api", "twilio", "howto"]
categories: ["tech"]
---
This post will explain how to use a FastAPI endpoint as the `status_callback` for twilio's voice API.
I will use FastAPI, Twilio and ngrok to demonstrate the behavior.
I start of with this FastAPI file, importing the necessary dependencies and describing a simple endpoint to place calls through Twilio's API:

```python
from fastapi import FastAPI
from twilio.rest import Client


# twilio variables. get them from your twilio console:
# https://www.twilio.com/console
account_sid = "YourTwilioAccountSID"
auth_token = "YourTwilioAuthToken"

app = FastAPI()

@app.post("/call_with_twilio")
async def call_with_twilio(
    from_number: str,
    to_number: str,
    audio_url: str,
):
    # init twilio client
    client = Client(account_sid, auth_token)
    
    # send call request to twilio api
    client.calls.create(
        url=audio_url,
        to=to_number,
        from_=from_number,
    )
    return {
        "message": "Call initiated successfully",
        "Recipients; to": to_number,
        "Emitter; from": from_number,
        "Played Audio Message; url": audio_url,
    }
```

So far, so good, as this is pretty much copied directly from Twilio's documentation: Each request to this endpoint triggers a call via Twilio using the provided `from` and `to` phone-numbers and plays the provided audio file from `url`.

Now, I wanted to see whats going on, that is see each step in the processing of each call. Twilio provides a way to do so via a `status_callback` parameter passed in the api call. Let's update our code and add another endpoint `call_status` to process the feedback for each call:

```python {hl_lines=["21-24", "26-41", "53-62"]}
from fastapi import FastAPI
from twilio.rest import Client


# twilio variables. get them from your twilio console:
# https://www.twilio.com/console
account_sid = "YourTwilioAccountSID"
auth_token = "YourTwilioAuthToken"

app = FastAPI()

@app.post("/call_with_twilio")
async def call_with_twilio(
    from_number: str,
    to_number: str,
    audio_url: str,
):
    # init twilio client
    client = Client(account_sid, auth_token)
    # set url for callbacks on call events
    status_callback_url = (
        "https://example.com"
        + "/call_status"
    )
    # send call request to twilio api
    client.calls.create(
        method="GET",
        status_callback=status_callback_url,
        status_callback_event=[
            "queued",
            "initiated",
            "ringing",
            "answered",
            "in-progress",
            "completed",
            "busy",
            "no-answer",
            "canceled",
            "failed",
        ],
        status_callback_method="POST",
        url=audio_url,
        to=to_number,
        from_=from_number,
    )
    return {
        "message": "Call initiated successfully",
        "Recipients; to": to_number,
        "Emitter; from": from_number,
        "Played Audio Message; url": audio_url,
    }

@app.post("/call_status")
async def call_status(request: Request):
    form_data = await request.form()
    decoded_data = {}

    for key, value in form_data.items():
        decoded_data[key] = value

    print(decoded_data)
    return {"decoded_data": decoded_data}
```

I added a variable which holds the full url to `call_status`. If you want to check if this works our, substitute the part before `/call_status` with a ngrok url. Furthermore I added some parameters to the `client.calls.create()` function:
* `method="GET"` : specifies which method to use to call the Twilio API
* `status_callback=status_callback_url` : is the url Twilio will use to continuously send status change events to
* `status_callback_event=["initiated", "ringing", ...]` : on which events Twilio should send feedback. If nothing is specified this defaults to "completed".
* `status_callback_method="POST"` : which method will be used to send the feedback to the specified url

Placing a call through endpoint `call_with_twilio` will now trigger several requests (one for each status change) to `https://example.com/call_status`. In `/call_status` I can then further process the received data: store it in a database, send it to another endpoint, store it in the session,... whatever you wish to do with it.

# Using requests directly
The important thing to note is that Twilio returns the status data not as json, but as formdata[^1]. Furthermore the form does not always contain the same fields. A status report with `"CallStatus": "completed"` has for example formfields for `"SipResponseCode": "200"` and `"Duration": "1"`, which other status don't have. That's why I opted to receive the request directly[^2] and process it from there to a dictionary.

# Using FastAPI's `Form()`
Another possibility to receive the formdata is via FastAPI's `Form()`[^3]:
```python {hl_lines=[1, 2, "56-62"]}
from fastapi import FastAPI, Form
from typing import Annotated
from twilio.rest import Client


# twilio variables. get them from your twilio console:
# https://www.twilio.com/console
account_sid = "YourTwilioAccountSID"
auth_token = "YourTwilioAuthToken"

app = FastAPI()

@app.post("/call_with_twilio")
async def call_with_twilio(
    from_number: str,
    to_number: str,
    audio_url: str,
):
    # init twilio client
    client = Client(account_sid, auth_token)
    # set url for callbacks on call events
    status_callback_url = (
        "https://example.com"
        + "/call_status"
    )
    # send call request to twilio api
    client.calls.create(
        method="GET",
        status_callback=status_callback_url,
        status_callback_event=[
            "queued",
            "initiated",
            "ringing",
            "answered",
            "in-progress",
            "completed",
            "busy",
            "no-answer",
            "canceled",
            "failed",
        ],
        status_callback_method="POST",
        url=audio_url,
        to=to_number,
        from_=from_number,
    )
    return {
        "message": "Call initiated successfully",
        "Recipients; to": to_number,
        "Emitter; from": from_number,
        "Played Audio Message; url": audio_url,
    }

@app.post("/call_status")
async def call_status(
    Called: Annotated[str, Form()],
    ToState: Annotated[str, Form()],
    CallerCountry: Annotated[str, Form()],
    Direction: Annotated[str, Form()],
    Timestamp: Annotated[str, Form()],
    CallbackSource: Annotated[str, Form()],
    (...)):
    

    print(Called)
    print(ToState)
    (...)
    return {}
```

This is just the basic functionality for receiving the data as FormData. As above, adapt the snippet to do whatever you want with the received data[^4]. This version has the drawback that you have to define the fields beforehand and since they change depending on the status, you will have to adapt to this.





[^1]: See the respective HEADER: `Content-Type: application/x-www-form-urlencoded; charset=UTF-8`
[^2]: FastAPI Documentation for using requests directly: https://fastapi.tiangolo.com/advanced/using-request-directly/
[^3]: FastAPI Documentation for receiving formdata instead of json: https://fastapi.tiangolo.com/tutorial/request-forms/
[^4]: A hopefully complete list of fields returned from Twilio ready to use with FastAPI:
    ```
    Called: Annotated[str, Form()],
    ToState: Annotated[str, Form()],
    CallerCountry: Annotated[str, Form()],
    Direction: Annotated[str, Form()],
    Timestamp: Annotated[str, Form()],
    CallbackSource: Annotated[str, Form()],
    SipResponseCode: Annotated[str, Form()],
    CallerState: Annotated[str, Form()],
    ToZip: Annotated[str, Form()],
    SequenceNumber: Annotated[str, Form()],
    CallSid: Annotated[str, Form()],
    To: Annotated[str, Form()],
    CallerZip: Annotated[str, Form()],
    ToCountry: Annotated[str, Form()],
    CalledZip: Annotated[str, Form()],
    ApiVersion: Annotated[str, Form()],
    CalledCity: Annotated[str, Form()],
    CallStatus: Annotated[str, Form()],
    Duration: Annotated[str, Form()],
    From: Annotated[str, Form()],
    CallDuration: Annotated[str, Form()],
    AccountSid: Annotated[str, Form()],
    CalledCountry: Annotated[str, Form()],
    CallerCity: Annotated[str, Form()],
    ToCity: Annotated[str, Form()],
    FromCountry: Annotated[str, Form()],
    Caller: Annotated[str, Form()],
    FromCity: Annotated[str, Form()],
    CalledState: Annotated[str, Form()],
    FromZip: Annotated[str, Form()],
    FromState: Annotated[str, Form()]```