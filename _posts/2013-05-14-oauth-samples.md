---
layout: post
title: Sample authentication on Google Drive and SkyDrive - OAuth 2.0
tags: [python, oauth, linux]
description: Two little examples of oauth 2.0 for google drive and skydrive
last_updated: 14/05/2013
---

### Two little sample codes of OAuth using python and requests.

This little diagram is worth showing.

<pre style="background: white; border: none">
+--------+                               +---------------+
|        |--(A)- Authorization Request ->|   Resource    |
|        |                               |     Owner     |
|        |<-(B)-- Authorization Grant ---|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(C)-- Authorization Grant -->| Authorization |
| Client |                               |     Server    |
|        |<-(D)----- Access Token -------|               |
|        |                               +---------------+
|        |
|        |                               +---------------+
|        |--(E)----- Access Token ------>|    Resource   |
|        |                               |     Server    |
|        |<-(F)--- Protected Resource ---|               |
+--------+                               +---------------+
</pre>


SkyDrive:

	#!/bin/python
	#
	# This application will get the access_token for the Microsoft SkyDrive
	# service based in the application id and secret

	import requests
	import json

	CLIENT_SECRET = ""
	CLIENTE_ID = ""

	def get_acess_token( client_id, client_secret ):
	print "get_acess_token"

	#
	# get the acess code
	#
	url = "https://login.live.com/oauth22_authorize.srf"
	payload = {
		"client_id": client_id,
		"scope": "wl.signin",
		"response_type": "code",
		}

	r = requests.get( url, params=payload )

	# put this url into your browser end hit go
	print r.url

	# enter the code returned from the url redirect
	code = raw_input()

	#
	# get the acess token
	#
	url = "https://login.live.com/oauth20_token.srf"
	payload = {
		"client_id": client_id,
		"client_secret": client_secret,
		"grant_type": "authorization_code",
		"code": code,
		}

	r = requests.get( url, params=payload )
	response = r.json()
	return response["access_token"]

	if __name__ == '__main__':
	access_token = get_acess_token( CLIENTE_ID, CLIENT_SECRET )



Google Drive:

	#!/bin/python
	#
	# This application will get the access_token for the Google Drive
	# service based in the application id and secret

	import requests
	import json

	CLIENT_SECRET = ""
	CLIENTE_ID = ""

	def get_acess_token( client_id, client_secret ):
	    print "get_acess_token"

	    #
	    # get the acess code
	    #
	    url = "https://accounts.google.com/o/oauth2/auth"
	    payload = {
		    "client_id": client_id, "scope": "email",
		    "response_type": "code",
		    "redirect_uri":"https://example.com/oauth2callback",
		    }

	    r = requests.get( url, params=payload )

	    # put this url into your browser end hit go
	    print r.url

	    # enter the code returned from the url redirect
	    code = raw_input()

	    #
	    # get the acess token
	    #
	    url = "https://accounts.google.com/o/oauth2/token"
	    payload = {
		    "code": code, "client_id": client_id,
		    "client_secret": client_secret,
		    "redirect_uri":"https://example.com/oauth2callback",
		    "grant_type": "authorization_code",
		    }

	    r = requests.post( url, data=payload )
	    response = r.json()
	    print response['id_token']
	    print response['access_token']

	if __name__ == '__main__':
	    access_token = get_acess_token( CLIENTE_ID, CLIENT_SECRET )



