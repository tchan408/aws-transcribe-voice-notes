# Amazon Transcribe for Voice Notes created from Vonage Client SDK

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/nexmo-se/aws-transcribe-voice-notes)

Use this Transcribe reference connection for transcription of voice notes created from the Client SDK in a named conference.

## Amazon Transcribe reference connection

In order to get started, you will need to have an [AWS account](http://aws.amazon.com).

You will also need to know an active AWS Access Key ID and Secret Key pair.

If necessary, create a new pair of keys:
- Log in to your [AWS Management Console](http://aws.amazon.com).
- Click on your user name at the top right of the page.
- Click on the My Security Credentials link from the drop-down menu.
- Go to Access keys section,
- Click on \[Create New Access Key\] (\*)

(\*) *Note: Your AWS account may be limited to only 2 active Access Keys. To create a new pair of Keys, you may need to "Make Inactive" an existing active Access Key ID, however before doing so, you need to absolutely make sure that key is not used by your other applications.*

## About this reference connection

Vonage Voice API's Amazon Transcribe reference connection for Voice Notes transcribe audio clips created with Vonage API Client SDK (WebRTC client).

The Voice Note audio file must be converted to PCM 16 bits 16 kHz mono before submitting it via HTTP POST to this reference connection.

Your Vonage Voice API application uses HTTP POST to the reference connection address with the follwing requirements:
- The Voice Note audio file as binary payload for the HTTP POST body,
- Must include at least the following query parameters:
	- _**webhook_url**_ (e.g. https://myserver.mycompany.com:32000/transcript) where the transcript will be posted by the reference connection
	- _**language_code**_ (e.g. en-US), which defines the transcription language
- Your application may send/use any additional query parameter names and values for your application logic needs, except it **may not** use/send the following reserved query parameter names:
	- _**transcript**_
	- _**service**_ 

A few seconds later, the reference connection posts back to your Vonage Voice API application webhook_url a JSON formatted payload (in the body of an HTTP POST):</br>
	- the _**"transcript"**_,</br>
	- the name of the _**"service"**_, which is "AWS Transcribe" in this case,</br> 
	- and all other values sent as query parameters of the original request to the reference connection, e.g. _**"webhook_url"**_, _**"language_code"**_, and any additional query parameters that have been sent in the original HTTP POST</br>

## Running Transcibe reference connection for Voice Notes

You may select one of the following 4 types of deployments.

### Docker deployment

Copy the `.env.example` file over to a new file called `.env`:
```bash
cp .env.example .env
```

Edit `.env` file,<br/>
set the 3 first parameters with their respective values retrieved from your AWS account,<br/>
set the `PORT` value (e.g. *5000*) where transcriptions requests will be received.
The `PORT` value needs to be the same as specified in `Dockerfile` and `docker-compose.yml` files.

Launch the Transcribe & Comprehend reference connection as a docker container instance:

```bash
docker-compose up
```
Your docker container's public hostname and port will be used by your Vonage Voice API application as the address to where to submit the transcription request `https://<docker_host_name>:<proxy_port>/transcribe`, e.g. `https://myserver.mydomain.com:40000/transcribe`

### Local deployment

To run your own instance locally you'll need an up-to-date version of Python 3.8 (we tested with version 3.8.5).

Copy the `.env.example` file over to a new file called `.env`:

```bash
cp .env.example .env
```

Edit `.env` file,<br/>
set the 3 first parameters with their respective values retrieved from your AWS account,<br/>
set the `PORT` value where transcriptions requests will be received.

Install dependencies once:
```bash
pip install --upgrade -r requirements.txt
```

Launch the reference connection service:
```bash
python server.py
```

Your server's public hostname and port will be used by your Vonage Voice API application as the address to where to submit the transcription request `https://<serverhostname>:<port>/transcribe`, e.g. `https://abcdef123456.ngrok.io/transcribe`

### Command Line Heroku deployment

Install [git](https://git-scm.com/downloads).

Install [Heroku command line](https://devcenter.heroku.com/categories/command-line) and login to your Heroku account.

Download this sample application code to a local folder, then go to that folder.

If you do not yet have a local git repository, create one:</br>
```bash
git init
git add .
git commit -am "initial"
```

Deploy this reference connection application to Heroku from the command line using the Heroku CLI:

```bash
heroku create myappname
```

On your Heroku dashboard where your reference connection application page is shown, click on `Settings` button,
add the following `Config Vars` and set them with their respective values retrieved from your AWS account:</br>
AWS_ACCESS_KEY_ID</br>
AWS_SECRET_ACCESS_KEY</br>
AWS_DEFAULT_REGION</br>

```bash
git push heroku master
```

On your Heroku dashboard where your reference connection application page is shown, click on `Open App` button, that URL will be the one to be used by your Vonage Voice API application as where to submit the HTTP POST, e.g. `https://myappname.herokuapp.com/transcribe`.

### 1-click Heroku deployment

Click the 'Deploy to Heroku' button at the top of this page, and follow the instructions to enter your Heroku application name and the 3 AWS parameter respective values retrieved from your AWS account.

Once deployed, on the Heroku dashboard where your reference connection application page is shown, click on `Open App` button, that URL will be the one to be used by your Vonage Voice API application as where to submit the HTTP POST, e.g. `https://myappname.herokuapp.com/transcribe`.

### Quick test

Quickly test your reference connection as follows:</br>
- Have a sample audio file ready with format PCM encoding, 16-bit sample size, 16 kHz sample frequency, e.g. _sampleaudio.wav_,</br>
- Have your deployed reference connection server URL, e.g. https://myapp.herokuapp.com/transcribe</br>
- Have the webhook call back URL to your client application, e.g. https://xxxx.ngrok.io/transcript</br>

Test the transcription using this curl command:</br>

```bash
curl -X POST "https://myappname.herokuapp.com/transcribe?webhook_url=https://xxxx.ngrok.io/transcript&entity=customer&id=abcd&language_code=en-US" -H "Content-Type:application/octet-stream" --data-binary @sampleaudio.wav
```
A JSON formatted response will be posted to the _webhook_url_ URL, including the transcript and all custom query parameters needed by your application logic from the original POST request.

## Usage capacity

This reference connection is a multi-threaded application that submits concurrent transcription requests to Amazon Transcribe in parallel.

Make sure your voice application and reference connection application do not submit more than the maximum allowed (default = 5) concurrent transcription requests on your Amazon Transcribe account.

You may see more information on that subject [here](https://docs.aws.amazon.com/transcribe/latest/dg/limits-guidelines.html).
