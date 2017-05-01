voice_lirc
========

``voice_lirc`` is an Amazon Echo / Alexa voice command skill forked from lirc_web. It is a [nodejs](http://nodejs.org) app derived from [lirc_web](https://github.com/alexbain/lirc_web) that creates a web interface & JSON API for the [LIRC](http://lirc.org) project. It uses [lirc_node](https://github.com/alexbain/lirc_node) to handle communication between LIRC and nodejs.

This project allows you to control LIRC from your Amazon Echo so you can issue infrared remote control commands via voice command.

## Installation

You'll need to have [LIRC](http://lirc.org) installed and configured on your machine to use ``voice_lirc``. In addition, you'll need to install [nodejs](http://nodejs.org). Once you have LIRC and nodejs installed and configured, you'll be able to install ``voice_lirc``:

    npm install -g voice_lirc
    voice_lirc

Note that you may need to run the `npm install` command with `sudo`.

### Viewing

Verify the web interface works by opening ``http://SERVER:3000/`` in a web browser.

If you want to have `lirc_web`  available via port 80 and start on boot, there are example NGINX and Upstart configuration files included in the ``example_configs/`` directory.

## Configuration

As of v0.0.8, ``lirc_web`` supports customization through a configuration file.

You may place this configuration file in one of two locations and `lirc_web` will detect it:

1. Place a file named `.lirc_web_config.json` in the home directory of the user running `lirc_web` (global installation)
2. Place a file named `config.json` in the root of the `lirc_web` project directory (local / development installation)

These are the available configuration options:

1. ``repeaters`` - buttons that repeatedly send their commands while pressed. A common example are the volume buttons on most remote controls. While you hold the volume buttons down, the remote will repeatedly send the volume command to your device.
2. ``macros`` - a collection of commands that should be executed one after another. This allows you to automate actions like "Play Xbox 360" or "Listen to music via AirPlay". Each step in a macro is described in the format ``[ "REMOTE", "COMMAND" ]``, where ``REMOTE`` and ``COMMAND`` are defined by what you have programmed into LIRC. You can add delays between steps of macros in the format of ``[ "delay", 500 ]``. Note that the delay is measured in milliseconds so 1000 milliseconds = 1 second.  You can also add a repeater macro with a delay by using the format ``[ "REMOTE", ["COMMAND", delay]]`` in place of a normal``COMMAND`` (Refer to Xbox Off command below). 
3. ``commandLabels`` - a way to rename commands that LIRC understands (``KEY_POWER``, ``KEY_VOLUMEUP``) with labels that humans prefer (``Power``, ``Volume Up``).
4. ``remoteLabels`` - a way to rename the remotes that LIRC understands (``XBOX360``) with labels that humans prefer (``Xbox 360``).
5. ``blacklists`` - a way to hide unused commands from your remotes.
6. ``server`` - server configuration settings (ports, [SSL](http://serverfault.com/a/366374)).
7. ``socket`` - to specify the lircd socket for irsend.


#### Example config.json:


    {
      "server" : {
        "port" : 3000,
        "ssl" : false,
        "ssl_cert" : "/home/pi/lirc_web/server.cert",
        "ssl_key" : "/home/pi/lirc_web/server.key",
        "ssl_port" : 3001
      },
      "repeaters": {
        "SonyTV": {
          "VolumeUp": true,
          "VolumeDown": true
        }
      },
      "macros": {
        "Play Xbox 360": [
          [ "SonyTV", "Power" ],
          [ "delay", 500 ],
          [ "SonyTV", "Xbox360" ],
          [ "Yamaha", "Power" ],
          [ "delay", 250 ],
          [ "Yamaha", "Xbox360" ],
          [ "Xbox360", "Power" ]
        ],
        "Listen to Music": [
          [ "Yamaha", "Power" ],
          [ "delay", 500 ],
          [ "Yamaha", "AirPlay" ]
        ],
        "Xbox Off": [
          [ "XboxOne", [ "Power", "1600" ] ],
          [ "delay", "1010" ],
          [ "XboxOne", "Up" ],
          [ "XboxOne", "Select" ]
        ],
      },
      "commandLabels": {
        "Yamaha": {
          "Power": "Power",
          "Xbox360": "Xbox 360",
          "VolumeUp": "Volume Up",
          "VolumeDown": "Volume Down"
        }
      },
      "remoteLabels": {
         "Xbox360": "Xbox 360"
      },
      "blacklists": {
         "Yamaha": [
           "AUX2",
           "AUX3"
         ]
      },
      "socket": "/run/lirc/lircd1"
    }

Please see the `example_configs/` directory.

## Using lirc_web macros with Amazon Alexa

To keep things simple, we're going to use the macros feature to invoke sequences of IR commands using Alexa, so you'll want to set those up properly with easily comprehensible names.

## Integrating with Amazon Alexa
* Remember to first set up LIRC on the box so that it will work with your Infrared Remote Controllable devices!!
```
node voice_lirc
```
* If successfully running, you will see:

```
Open Source Universal Remote UI + API has started on port 3000 (http).
```

* Sign up or log in to the [AWS Console](http://console.aws.amazon.com) and choose Lambda from the Services > All Services menu. 
* Create a Lambda function, skip the intro wizard, name this function 'LIRC' and paste the contents of 'lambda redirect.js'.  A description isn't required.
* Seperate from AWS, log in to your [Amazon Apps & services Developer Account](https://developer.amazon.com/appsandservices) and under Apps & Services, choose Alexa.
* You will need to add a new Alexa Skill.  The skill has an associated name (lirc), invocation name (tv), intent schema & sample utterances included here that must be specified in order for the functions to work.
* For each skill, specify the ARN of the lambda functions created on the AWS console.

### Notes ###
* Ensure that the port you specifed (default: 3000) is open by your ISP and that it's forwarded to the computer running voice_lirc.

## License

(The MIT License)

Copyright (c) 2013-2017 Alex Bain &lt;alex@alexba.in&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.
