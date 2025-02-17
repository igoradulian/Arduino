ESP8266WiFi library
===================

ESP8266 is all about Wi-Fi. If you are eager to connect your new ESP8266 module to a Wi-Fi network to start sending and receiving data, this is a good place to start. If you are looking for more in depth details of how to program specific Wi-Fi networking functionality, you are also in the right place.


Introduction
------------

The `Wi-Fi library for ESP8266 <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ has been developed based on `ESP8266 SDK <https://github.com/espressif/ESP8266_NONOS_SDK>`__, using the naming conventions and overall functionality philosophy of the `Arduino WiFi library <https://www.arduino.cc/en/Reference/WiFi>`__. Over time, the wealth of Wi-Fi features ported from ESP8266 SDK to `esp8266 /
Arduino <https://github.com/esp8266/Arduino>`__ outgrew `Arduino WiFi library <https://www.arduino.cc/en/Reference/WiFi>`__ and it became apparent that we would need to provide separate documentation on what is new and extra.

This documentation will walk you through several classes, methods and properties of the `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ library. If you are new to C++ and Arduino, don't worry. We will start from general concepts and then move to detailed description of members of each particular class including usage examples.

The scope of functionality offered by the `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ library is quite extensive and therefore this description has been broken up into separate documents marked with :arrow\_right:.

Quick Start
~~~~~~~~~~~

Hopefully, you are already familiar how to load the `Blink.ino <https://github.com/esp8266/Arduino/blob/master/libraries/esp8266/examples/Blink/Blink.ino>`__ sketch to an ESP8266 module and get the LED blinking. If not, please use `this tutorial <https://learn.adafruit.com/adafruit-huzzah-esp8266-breakout/using-arduino-ide>`__ by Adafruit or `another great tutorial <https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/introduction>`__ developed by Sparkfun.

To hook up the ESP module to Wi-Fi (like hooking up a mobile phone to a hot spot), you need only a couple of lines of code:

.. code:: cpp

    #include <ESP8266WiFi.h>

    void setup()
    {
      Serial.begin(115200);
      Serial.println();

      WiFi.begin("network-name", "pass-to-network");
      
      Serial.print("Connecting");
      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }
      Serial.println();
      
      Serial.print("Connected, IP address: ");
      Serial.println(WiFi.localIP());
    }

    void loop() {}

In the line ``WiFi.begin("network-name", "pass-to-network")`` replace ``network-name`` and ``pass-to-network`` with the name and password of the Wi-Fi network you would like to connect to. Then, upload this sketch to ESP module and open the serial monitor. You should see something like:

.. figure:: pictures/wifi-simple-connect-terminal.png
   :alt: Connection log on Arduino IDE's Serial Monitor


How does it work? In the first line of the sketch, ``#include <ESP8266WiFi.h>`` we are including the `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ library. This library provides ESP8266 specific Wi-Fi routines that we are calling to connect to the network.

The actual connection to Wi-Fi is initialized by calling:

.. code:: cpp

    WiFi.begin("network-name", "pass-to-network");

The connection process can take couple of seconds and we are checking for whether this has completed in the following loop:

.. code:: cpp

      while (WiFi.status() != WL_CONNECTED)
      {
        delay(500);
        Serial.print(".");
      }

The ``while()`` loop will keep looping as long as ``WiFi.status()`` is other than ``WL_CONNECTED``. The loop will exit only if the status changes to ``WL_CONNECTED``.

The last line will then print out the IP address assigned to the ESP module by `DHCP <https://whatismyipaddress.com/dhcp>`__:

.. code:: cpp

    Serial.println(WiFi.localIP());

If you don't see the last line but just more and more dots ``.........``, then likely name or password to the Wi-Fi network is entered incorrectly in the sketch. Verify the name and password by connecting from scratch to this Wi-Fi network with a PC or a mobile phone.

*Note:* if connection is established, and then lost for some reason, ESP will automatically reconnect to the last used access point once it is again back on-line. This will be done automatically by Wi-Fi library, without any user intervention.

That's all you need to connect ESP8266 to Wi-Fi. In the following chapters we will explain what cool things can be done by the ESP once it's connected.

Who is Who
~~~~~~~~~~

Devices that connect to Wi-Fi networks are called stations (STA). Connection to Wi-Fi is provided by an access point (AP), that acts as a hub for one or more stations. The access point on the other end is connected to a wired network. An access point is usually integrated with a router to provide access from a Wi-Fi network to the internet. Each access point is recognized by a SSID (**S**\ ervice **S**\ et **ID**\ entifier), that essentially is the name of network you select when connecting a device (station) to the Wi-Fi.

ESP8266 modules can operate as a station, so we can connect it to the Wi-Fi network. It can also operate as a soft access point (soft-AP), to establish its own Wi-Fi network. When the ESP8266 module is operating as a soft access point, we can connect other stations to the ESP module. ESP8266 is also able to operate as both a station and a soft access point mode. This provides the possibility of building e.g. `mesh networks <https://en.wikipedia.org/wiki/Mesh_networking>`__.

.. figure:: pictures/esp8266-station-soft-access-point.png
   :alt: ESP8266 operating in the Station + Soft Access Point mode

The `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ library provides a wide collection of C++
`methods <https://en.wikipedia.org/wiki/Method_(computer_programming)>`__ (functions) and `properties <https://en.wikipedia.org/wiki/Property_(programming)>`__ to configure and operate an ESP8266 module in station and / or soft access point mode. They are described in the following chapters.

Class Description
-----------------

The `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ library is broken up into several classes. In most of cases, when writing the code, the user is not concerned with this classification. We are using it to break up description of this library into more manageable pieces.

.. figure:: pictures/doxygen-class-index.png
   :alt: Index of classes of ESP8266WiFi library

Chapters below describe all function calls (`methods <https://en.wikipedia.org/wiki/Method_(computer_programming)>`__ and `properties <https://en.wikipedia.org/wiki/Property_(programming)>`__ in C++ terms) listed in particular classes of `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__. The description is illustrated with application examples and code snippets to show how to use functions in practice. This information is broken up into the following documents. 

Station
~~~~~~~

Station (STA) mode is used to get the ESP module connected to a Wi-Fi network established by an access point.

.. figure:: pictures/esp8266-station.png
   :alt: ESP8266 operating in the Station mode

Station class has several features to facilitate the management of a Wi-Fi connection. In case the connection is lost, the ESP8266 will automatically reconnect to the last used access point, once it is available again. The same happens on module reboot. This is possible since ESP saves the credentials to the last used access point in flash (non-volatile) memory. Using the saved data ESP will also reconnect if sketch has been changed but code does not alter the Wi-Fi mode or credentials.

`Station Class documentation <station-class.rst>`__

Check out separate section with `examples <station-examples.rst>`__.

Soft Access Point
~~~~~~~~~~~~~~~~~

An `access point (AP) <https://en.wikipedia.org/wiki/Wireless_access_point>`__ is a device that provides access to a Wi-Fi network to other devices (stations) and connects them to a wired network. The ESP8266 can provide similar functionality, except it does not have interface to a wired network. Such mode of operation is called soft access point (soft-AP). The maximum number of stations that can simultaneously be connected to the soft-AP can be set `from 0 to 8 <https://bbs.espressif.com/viewtopic.php?f=46&t=481&p=1832&hilit=max_connection#p1832>`__, but defaults to 4.

.. figure:: pictures/esp8266-soft-access-point.png
   :alt: ESP8266 operating in the Soft Access Point mode

The soft-AP mode is often used and an intermediate step before connecting ESP to a Wi-Fi in a station mode. This is when SSID and password to such network is not known upfront. ESP first boots in soft-AP mode, so we can connect to it using a laptop or a mobile phone. Then we are able to provide credentials to the target network. Then, the ESP is switched to the station mode and can connect to the target Wi-Fi.

Another handy application of soft-AP mode is to set up `mesh networks <https://en.wikipedia.org/wiki/Mesh_networking>`__. The ESP can operate in both soft-AP and Station mode so it can act as a node of a mesh network.

`Soft Access Point Class documentation <soft-access-point-class.rst>`__

Check out the separate section with `examples <soft-access-point-examples.rst>`__.

Scan
~~~~

To connect a mobile phone to a hot spot, you typically open Wi-Fi settings app, list available networks and pick the hot spot you need. Then enter a password (or not) and you are in. You can do the same with the ESP. Functionality of scanning for, and listing of available networks in range is implemented by the Scan Class.

`Scan Class documentation <scan-class.rst>`__

Check out the separate section with `examples <scan-examples.rst>`__.

Client
~~~~~~

The Client class creates `clients <https://en.wikipedia.org/wiki/Client_(computing)>`__ that can access services provided by `servers <https://en.wikipedia.org/wiki/Server_(computing)>`__ in order to send, receive and process data.

.. figure:: pictures/esp8266-client.png
   :alt: ESP8266 operating as the Client

Check out the separate section with `list of functions <client-class.rst>`__

WiFi Multi
~~~~~~~~~~

`ESP8266WiFiMulti.h` can be used to connect to a WiFi network with strongest WiFi signal (RSSI). This requires registering one or more access points with SSID and password. It automatically switches to another WiFi network when the WiFi connection is lost.

Example:

.. code:: cpp
    #include <ESP8266WiFiMulti.h>

    ESP8266WiFiMulti wifiMulti;

    // WiFi connect timeout per AP. Increase when connecting takes longer.
    const uint32_t connectTimeoutMs = 5000;

    void setup()
    {
      // Set in station mode
      WiFi.mode(WIFI_STA);

      // Register multi WiFi networks
      wifiMulti.addAP("ssid_from_AP_1", "your_password_for_AP_1");
      wifiMulti.addAP("ssid_from_AP_2", "your_password_for_AP_2");
      wifiMulti.addAP("ssid_from_AP_3", "your_password_for_AP_3");
    }

    void loop()
    {
      // Maintain WiFi connection
      if (wifiMulti.run(connectTimeoutMs) == WL_CONNECTED) {
          ...
      }
    }

BearSSL Client Secure and Server Secure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`BearSSL::WiFiClientSecure` and `BearSSL::WiFiServerSecure` are extensions of the standard `Client <#client>`__ and `Server <#server>`__ classes where connection and data exchange with servers and clients using `secure protocol <https://en.wikipedia.org/wiki/Transport_Layer_Security>`__.  It supports `TLS 1.2 <https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.2>`__ using a wide variety of modern ciphers, hashes, and key types.

.. figure:: pictures/esp8266-client-secure.png
   :alt: ESP8266 operating as the Client Secure

Secure clients and servers require significant amounts of additional memory and processing to enable their cryptographic algorithms.  In general, only a single secure client or server connection at a time can be processed given the little RAM present on the ESP8266, but there are methods of reducing this RAM requirement detailed in the relevant sections.

`BearSSL::WiFiClientSecure <bearssl-client-secure-class.rst>`__ contains more information on using and configuring TLS connections.

`BearSSL::WiFiServerSecure <bearssl-server-secure-class.rst>`__ discusses the TLS server mode available.  Please read and understand the `BearSSL::WiFiClientSecure <bearssl-client-secure-class.rst>`__ first as the server uses most of the same concepts.

Check out the separate section with `examples <client-secure-examples.rst>`__ .


Server
~~~~~~

The Server Class creates `servers <https://en.wikipedia.org/wiki/Server_(computing)>`__ that provide functionality to other programs or devices, called `clients <https://en.wikipedia.org/wiki/Client_(computing)>`__.

.. figure:: pictures/esp8266-server.png
   :alt: ESP8266 operating as the Server

Clients connect to sever to send and receive data and access provided functionality.

Check out separate section with `examples <server-examples.rst>`__ / `list of functions <server-class.rst>`__.

UDP
~~~

The UDP Class enables the `User Datagram Protocol (UDP) <https://en.wikipedia.org/wiki/User_Datagram_Protocol>`__ messages to be sent and received. The UDP uses a simple "fire and forget" transmission model with no guarantee of delivery, ordering, or duplicate protection. UDP provides checksums for data integrity, and port numbers for addressing different functions at the source and destination of the datagram.

Check out separate section with `examples <udp-examples.rst>`__ / `list of functions <udp-class.rst>`__.

Generic
~~~~~~~

There are several functions offered by ESP8266's `SDK <https://bbs.espressif.com/viewtopic.php?f=51&t=1023>`__ and not present in `Arduino WiFi library <https://www.arduino.cc/en/Reference/WiFi>`__. If such function does not fit into one of classes discussed above, it will likely be in Generic Class. Among them is handler to manage Wi-Fi events like connection, disconnection or obtaining an IP, Wi-Fi mode changes, functions to manage module sleep mode, hostname to an IP address resolution, etc.

Check out separate section with `examples <generic-examples.rst>`__ / `list of functions <generic-class.rst>`__.

Diagnostics
-----------

There are several techniques available to diagnose and troubleshoot issues with getting connected to Wi-Fi and keeping connection alive.

Check Return Codes
~~~~~~~~~~~~~~~~~~

Almost each function described in chapters above returns some diagnostic information.

Such diagnostic may be provided as a simple ``boolean`` type ``true`` or ``false`` to indicate operation result. You may check this result as described in examples, for instance:

.. code:: cpp

    Serial.printf("Wi-Fi mode set to WIFI_STA %s\n", WiFi.mode(WIFI_STA) ? "" : "Failed!");

Some functions provide more than just a binary status information. A good example is ``WiFi.status()``.

.. code:: cpp

    Serial.printf("Connection status: %d\n", WiFi.status());

This function returns following codes to describe what is going on with Wi-Fi connection: 

* 0 : ``WL_IDLE_STATUS`` when Wi-Fi is in process of changing between statuses 
* 1 : ``WL_NO_SSID_AVAIL``\ in case configured SSID cannot be reached 
* 3 : ``WL_CONNECTED`` after successful connection is established 
* 4 : ``WL_CONNECT_FAILED`` if connection failed 
* 6 : ``WL_CONNECT_WRONG_PASSWORD`` if password is incorrect 
* 7 : ``WL_DISCONNECTED`` if module is not configured in station mode

It is a good practice to display and check information returned by functions. Application development and troubleshooting will be easier with that.

Use printDiag
~~~~~~~~~~~~~

There is a specific function available to print out key Wi-Fi diagnostic information:

.. code:: cpp

    WiFi.printDiag(Serial);

A sample output of this function looks as follows:

::

    Mode: STA+AP
    PHY mode: N
    Channel: 11
    AP id: 0
    Status: 5
    Auto connect: 1
    SSID (10): sensor-net
    Passphrase (12): 123!$#0&*esP
    BSSID set: 0

Use this function to provide snapshot of Wi-Fi status in these parts of application code, that you suspect may be failing.

Enable Wi-Fi Diagnostic
~~~~~~~~~~~~~~~~~~~~~~~

By default the diagnostic output from Wi-Fi libraries is disabled when you call ``Serial.begin``. To enable debug output again, call ``Serial.setDebugOutput(true)``. To redirect debug output to ``Serial1`` instead, call ``Serial1.setDebugOutput(true)``. For additional details regarding diagnostics using serial ports please refer to `the documentation <../reference.rst>`__.

Below is an example of output for sample sketch discussed in `Quick Start <#quick-start>`__ above with ``Serial.setDebugOutput(true)``:

::

    Connectingscandone
    state: 0 -> 2 (b0)
    state: 2 -> 3 (0)
    state: 3 -> 5 (10)
    add 0
    aid 1
    cnt 

    connected with sensor-net, channel 6
    dhcp client start...
    chg_B1:-40
    ...ip:192.168.1.10,mask:255.255.255.0,gw:192.168.1.9
    .
    Connected, IP address: 192.168.1.10

The same sketch without ``Serial.setDebugOutput(true)`` will print out only the following:

::

    Connecting....
    Connected, IP address: 192.168.1.10

Enable Debugging in IDE
~~~~~~~~~~~~~~~~~~~~~~~

Arduino IDE provides convenient method to `enable debugging <../Troubleshooting/debugging.rst>`__ for specific libraries.

What's Inside?
--------------

If you like to analyze in detail what is inside of the ESP8266WiFi library, go directly to the `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi/src>`__ folder of esp8266 / Arduino repository on the GitHub.

To make the analysis easier, rather than looking into individual header or source files, use one of free tools to automatically generate documentation. The class index in chapter `Class Description`_ above has been prepared in no time using great `Doxygen <https://www.doxygen.nl/>`__, that is the de facto standard tool for generating documentation from annotated C++ sources.

.. figure:: pictures/doxygen-esp8266wifi-documentation.png
   :alt: Example of documentation prepared by Doxygen

The tool crawls through all header and source files collecting information from formatted comment blocks. If developer of particular class annotated the code, you will see it like in examples below.

.. figure:: pictures/doxygen-example-station-begin.png
   :alt: Example of documentation for station begin method by Doxygen

.. figure:: pictures/doxygen-example-station-hostname.png
   :alt: Example of documentation for station hostname property by Doxygen

If code is not annotated, you will still see the function prototype including types of arguments, and can use provided links to jump straight to the source code to check it out on your own. Doxygen provides really excellent navigation between members of library.

.. figure:: pictures/doxygen-example-udp-begin.png
   :alt: Example of documentation for UDP begin method (not annotated in code)by Doxygen

Several classes of `ESP8266WiFi <https://github.com/esp8266/Arduino/tree/master/libraries/ESP8266WiFi>`__ are not annotated. When preparing this document, `Doxygen <https://www.doxygen.nl/>`__ has been tremendous help to quickly navigate through almost 30 files that make this library.
