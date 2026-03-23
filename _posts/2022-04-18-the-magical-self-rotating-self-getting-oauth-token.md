---
layout: post
title: "The Magical Self-Rotating, Self-Getting OAuth Token"
date: 2022-04-18 00:00:00 
---

This class is one of my favorite simple patterns to implement when writing Python code against a web API. It’s a class which maintains the lifecycle of an OAuth bearer token for you, so that each time you use the token elsewhere in your code, you get a valid token back.

The class makes the following assumptions:

- We have client credentials for our OAuth app (no interactive login is required to get a fresh token).
- The token expires after some number of seconds.
- We can get a fresh token on each run with our client credentials, without having to persist a refresh token in memory, on disk or in a Vault. This code can be adapted to use a refresh token fairly easily.

The class was originally built against the Azure graph API using the `msal` library, but you can adapt it to use the SDK of any app. For our example, I’ve used a generic app with the `requests` library. 

Here’s the entire class:

```python
import requests
import datetime

class MagicToken:
	config = {
		"client_id": "123456",
		"client_secret": "shh",
		"scope": [ "read",  "write", "etc" ],
		"auth_endpoint": "https://api.my-great-app.com/v1.0/auth"
	}
	
	def __init__(self):
		self.__acquire_token()
	
	def __acquire_token(self):
		payload = {
			"client_id" : config['client_id'],
			"client_secret" : config['client_secret'],
	    "scope" : config['scope']
		}
	  url = config['auth_endpoint']
	  response = requests.post(url, data=payload)
		self.__access_token = result['access_token']
		self.__token_expires = datetime.datetime.now(datetime.timezone.utc) + datetime.timedelta(seconds=result['expiration'])
		return
	
	@property
	def token(self):
		if self.__token_expires > datetime.datetime.now(datetime.timezone.utc):
			return self.__access_token
		else:
			self.__acquire_token()
			return self.__access_token
```

Let’s break it down piece by piece.

```python
config = {
	"client_id": "123456",
	"client_secret": "shh",
	"scope": [ "read",  "write", "etc" ],
	"auth_endpoint": "https://api.my-great-app.com/v1.0/auth"
}
```

This is the basic configuration required to get our token. Client ID, client secret, and the authorization endpoint.

```python
def __init__(self):
	self.__acquire_token()

def __acquire_token(self):
	payload = {
		"client_id" : config['client_id'],
		"client_secret" : config['client_secret'],
    "scope" : config['scope']
	}
  url = config['auth_endpoint']
  response = requests.post(url, data=payload)
	self.__access_token = result['access_token']
	self.__token_expires = datetime.datetime.now(datetime.timezone.utc) + datetime.timedelta(seconds=result['expiration'])
	return
```

When we first start our app, we need to acquire a token. The `__acquire_token()` method starts with `__` in order to indicate that it shouldn’t be called from elsewhere in our code, it’s effectively a “private” that should only be called from within our class.

When we receive our OAuth response from the server, we want to capture the token itself, as well as the expiration time. Most servers give you back a number of seconds until expiration. If there’s a refresh token as well, you might need to use that to get your next token. We convert this number of seconds into a `datetime` object, in the future from the current time.

```python
@property
def token(self):
	if self.__token_expires > datetime.datetime.now(datetime.timezone.utc):
		return self.__access_token
	else:
		self.__acquire_token()
		return self.__access_token
```

This is where the magic comes in. We can use Python’s [@property decorator](https://www.programiz.com/python-programming/property) to handle the lifecycle of the token. Whenever your other code needs to use this token, you can do it like this:

```python
access_token = MagicToken()

def get_user(id):
  headers = {
		"Authorization": "Bearer " + access_token.token,
		"Content-Type": "application/json"
	}
  url = "https://api.my-great-app.com/v1.0/users/" + id"
  response = requests.get(url, headers=headers)
	return response
```

`access_token.token` is now a computed property. If you request it, the `token()` function will determine if the token is expired or not. If it’s still valid, it will return the token. If not, it will first acquire a new token, then return the new token.

We only need a getter for this property, since nothing outside of the class needs to be able to set the token.

And now, your token will magically manage itself, leaving you free to write good code against your favorite API.