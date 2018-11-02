# Device Name Example

This example shows how to retrieve get the device name and use it in a subscription handler.

Here's the complete flow:

![flow](images/flow.png)

## Save the device list

The top part of the flow retrieves the device list at startup and every 15 minutes. This is much more efficient and easier to implement than looking up the device name every time.

Start with an inject node:

![inject](images/inject.png)

Next is a Particle API node to list the devices in the account:

![list devices](images/list-devices-api.png)

And a function to save the results:

![save](images/save-device-list.png)

Here's the code:

```
var deviceNames = {};

// Payload is an array of devices for this account
for(var ii = 0; ii < msg.payload.length; ii++) {
    deviceNames[msg.payload[ii].id] = msg.payload[ii].name;    
}
    
// Save for future use by this node    
flow.set('deviceNames', deviceNames);

return msg;   
```

You should see something like this in the debug log after deploying your flow.

![debug](images/debug-1.png)

You can expand it to see all of the things in the list devices message.

## Subscribe

The second part subscribes to an event using the Particle subscribe node:

![subscribe](images/subscribe.png)

We prepare the message using a function node:

![subscribe](images/prepare-message.png)

Here's the code:

```
var deviceNames = flow.get('deviceNames', deviceNames);

msg.payload = 'got event ' + msg.event + 
    ' with payload ' + msg.payload + 
    ' at ' + msg.published_at + 
    ' from ' + deviceNames[msg.device];


return msg;
```

The first line gets the saved device names that we saved from the top of the flow.

The rest of it generates a message from it.

That's just connected to a debug node for this test.

If you generate the event (see firmware below) you should see something like this in the debug log.

![debug](images/debug-2.png)

## Firmware

This is the Photon test firmware I used. When you click the SETUP/MODE button, it generates an event that should display in the rules engine.

```
#include "Particle.h"

void buttonClicked(system_event_t event, int param);

int timesSent = 1;
bool clicked = false;

void setup() {
	Serial.begin();
	System.on(button_click, buttonClicked);
}

void loop() {
	if (clicked) {
		clicked = false;
		char buf[256];
		snprintf(buf, sizeof(buf), "hello %d", timesSent++);

		Particle.publish("rulesTest1", buf, PRIVATE);
	}
}


void buttonClicked(system_event_t event, int param) {
	clicked = true;
}

```

