---
author: adraffe
comments: true
date: 2013-12-02 18:25:01+00:00
layout: post
link: http://adamraffe.com/2013/12/02/nx-api-on-the-nexus-9000/
slug: nx-api-on-the-nexus-9000
title: NX-API on the Nexus 9000
wordpress_id: 666
categories:
- Nexus 9000
tags:
- Nexus
---

It's been a while since I posted - I've been spending a lot of time getting up to speed on the new Nexus 9000 switches and ACI. The full ACI fabric release is a few months away, but one of the interesting programmability aspects of the Nexus 9000 (running in "Standalone", or NX-OS mode) is the NX-API. So what is it exactly?<!-- more -->

The NX-API essentially provides a simple way to run CLI commands on one or more devices using a REST API. If you aren't familiar with the term, a REST API exposes resources and services using well known HTTP methods (such as GET, POST, etc). To access the NX-API, we send an HTTP GET or POST request to a well known URL, for example _http://<switch-ip>/ins_. We also need a way of passing actual data to the API - the NX-API expects to see input based on a simple XML string. Contained within this XML string are the commands you want to run.

**Getting Started
**

To get started with the NX-API, you first need to enable the feature on the switch. This is easy - it's just like enabling any other NX-OS feature:

    
    Nexus-9000(config)# feature nxapi


The best way to get up to speed with how the NX-API works is through the built in _sandbox_ environment - this is a web-based interface that allows you to test commands and the resulting output. You can get to the sandbox simply by browsing to the management IP address of the switch:

[![NX-API-Sandbox](http://adamraffe.files.wordpress.com/2013/11/nx-api-sandbox.png?w=550)](http://adamraffe.files.wordpress.com/2013/11/nx-api-sandbox.png)

On the left hand side of the sandbox screen, you can select the type of command you want to send (more on that in a moment), the output format (XML or JSON) and the actual command you want to send (in the above screen we are sending "show version"). On the right, we can see the output of the command returned to us in XML format. What if you prefer the output in JSON format? No problem, you just change the output format:

[![JSON-Output](http://adamraffe.files.wordpress.com/2013/11/json-output.png?w=550)](http://adamraffe.files.wordpress.com/2013/11/json-output.png)

**What input does the NX-API expect to see?**

NX-API expects to see input that conforms to a simple XML format, as follows:

    
    xml_string="<?xml version=\"1.0\" encoding=\"ISO-8859-1\"?> \
         <ins_api>                          \
         <version>0.1</version>             \
         <type>"cli_show"</type>            \
         <chunk>0</chunk>                   \
         <sid>session1</sid>                \
         <input>"show version"</input>      \
         <output_format>xml</output_format> \
         </ins_api>"


Let's have a closer look at some of these fields:

**_Version:_** Always 0.1 at the time of writing.

**_Type:_** There are several command types that can be executed through the NX-API. I'll explain these below.

**_Input:_** The actual NX-OS CLI command that you want to send to the device using the API.

**_Chunk:_** If this field is set to 1, the output should be 'chunked'. This might be necessary if the output of a command is long - in that case, chunks of the output can be sent back to the client as they are ready.

**Command Types**

As mentioned above, there are a number of command 'types' available for sending show and configuration commands to a device using the NX-API. Which type you need to use depends somewhat on the actual command you want to send:

_cli_show: _You should use the "cli_show" type when you want to send a show command that supports structured XML output. How do you know whether a command supports XML output? If you go to the CLI of the switch and run the command with the "| xml" option on the end, you'll be able to see whether that command returns XML output or not. What happens if you try and use the "cli_show" command type with a command that doesn't support XML? You will receive a message stating "structured output unsupported" from the API:

    
    <code id="responseMsg"><?xml version="1.0" encoding="UTF-8"?>
    <ins_api>
      <type>cli_show</type>
      <version>0.1</version>
      <sid>eoc</sid>
      <outputs>
        <output>
          <input>show clock</input>
          <msg>Structured output unsupported</msg>
          <code>501</code>
        </output>
      </outputs>
    </ins_api></code>


If that happens, you'll need to use the second command type - "cli_show_ascii".

**_cli_show_ascii:_** This command type returns output in ASCII format, with the entire output inside one <body> element. Any command on the switch (including ones that don't support structured XML output) should work with this command type, although it will be more difficult to parse compared to those commands that return XML.

**_cli_conf: _**Use this command type when you want to send configuration commands (as opposed to show commands) to the API.

**_bash:_ **You can use this command type if you want to send Bash shell commands to the device through the NX-API.

**Authenticating to the NX-API**

In order to use the NX-API, you must first authenticate to the device. This can be done using basic HTTP authentication - the client should send the username and password to the device using the HTTP authorisation header. The first time the user authenticates, the device will send back a session cookie (named _nxapi_auth_). This session cookie should be sent in subsequent requests to the NX-API (along with the credentials) to reduce the load on the authentication module on the switch.

Hopefully this gives you an overview of what the NX-API does. Although the design of the API has been kept intentionally simple, there are a large number of potential uses and it should prove a useful tool for managing the network environment. Thanks for reading.
