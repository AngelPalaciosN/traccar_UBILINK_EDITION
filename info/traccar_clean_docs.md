# TRACCAR SELECTED DOCUMENTATION


================================================================================
# Source URL: https://www.traccar.org/quick-start/
================================================================================

# Quick Start

This guide provides the initial steps to set up Traccar, login and connect your first device.

1 Download and Install

Start here if you plan to host your own instance of Traccar. If you're using one of our [subscriptions](/pricing/), [demo servers](/demo-server/) or already have Traccar installed and ready, skip to [Point 2](#point-2).

You can download an installer package for your operating system from the [downloads](/download/) page.

For the installation instructions check the corresponding section on our [main documentation page](/documentation/).

2 Open the App

Open the server URL in a browser or use the [Traccar Manager](/manager/) mobile app.

If you host your own server, the URL is <http://localhost:8082> for a local connection or **http://your.server.ip.address:8082** for an external access.

On some older version of Traccar, the default administrator account is created automatically with login **admin** and password **admin**. On newer versions there are no default users, so you have to register. First registered user automatically becomes an admin. If you're using a subscription, you will be provided with the URL and login details.

3 Register a Device

Click the add button (the **plus** icon) in the devices toolbar to register a new device.

You have to fill out the name and identifier fields. Name can be anything. The identifier has to match the unique id your device is reporting to the server. For most devices you should use IMEI or the serial number as the unique identifier.

If you don't know your device identifier, you can configure the device to report the data first and look for the id in the log. Check the [troubleshooting guide](/troubleshooting/) for more details.

4 Configure the Device

Configure your device to start reporting the location data to your server. For more details on how to configure your specific model check your device manual or contact your device vendor support.

If you host your own server, your system must have a public IP address. Not all internet providers assign public IPs. Your current address appears to be **181.143.26.20**, but it might not be your dedicated address if you are behind a NAT. If you don't have a public IP, you can use a [VPS server](/install-digitalocean/) or one of our [subscriptions](/pricing/) instead of hosting locally.

To select the correct port number, find your device in the [list of supported devices](/devices/). Port column indicates the default port number in Traccar for your device.

================================================================================
# Source URL: https://www.traccar.org/identify-protocol/
================================================================================

# Protocol Identification

As the first step, try finding your device in the list of [supported devices](/devices/). The last column contains default port number for the device. If you have a device made in China, make sure to also check the information about [clones](/clones/).

If a given port does not work for your device or you cannot find your device in the list of supported devices, try collecting some samples using the following steps:

* Configure your device to send data to the port 5001
* Wait for the device to send a few messages to the server
* Find HEX message from the device in the [log file](/troubleshooting/#point-1)

Once you have some samples and you are sure those samples are from your device, you can try matching them with common patterns below or compare to test cases we have for various protocols:

* [Test cases for protocols](https://github.com/traccar/traccar/tree/master/src/test/java/org/traccar/protocol)

If protocol is text-based, you have to [convert HEX to text](/hex-decoder/) first.

## Common protocol patterns

If your device is using GT06 protocol (correct port 5023), HEX samples will start with `7878` or `7979`, like this:

```
78780D01086471700328358100093F040D0A
```

If your device is using JT808 protocol (correct port 5015), HEX samples will start and end with `7e`. Note that there are some other protocols that use `7e`, so it's not a guaranteed match. Example:

```
7E01000021013345678906000F002C012F373031313142534A2D4D3742203030303030303001D4C1423838383838B47E
```

If your device is using GPS103 protocol (correct port 5001), decoded message text would start with `imei:` and end with a semi-colon (`;`). Example:

```
imei:864035050002451,tracker,201223064947,,F,064947,A,1935.70640,N,09859.94436,W,0.025,;
```

If your device is using H02 protocol (correct port 5013), decoded message text would start with `*HQ,` and end with a number sign (`#`). Note that some H02-based devices send a mix of text and binary messages. Text message example:

```
*HQ,4210209006,V1,201844,A,2608.9437,N,08016.2521,W,000.80,000,150317,FFFFF9FF,310,260,0,0,6#
```

If your device is using Watch protocol (correct port 5093), decoded message text would start with opening square bracket followed by 2 upper-case characters, followed by device id surrounded by star signs, like `[3G*8308373902*`, and end with a closing square bracket. Example:

```
[SG*9051004074*0058*AL,120117,145602,V,40.058413,N,76.336618,W,11.519,188,99,00,01,80,0,50,00000000,0,1,0,0,,10]
```

================================================================================
# Source URL: https://www.traccar.org/user-management/
================================================================================

# User Management

Traccar user management model supports use cases from simple individual user accounts to large organizations with multiple managers, administrators and regular users.

![](/images/documentation/user-management/user-1.png)

## Main user roles

**Admin** - superuser with full unlimited access to the whole Traccar server.

**Manager** - user with extended capabilities allowing them to manage a subset of users and register new ones.

**User** - ordinary user that can manipulate their own assets and add new ones.

## Limited user roles

**Readonly user** - a user that cannot add/edit/remove anything in the system. They can only monitor their assigned objects. Can be useful for public/embedded access.

**Device readonly user** - ordinary user with a restriction on device manipulation. Other settings can be edited without limits.

## User limits

**Device limit** - the limit on number of devices the user can have. The user cannot add devices more than his **device limit**.

If **device limit** is set to **-1**, it means that the user has no limit on the number of devices.

If **device limit** is set to **0**, it means that user cannot add any devices, but they can edit or remove existing ones.

**User limit** - the limit on number of users that manager can have. The manager cannot add users more than his **user limit**.

If **user limit** is set to **-1**, it means that the manager has no limit.

If **user limit** is set to **0**, it means that the user is not a manager. The difference between **manager** and **regular user** is in their **user limit** value. A manager has the **user limit** not equals to **0**.

Only **administrator** can change these fields.

User created by a **manager** will always have **0** value for both limits.

Self registered users will have **device limit** set to **-1** or value from server config **users.defaultDeviceLimit**.

**Expiration time** - the time after which the user cannot log in to the system.

**Disabled** - user cannot log in if they are **disabled**.

**User** or **manager** cannot edit these two fields in their own accounts.

**Manager** can edit these fields for the users they have access to, with one restriction. If a **manager** has an **expiration time**, they cannot set other users' **expiration time** later than their own.

Another important restriction is that only an **administrator** can unlink devices from themselves.

## Embedded or public view

Start with creating user:

* Register a new user
* Make the user readonly
* Generate a token for the user

Now the user can login using the token in URL. For example: <http://demo.traccar.org/?token=ABCDEFGHIJKLMN>.

## Traccar as a service

Service administrator can create one **manager** user for every client, set **user limit** and **device limit** according to the subscription. For example, 5 users and 50 devices. They can also set **expiration time** to limit subscription period.

In the example above, the client can add 5 users and 50 devices, link devices to users, create and link groups, geofences and everything else within the specified limits.

Manager's **device limit** will work for the whole client because client users can add new devices if only administrator explicitly allowed it.

================================================================================
# Source URL: https://www.traccar.org/events/
================================================================================

# Events and Notifications

Traccar supports a number of different **event** types. Most events are generated on the server side based on the data reported by devices. There is a special **alarm** event type, which represents events directly reported by devices.

Each user can configure notifications for certain types of events. Notifications can be sent using one or multiple channels, like email, push notifications, SMS and other. A notification can be enabled for all devices or linked to individual devices or groups of devices. Events are generated and can be accessed using events report even if no notifications are configured for a certain device and type of event. Notifications can be configured in the settings menu:

![](/images/documentation/notifications/settings.png)

By default Traccar has web and email notification channels enabled. To add or remove channels you need to edit the configuration file. See [notifications configuration](/notifications/) page for more details.

It is also possible to limit notification time using a [calendar](/calendars/).

## Event Types

| Type | Description |
| --- | --- |
| **Command Result** | A result of a command execution on the device. Support for results varies by protocol. Result details are stored in the **result** attribute. |
| **Status Online** | Device is connected to the server. This event type is not associated with a position. |
| **Status Unknown** | Device is connected, but has not reported any data for a period of time. It can also indicate that the device has not closed the connection gracefully. |
| **Status Offline** | Device has disconnected from the server. |
| **Device Inactive** | Device has not reported any location data for a very long period of time. This event needs to be explicitly enabled using **deviceInactivityStart** and **deviceInactivityPeriod** attributes. |
| **Device Moving** | Device is moving. For more details on the logic see the [trips and stops](/trips-stops/) documentation. |
| **Device Stopped** | Device has stopped. For more details on the logic see the [trips and stops](/trips-stops/) documentation. |
| **Speed Limit Exceeded** | This event is generated when a device exceeds the configured speed limit. The threshold value can be set using device, group or server **speedLimit** attribute. |
| **Fuel Drop** | Sharp fuel level drop over **fuelDropThreshold**. Device has to support fuel level for this event to work. |
| **Fuel Increase** | Sharp fuel level increase over **fuelIncreaseThreshold**. Device has to support fuel level for this event to work. |
| **Geofence Entered** | Device has entered a geofence area. The geofence has to be linked to a device or a group of devices. |
| **Geofence Exited** | Device has exited a geofence area. The geofence has to be linked to a device or a group of devices. |
| **Proximity Enter** | A linked device has come within the configured distance of this device. Set the distance threshold in meters using the **proximityEnterDistance** device attribute (0 disables the event). |
| **Proximity Exit** | A linked device has moved beyond the configured distance from this device. Set the distance threshold in meters using the **proximityExitDistance** device attribute (0 disables the event). |
| **Unaccompanied Motion** | Device has started moving while no linked device is within the configured distance. Set the distance threshold in meters using the **unaccompaniedDistance** device attribute (0 disables the event). |
| **Alarm** | This event type represents any device-reported events/alarms. There are a large number of subtypes. Some alarms might duplicate events, but those are generated directly on the device vs events that are generated on the server side. See the section below for some alarm examples. |
| **Ignition On** | Ignition value has changed from off to on. This event requires the device to continuously report the ignition value. |
| **Ignition Off** | Ignition value has changed from on to off. This event requires the device to continuously report the ignition value. |
| **Maintenance Required** | Event for periodic maintenance. It requires a special maintenance configuration in the settings. Maintenance schedule can be linked to a device or a group of devices. |
| **Driver Changed** | This event is generated when the driver id changes. Driver identification has to be reported by the device. Just linking a driver in settings is not enough. |
| **Media** | Event indicating that the device has uploaded some media data. It can be an image, a video or an audio file. Support varies by protocol. |

## Alarm Types

List of the most commonly used alarms. The list is not exhaustive.

| Type | Description |
| --- | --- |
| **General** | A generic case when the device does not specify the type of alarm. |
| **SOS** | Panic alarm. This type of alarm is usually generated by a button press on the device, but depends on the specific device model. |
| **Vibration** | Usually this alarm is generated when a stationary device starts moving. Often it is based on the accelerometer sensor in the device. |
| **Overspeed** | Speeding alarm. It is similar to the **speed limit exceeded** event, but generated on the device, so it can be more accurate if your device supports it. |
| **Low Power** | External power supply voltage is low. |
| **Low Battery** | Internal battery level is low. |
| **Geofence Enter** | Similar to the server event, but generated on the device, so it can be more accurate if your device supports it. |
| **Geofence Exit** | Similar to the server event, but generated on the device, so it can be more accurate if your device supports it. |
| **Tampering** | The device is being tampered with. It usually indicates that someone has opened the device casing, but sensor type and trigger depends on the specific device model. |

================================================================================
# Source URL: https://www.traccar.org/geofences/
================================================================================

# Geofences

Geofences are geographical zones that can be used to generate events about device movement in and out of the zone.

There are three supported geometry types:

### Polygon

![](/images/documentation/geofences/polygon.png)

### Polyline

![](/images/documentation/geofences/polyline.png)

### Circle

![](/images/documentation/geofences/circle.png)

## Permissions

If a geofence is linked to a device, it means that the system will monitor device entering and exiting that geofence.

If a geofence is linked to a group, it means that the system will monitor all devices in that group entering and exiting the geofence.

If a user has access to a geofence, it means that the user can edit/remove the geofence and can subscribe to the geofence events.

[Calendars](/calendars/) can be linked to geofences. It will limit events generation to the calendar schedule.

## Other Information

Distance around a polyline geofence, when device is considered inside the geofence, can be configured with the **geofence.polylineDistance** configuration parameter (25 meters by default).

The color of a geofence on the map can be set in the geofence attribute **color**. Any valid HTML colors are accepted.

![](/images/documentation/geofences/colors.png)

If you need control device movement along a predefined route, you can use a polyline geofence. Also, if you need control when a device started and ended the route, you can add additional geofences at the start and at the end.

![](/images/documentation/geofences/route.png)

================================================================================
# Source URL: https://www.traccar.org/trips-stops/
================================================================================

# Motion

The standardized **motion** attribute was introduced to have a universal way of handling device motion for all protocols. Some devices report this attribute directly. For other devices it is automatically calculated based on the speed value and the **speedThreshold** parameter. It is also possible to override it using [computed attributes](/computed-attributes/).

The system analyzes positions and switches motion state according to the configuration. Traccar switches state from **stopped** to **moving** if a device is reporting **motion** as **true** for more than **minimalTripDuration** seconds or if trip distance is more than **minimalTripDistance** meters. Traccar switches state from **moving** to **stopped** if a device is reporting **motion** as **false** for more than **minimalParkingDuration** seconds or if **useIgnition** is enabled and the ignition is off. Only continuous periods can switch the state; any fluctuations reset the detection.

The following parameters can be used to configure the detection logic:

* **report.trip.minimalTripDuration** - trips of less than minimal duration and minimal distance are ignored. 300 seconds and 500 meters by default.
* **report.trip.minimalTripDistance** - trips of less than minimal duration and minimal distance are ignored. 300 seconds and 500 meters by default.
* **report.trip.minimalParkingDuration** - stops with less than the minimal duration are not detected as stops. 300 seconds by default.
* **report.trip.minimalNoDataDuration** - gaps in reported positions longer than this value are considered as stops (only in reports). 3600 seconds by default.
* **report.trip.useIgnition** - force switch to stop state if ignition is reported and off. Disabled by default.

## New Trip Logic

If **report.trip.newLogic** is enabled, Traccar uses an alternative motion model based on spatial clustering instead of relying only on speed transitions. Motion starts after the device moves sufficiently far from the last motion reference point, and motion stops after the device remains within a limited area for a minimum time. This approach reduces false stop/start transitions during short pauses. Long gaps in incoming data can also split movement intervals to keep trip timelines more realistic.

* **report.trip.minDistance** - minimum distance required to switch to moving. 200 meters by default.
* **report.trip.minDuration** - minimum stationary time required to switch to stopped. 180 seconds (3 minutes) by default.
* **report.trip.stopGap** - gap duration after which movement can be split around missing data. 3600 seconds (1 hour) by default.

## Trips and Stops

Traccar provides two advanced report types called Trips and Stops. It uses the same motion detection algorithm to determine trips and stops for those reports. The stops report can detect gaps between reported positions and treat them as a stop.

## Fast and Slow Reports

Trips, stops and summary reports have two implementations. The system picks between them automatically based on the requested period: if it is longer than **report.fastThreshold** seconds (one day by default), the fast version is used, otherwise the slow one.

The **slow** version reprocesses all positions in the period and recomputes trips and stops from scratch using the current configuration. The **fast** version instead reuses the **deviceMoving** and **deviceStopped** events that were recorded in real time, which is much faster but less accurate. Some details are missing (for example, maximum speed is reported as zero), and the main drawback is that if positions arrive out of order, the real-time events end up misplaced and fast reports reflect those inaccuracies, while the slow version would sort positions correctly.

## Examples

Following are two examples to illustrate how the logic works.

The first example illustrates the most common case of reporting. This can be a device that has an internal battery or connected to an uninterrupted external power supply. It keeps reporting even when the vehicle stops, but it can go into a sleep mode after some period (rows 19 - 21) if there is no movement.

![](/images/documentation/trips-stops/ideal.png)

Trip 1 is detected because period started from zero speed (initialized as stopped), distance longer than default 500 meters or duration longer than 5 minutes, and it is followed by stop longer than **minimalParkingDuration**.

Stop 1 is detected because it has a duration longer than **minimalParkingDuration** and the speed is close to zero.

Trip 2 is detected from row 27; rows 24 - 25 are ignored as fluctuation. Parameters are more than minimal, and it is preceded and followed by positions with zero speed.

The second example illustrates a case when a device does not have an internal battery and is powered only when ignition is on. Such devices always cold-start and have a delay in GPS fix. For such devices it is common to start reporting with a speed higher than zero. It is impossible to use the same algorithm as in the first example, so additional logic was introduced. Gaps between reporting intervals are detected as stops.

![](/images/documentation/trips-stops/gaps.png)

The **report.trip.minimalNoDataDuration** parameter is set to 600 (10 minutes). You can see that the speed is never less than the **speedThreshold**, but the gap (rows 15 - 28) is more than 10 minutes. It is interpreted as a stop. Preceding trip is detected correctly. Following movement period is not detected as a trip because the algorithm cannot determine when it ends. Only full stops or trips are recorded.

================================================================================
# Source URL: https://www.traccar.org/permissions-groups/
================================================================================

# Permissions

Almost all entities in Traccar have associated permissions and can be linked to user accounts. It applies not only to devices, but also things like geofences, notifications etc.

Administrators and managers can control permissions for their users from the **Users** menu in the settings. It is accessible from the **Connections** menu option in the users list.

When an object is shared between several users, they have full access to it, including the ability to delete it. If you want to avoid it, you can duplicate instances so that each user has their own object. Duplicating a device with the same unique id is not possible, so Traccar provides a special **device readonly** user setting to restrict users for modifying devices.

## Devices

Some entities can be linked to devices. For example, a geofence or a notification can be linked to a device. When linked, it means that it's associated with the device, but it does not affect user permissions. To provide user access, it also needs to be linked to the user account.

For example, if a geofence is linked to a device, it means that the geofence events will be generated for this device.

Linking can be done from the settings. It is accessible from the **Connections** menu option in the devices list.

## Groups

A group represents a group of devices. It is not possible to group any other entities, but you can link some entities to groups the same way you link them to devices. As with devices, linking to a group does not affect user permissions.

For example, if a geofence is linked to a group, it means that the geofence events will be generated for all devices in the group and subgroups.

Linking can be done from the settings. It is accessible from the **Connections** menu option in the groups list.

To add a device to a group, edit the device and select the **Group** under the **Extra** section. Groups can be nested. To add a group to another group, edit the group and select the parent **Group** under the **Extra** section.

================================================================================
# Source URL: https://www.traccar.org/commands/
================================================================================

# Commands

Most of GPS devices support commands. They can be used for device configuration, changing state or requesting information. Commands can be sent via the same network connection used to upload data from the device to the server or via SMS. Many devices support both options.

Traccar command support varries by protocol, but all of the protocols should support at least the **custom command**. Custom command must be in HEX format for most binary protocols and in text for text-based protocols.

SMS needs to be enabled on the server to send command via SMS, and device have to have the **phone** field set correctly. See [SMS API configuration](/http-sms-api/) for more information on how to enabled SMS.

Command results are supported for a limited number of protocols. If it is received and decoded, a command result [event](/events/) will be generated.

In case a command is sent via the network connection and the device in not online, it will be queued. All commands from the device queue will be sent as soon as the device becomes online. It is not recommended to put a lot of commands in queue because some devices ignore next command until they handle previous.

By default all regular users can send commands. It is possible to restrict users from sending arbitrary commands using **limit commands** setting in the account permissions. Such user can only send pre-defined saved commands linked to their account.

## Saved Commands

Often users need to send the same set of commands frequently. **Saved commands** is a way to store a command with parameters to be used later. It is the most useful for the **custom command**.

User can create a **saved command** with desired parameters from the corresponding setting menu:

![](/images/documentation/commands/new.png)

Then link it to a device or a group of devices:

![](/images/documentation/commands/link.png)

To send the command, select it from the list on the **send command** screen:

![](/images/documentation/commands/send.png)

================================================================================
# Source URL: https://www.traccar.org/map-layers/
================================================================================

# Map Layers

Traccar web app supports multiple map layers, including popular providers like OpenStreetMap, Mapbox and many others. By default we use LocationIQ vector map, which is based on OpenStreetMap data.

Some map options are disabled by default and some require an API key. To enable map layers check **Active Maps** under the map section on the preferences page:

![](/images/documentation/maps/preferences.png)

It is also possible to set map layers and keys globally in the server attributes.

## Custom Map

Using a custom layer, it is possible to use any standard tile server with Traccar. For example, it is possible to use an offline or locally hosted map. There are also many third-party services that provide various tile options that can be used with a custom map.

To enable custom map, set **custom map** in the server settings. It has to be a special URL with {x}, {y} and {z} placeholders.

Examples of custom map URLs:

* Google Maps with traffic:

  ```
  https://mt0.google.com/vt/lyrs=m,traffic&hl=en&x={x}&y={y}&z={z}&s=Ga
  ```
* ArcGis Topo:

  ```
  https://services.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}.png
  ```
* ArcGis Imagery:

  ```
  https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}.png
  ```

================================================================================
# Source URL: https://www.traccar.org/computed-attributes/
================================================================================

# Computed Attributes

Computed attributes allow dynamic position attributes modification. Most common use cases for computed attributes:

* Map generic inputs and outputs to specific values (for example, ignition value from a generic input)
* Calculate a value on the server side based on other attributes (for example, fuel consumption from an odometer value)

Computed attributes are applied to all incoming positions of a linked device. To link, go into settings, open devices or groups page, and click on the link icon. Then select attributes in the dropdown.

The order in which computed attributes are applied is defined by the priority parameter. Higher value means it will be executed first. Negative priority values allow you to execute parameters early in the position processing pipeline, before filtering and other handling is done.

Each computed attributes contains 4 fields:

* **Description** - a human readable description
* **Attribute** - name or key of the resulting position attribute
* **Expression** - an expression to compute, written in [JEXL](https://commons.apache.org/proper/commons-jexl/)
* **Type** - output data type (number, boolean or a string)

You can pick from a list of standard attributes with predefined types, but it is also possible to define your own custom attribute.

## Expression

Expression is the core of computed attributes feature. It uses a very flexible [JEXL syntax](https://commons.apache.org/proper/commons-jexl/reference/syntax.html) to compute the result.

All position fields are mapped as primitive objects (latitude, longitude, speed, course etc), they are always defined. Position attributes also mapped as primitive objects (satellites, battery, ignition, distance etc). The set of attributes depends on what the device is reporting.

An empty result of computation (**null**, but not an empty string) will not be stored to position and will clear any existing attribute with the same name.

The expression can be tested on the last position of some device.

![](/images/documentation/computed-attributes/test.png)

Using undefined variables will create warning messages in the log. If you are not certain that the device always reports an attribute, then it is recommended to wrap it in a ternary operator to check if the value is present. A ternary operator is a conditional check with examples for this shown below.

## Examples

You have a device connected to a vehicles electrical system and report its voltage in the "power" attribute, but do not have separate ignition status input. You can try to use the following computed attribute to assign a value to the ignition attribute:

```
Ignition
power ? power > 13.2 : null
Boolean
```

You have a device with some configurable inputs and you connected ignition wire to second one.

It is reported as boolean attribute "in2"

```
Ignition
in2 ? true : false
Boolean
```

or as 0/1

```
Ignition
in2 ? in2 == 1 : false
Boolean
```

or as a second bit in flags attribute

```
Ignition
flags ? (flags & 2) != 0 : false
Boolean
```

You have a device with some configurable analog inputs and connected fuel sensor to first one. Let's say analog input has 10 bit resolution (0..1024) and maximum value 20 volt. Fuel sensor report rest of fuel as voltage from 0 to 12 volts, where 0 V is empty and 12 V is full 40 liters tank.

```
Fuel
adc1 ? adc1 * 0.065 : null
Number
```

Here is a non standard case. There is a device that des not have any inputs to determine ignition. It has an internal battery, and its charger connected to ACC wire inside car. It was noticed that when device is charging it reports 4.18 V battery voltage, if not charging then voltage usually less than 4.18 V. Sometimes it is fluctuating and speed might be used for additional correction.

![](/images/documentation/computed-attributes/battery-chart.png)

```
Ignition
battery < 4.18 && speed < 1 ? false : true
Boolean
```

## Advanced Expressions

Using **new()**, you can create new instances of various Java objects using a fully-qualified java class name or Class. For example, **new("java.lang.Double", 10)** returns 10.0 double value.

* The **new** operator has to be enabled using the **processing.computedAttributes.newInstanceCreation**
* Currently it can only instantiate Double, Float, Integer, Long, Short, Character, Boolean, String and Byte

JEXL syntax allows the declaration of local variables within the expression using the **var** keyword followed by the variable name and its initial value. Local variables can be of any primitive data type, including boolean, numeric, and string types, as well as arrays of primitive types.

Local variables are disabled by default in Traccar and must be enabled in the server configuration file by adding the following line:

```
var maxSpeed = 90 // set the maximum allowed speed in km/h
speed > maxSpeed ? speed - maxSpeed : null
```

For example, you have a device connected to a vehicle's systems and report its speed in the **speed** attribute. You can use the following computed attribute to signal a speeding alarm:

```
var x = 2;
while (x lt 1024) { x = math:pow(x,2); }
x
```

Loops in JEXL allow you to iterate over collections and perform operations on each element. However, they can only be used if enabled in the configuration file with the line:

```
var list = [1, 2, 3, 4, 5];
var result = 0;
for (var item : list) {
  result += item;
}
result
```

JEXL supports **for**, **do while** and **while** loops. In addition, you can also use the **break** and **continue** keywords to control the flow of the loop.

Loop examples:

```
var x = 2; while (x lt 1024) { x = math:pow(x,2); } x
```

```
var list = [1, 2, 3, 4, 5]; var result = 0; for (var item : list) { result += item; } result
```

================================================================================
# Source URL: https://www.traccar.org/configuration-file/
================================================================================

# Configuration File

Most global server parameters are defined in the XML configuration file. On Linux, the file is located at `/opt/traccar/conf/`. On Windows, the location can be set during installation, but by default it is `C:\Program Files\Traccar\conf`. You can edit this file with any text editor -- just ensure it remains valid XML and that special characters are properly escaped.

Some parameters can be configured as attributes on Device, Server, and other objects through the web app. Refer to the badge next to each attribute key to see where a parameter can be used.

For Boolean settings, any value left unspecified is interpreted by the server as `false`.

You can also override configuration values with environment variables, which take precedence over the XML file. To enable this, set `CONFIG_USE_ENVIRONMENT_VARIABLES` to `true`. To convert config key to an environment variable key replace dots with underscore and convert the rest to screaming snake case. For example, `mail.smtp.fromName` will be `MAIL_SMTP_FROM_NAME`.

List of available configuration parameters:

##### [protocol].address config

Network interface for the protocol. If not specified, the server will bind to all interfaces. Multiple addresses (for example, one IPv4 and one IPv6) can be supplied as a comma-separated list.

##### [protocol].port config

Port number for the protocol. Most protocols use TCP on the transport layer. Some protocols use UDP. Some support both TCP and UDP.

##### [protocol].devices config

List of devices for polling protocols. The list should contain unique IDs separated by commas. Used only for polling protocols.

##### [protocol].interval config

Polling interval in seconds. Used only for polling protocols.

##### [protocol].ssl config

Enable SSL support for the protocol. Not all protocols support this.

##### [protocol].timeout config

Connection timeout value in seconds. Because sometimes there is no way to detect a lost TCP connection, old connections stay in an open state. On most systems there is a limit on the number of open connections, so this leads to problems with establishing new connections when the number of devices is high or device data connections are unstable.

##### devicePassword device

Device password. Commonly used in some protocols for sending commands.

##### [protocol].devicePassword config

Device password. Commonly used in some protocols for sending commands.

##### [protocol].mask config

Default protocol mask to use. Currently used only by Skypatrol protocol.

##### [protocol].messageLength config

Custom message length. Currently used only by H2 protocol for specifying binary message length.

##### [protocol].extended config

Enable extended functionality for the protocol. The reason it's disabled by default is that not all devices support it.

##### [protocol].utf8 config

Decode string as UTF8 instead of ASCII. Only applicable for some protocols.

##### [protocol].can config

Enable CAN decoding for the protocol. Similar to 'extended' configuration, it's not supported for some devices.

##### [protocol].ack config device

Indicates whether server acknowledgment is required. Only applicable for some protocols.

Default value: false

##### [protocol].ignoreFixTime config

Ignore device-reported fix time. Useful in case some devices report invalid time. Currently only available for GL200 protocol.

##### [protocol].decodeLow config

Decode additional TK103 attributes. Not supported for some devices.

##### [protocol].longDate config

Use long date format for Atrack protocol.

##### [protocol].decimalFuel config

Use decimal fuel value format for Atrack protocol.

##### [protocol].custom config

Indicates additional custom attributes for Atrack protocol.

##### [protocol].form config

Custom format string for Atrack protocol.

##### [protocol].frameMask config

Frame mask for Atrack protocol.

##### [protocol].config config

Protocol configuration. Required for some devices for decoding incoming data.

##### [protocol].alarmMap config

Alarm mapping for Atrack protocol.

##### [protocol].prefix config

Indicates whether TAIP protocol should have prefixes for messages.

##### [protocol].server config

Some devices require server address confirmation. Use this parameter to configure correct public address.

##### [protocol].speed config

Speed units for the protocol. Possible values: knots (default), kmh, mps, mph.

##### [protocol].format0 config

Custom format string 0 for GlobalSat protocol.

Default value: "TSPRXAB27GHKLMnaicz\*U!"

##### [protocol].format1 config

Custom format string 1 for GlobalSat protocol.

Default value: "SARY\*U!"

##### [protocol].reportColumns config

Custom report columns for Genx protocol.

Default value: "1,2,3,4"

##### suntech.protocolType config device

Protocol type for Suntech.

##### suntech.hbm config device

Suntech HBM configuration value.

##### [protocol].includeAdc config device

Format includes ADC value.

##### [protocol].includeRpm config device

Format includes RPM value.

##### [protocol].includeTemp config device

Format includes temperature values.

##### [protocol].disableCommands config

Disable commands for the protocol. Not all protocols support this option.

##### [protocol].format config device

Protocol format. Used by protocols that have configurable message format.

##### [protocol].dateFormat device

Protocol date format. Used by protocols that have configurable date format.

##### decoder.timezone config device

Device time zone. Most devices report UTC time, but in some cases devices report local time, so this parameter needs to be configured for the server to be able to decode the time correctly.

##### orbcomm.accessId config

ORBCOMM API access id.

##### orbcomm.password config

ORBCOMM API password.

##### smartcar.managementToken config

Smartcar management token used for webhook verification.

##### osmand.minAccuracy config

Minimum accuracy to include. If the value is lower, it will be set to zero.

Default value: 10.0

##### [protocol].alternative config device

Use alternative format for the protocol of commands.

Default value: false

##### [protocol].language config device

Protocol format includes a language field.

Default value: false

##### server.buffering.threshold config

If not zero, enable buffering of incoming data to handle ordering locations. The value is threshold for buffering in milliseconds.

Default value: 3000L

##### server.timeout config

Server wide connection timeout value in seconds. See protocol timeout for more information.

Default value: 1800

##### server.delayAcknowledgement config

Send device responses immediately before writing it in the database.

##### server.nettyBossThreads config

Number of Netty boss threads. If not specified or zero, Netty default value is used.

Default value: 0

##### server.nettyThreads config

Number of Netty worker threads. If not specified or zero, Netty default value is used.

Default value: 0

##### server.statistics config

Address for uploading aggregated anonymous usage statistics. Uploaded information is the same as what you can see on the statistics screen in the web app. It does not include any sensitive data (e.g. locations).

Default value: "https://www.traccar.org/analytics/"

##### fuelDropThreshold server device

Fuel drop threshold value. When fuel level drops from one position to another by more than this value, an event is generated.

Default value: 0.0

##### fuelIncreaseThreshold server device

Fuel increase threshold value. When fuel level increases from one position to another by more than this value, an event is generated.

Default value: 0.0

##### fuelCapacity device

Device fuel tank capacity in liters.

##### speedLimit server device

Speed limit value in knots.

Default value: 0.0

##### proximityEnterDistance device

Distance in meters at which a linked device entering this range triggers a proximity enter event. 0 to disable.

Default value: 0.0

##### proximityExitDistance device

Distance in meters at which a linked device leaving this range triggers a proximity exit event. 0 to disable.

Default value: 0.0

##### unaccompaniedDistance device

Distance in meters that defines "near" for the unaccompanied motion event. If the device starts moving with no linked device within this distance, an event is generated. 0 to disable.

Default value: 0.0

##### disableShare server

Disable device sharing on the server.

##### event.overspeed.thresholdMultiplier config

Speed limit threshold multiplier. For example, if the speed limit is 100, but we only want to generate an event if the speed is higher than 105, this parameter can be set to 1.05. Default multiplier is 1.0.

Default value: 1.0

##### event.overspeed.minimalDuration config

Minimal overspeed duration to trigger the event. Value in seconds.

##### event.overspeed.preferLowest config

Relevant only for geofence speed limits. Use the lowest speed limit from all geofences.

##### event.behavior.accelerationThreshold config

Driver behavior acceleration threshold. Value is in meter per second squared.

##### event.behavior.brakingThreshold config

Driver behavior braking threshold. Value is in meter per second squared.

##### event.ignoreDuplicateAlerts config

Do not generate an alert event if the same alert was present in the last known location.

Default value: true

##### event.status.enable config

Generate device connection status events (deviceOnline, deviceOffline, deviceUnknown) on status changes.

Default value: false

##### event.motion.speedThreshold config device

If the speed is above specified value, the object is considered to be in motion. Default value is 0.01 knots.

Default value: 0.01

##### database.memory config

Enable in-memory database instead of an SQL database.

##### database.driverFile config

Path to the database driver JAR file. Traccar includes drivers for MySQL, PostgreSQL and H2 databases. If you use one of those, you don't need to specify this parameter.

##### database.driver config

Database driver Java class. For H2 use 'org.h2.Driver'. MySQL driver class name is 'com.mysql.jdbc.Driver'.

##### database.url config

Database connection URL. By default Traccar uses H2 database.

##### database.user config

Database user name. Default administrator user for H2 database is 'sa'.

##### database.password config

Database user password. Default password for H2 admin (sa) user is empty.

##### database.changelog config

Path to Liquibase master changelog file.

Default value: "./schema/changelog-master.xml"

##### database.maxPoolSize config

Database connection pool size.

Default value: 20

##### database.checkConnection config

SQL query to check connection status. Default value is 'SELECT 1'. For Oracle database you can use 'SELECT 1 FROM DUAL'.

Default value: "SELECT 1"

##### database.saveOriginal config

Store original HEX or string data as "raw" attribute in the corresponding position.

##### database.throttleUnknown config

Throttle unknown device database queries when it sends repeated requests.

##### database.registerUnknown config

Automatically register unknown devices in the database.

##### database.registerUnknown.defaultCategory config

Default category for auto-registered devices.

##### database.registerUnknown.defaultGroupId config

The group id assigned to auto-registered devices.

##### database.registerUnknown.regex config

Automatically register unknown devices with regex filter.

##### database.positionPeriod config

Limit latest position queries to a certain time period in seconds. This is useful for TimescaleDB to avoid scanning all time chunks. Default value is 7776000 seconds (90 days). Zero value disables the limit.

##### database.saveEmpty config

Store empty messages as positions. For example, heartbeats.

##### database.maxLifetime config

Maximum lifetime in milliseconds of a connection in the pool.

##### database.positionBatchInterval config

If not zero, enable batching of position inserts. The value is the flush interval in milliseconds; positions accumulated during this window are written as a single JDBC batch. Trades up to this much latency for higher throughput on busy servers.

Default value: 0L

##### database.positionBatchSize config

Maximum number of positions written in a single batch. Default value is 100, which is safe across all supported databases including SQL Server (with its 2100 parameter limit). Postgres and MySQL can typically handle much larger batches; raise this for higher drain rate on busy servers.

Default value: 100

##### users.defaultDeviceLimit config

Device limit for self registered users. Default value is -1, which indicates no limit.

Default value: -1

##### users.defaultExpirationDays config

Default user expiration for self registered users. Value is in days. By default no expiration is set.

##### ldap.url config

LDAP server URL. For more info, check <a href="https://www.traccar.org/ldap/">LDAP config</a>.

##### ldap.user config

LDAP server login.

##### ldap.password config

LDAP server password.

##### ldap.force config

Force LDAP authentication.

##### ldap.base config

LDAP user search base.

##### ldap.idAttribute config

LDAP attribute used as user id. Default value is 'uid'.

Default value: "uid"

##### ldap.nameAttribute config

LDAP attribute used as username. Default value is 'cn'.

Default value: "cn"

##### ldap.mailAttribute config

LDAP attribute used as user email. Default value is 'mail'.

Default value: "mail"

##### ldap.searchFilter config

LDAP custom search filter. If not specified, '({idAttribute}=:login)' will be used as a filter.

##### ldap.adminFilter config

LDAP custom admin search filter.

##### ldap.adminGroup config

LDAP admin user group. Used if custom admin filter is not specified.

##### openid.clients config

List of OpenID Connect clients for the built-in provider. Value should be a comma-separated list of 'clientId:clientSecret:redirectUri' entries. Multiple redirect URIs can be specified using '|' as a separator.

##### openid.force config

Force OpenID Connect authentication. When enabled, the Traccar login page will be skipped and users are redirected to the OpenID Connect provider.

##### openid.allowRegistration config

Allow new users authenticated via OpenID Connect to be auto-created even when the server registration setting is disabled. When false, OpenID logins for unknown users are rejected unless the server has registration enabled.

##### openid.clientId config

OpenID Connect Client ID. This is a unique ID assigned to each application you register with your identity provider. Required to enable SSO.

##### openid.clientSecret config

OpenID Connect Client Secret. This is a secret assigned to each application you register with your identity provider. Required to enable SSO.

##### openid.issuerUrl config

OpenID Connect Issuer (Base) URL. This is used to automatically configure the authorization, token and user info URLs if provided.

##### openid.authUrl config

OpenID Connect Authorization URL. This can usually be found in the documentation of your identity provider or by using the well-known configuration endpoint, e.g. https://auth.example.com/.well-known/openid-configuration Required to enable SSO if openid.issuerUrl is not set.

##### openid.tokenUrl config

OpenID Connect Token URL. This can be found the same way as openid.authUrl. Required to enable SSO if openid.issuerUrl is not set.

##### openid.userInfoUrl config

OpenID Connect User Info URL. This can be found the same way as openid.authUrl. Required to enable SSO if openid.issuerUrl is not set.

##### openid.groupsClaimName config

OpenID Connect group scope claim name. If this is not provided, Traccar will use the "groups" scope name.

Default value: "groups"

##### openid.allowGroup config

OpenID Connect group to restrict access to. If this is not provided, all OpenID users will have access to Traccar. This option will only work if your OpenID provider supports the groups scope.

##### openid.adminGroup config

OpenID Connect group to grant admin access. If this is not provided, no groups will be granted admin access. This option will only work if your OpenID provider supports the groups scope.

##### status.timeout config

If no data is reported by a device for the given amount of time, status changes from online to unknown. Value is in seconds. Default timeout is 10 minutes.

Default value: 600L

##### status.ignoreOffline config

List of protocol names to ignore offline status. Can be useful to not trigger status change when devices are configured to disconnect after reporting a batch of data.

##### media.path config

Path to the media folder. Server stores audio, video and photo files in that folder. Sub-folders will be automatically created for each device by unique ID.

Default value: "./media"

##### web.address config

Optional parameter to specify a network interface for the web interface to bind to. By default, the server will bind to all available interfaces.

##### web.port config

Web interface TCP port number. By default, Traccar uses port 8082. To avoid specifying port in the browser you can set it to 80 (default HTTP port).

Default value: 8082

##### web.path config

Path to the web app folder.

Default value: "./web"

##### web.override config

Path to a folder with overrides. It can be used for branding to keep custom logos in a separate place.

Default value: "./override"

##### web.timeout config

WebSocket connection timeout in milliseconds. Default timeout is 5 minutes.

Default value: 300000L

##### web.sessionTimeout config

Authentication session timeout in seconds. By default, there is no timeout.

##### web.console config

Enable database access console via '/console' URL. Use only for debugging. Never use in production.

##### web.debug config

Server debug version of the web app. Not recommended to use for performance reasons. It is intended to be used for development and debugging purposes.

##### web.serviceAccountToken config

A token to log in as a virtual admin account. Can be used to restore access in case of issues with regular admin login. For example, if a password is lost and can't be restored.

##### web.origin config

Cross-origin resource sharing origin header value.

##### web.cacheControl config

Cache control header value. By default, resources are cached for one hour.

Default value: "max-age=3600,public"

##### web.localizationPath config

Path to localization files.

Default value: "./templates/translations"

##### totpEnable server

Enable TOTP authentication on the server.

##### totpForce server

Server attribute that indicates that TOTP authentication is required for new users.

##### server.forward config

Host for raw data forwarding.

##### forward.type config

Position forwarding format. Available options are "url", "json" and "kafka". Default is "url".

Default value: "url"

##### forward.exchange config

Position forwarding AMQP exchange.

Default value: "traccar"

##### forward.topic config

Position forwarding Kafka topic or AMQP routing key.

Default value: "positions"

##### forward.url config device

URL to forward positions. Data is passed through URL parameters. For example, {uniqueId} for device identifier, {latitude} and {longitude} for coordinates.

##### forward.header config

Additional HTTP header that can be used for authorization.

##### forward.retry.enable config

Enable position forwarding retries. When enabled, additional attempts are made to deliver positions. If initial delivery fails because of an unreachable server or an HTTP response different from '2xx', the software waits for 'forward.retry.delay' milliseconds to retry delivery. On subsequent failures, this delay is duplicated. If forwarding is retried for 'forward.retry.count', retrying is canceled and the position is dropped. Positions pending delivery are limited to 'forward.retry.limit'. If this limit is reached, positions are discarded.

##### forward.retry.delay config

Position forwarding retry first delay in milliseconds. Can be set to anything greater than 0. Defaults to 100 milliseconds.

Default value: 100

##### forward.retry.count config

Position forwarding retry maximum retries. Can be set to anything greater than 0. Defaults to 10 retries.

Default value: 10

##### forward.retry.limit config

Position forwarding retry pending positions limit. Can be set to anything greater than 0. Defaults to 100 positions.

Default value: 100

##### event.forward.type config

Events forwarding format. Available options are "json" and "kafka". Default is "json".

Default value: "json"

##### event.forward.exchange config

Events forwarding AMQP exchange.

Default value: "traccar"

##### event.forward.topic config

Events forwarding Kafka topic or AMQP routing key.

Default value: "events"

##### event.forward.url config

Events forwarding URL.

##### event.forward.header config

Events forwarding headers. Example value: FirstHeader: hello SecondHeader: world

##### templates.root config

Root folder for all template files.

Default value: "templates"

##### mail.debug config

Log emails instead of sending them via SMTP. Intended for testing purposes only.

##### mail.smtp.systemOnly config

Restrict global SMTP configuration to system messages only (e.g. password reset).

##### mail.smtp.ignoreUserConfig config

Force SMTP settings from the config file and ignore user attributes.

##### mail.smtp.host config user

The SMTP server to connect to.

##### mail.smtp.port config user

The SMTP server port to connect. Defaults to 25.

Default value: 25

##### mail.transport.protocol config user

Email transport protocol. Default value is "smtp".

Default value: "smtp"

##### mail.smtp.starttls.enable config user

If true, enables the use of the STARTTLS command (if supported by the server) to switch the connection to a TLS-protected connection before issuing any login commands.

##### mail.smtp.starttls.required config user

If true, requires the use of the STARTTLS command. If the server doesn't support the STARTTLS command, or the command fails, the connect method will fail.

##### mail.smtp.ssl.enable config user

If set to true, use SSL to connect and use the SSL port by default.

##### mail.smtp.ssl.trust config user

If set to "\*", all hosts are trusted. If set to a whitespace separated list of hosts, those hosts are trusted. Otherwise, trust depends on the certificate the server presents.

##### mail.smtp.ssl.protocols config user

Specifies the SSL protocols that will be enabled for SSL connections.

##### mail.smtp.username config user

SMTP connection username.

##### mail.smtp.password config user

SMTP connection password.

##### mail.smtp.from config user

Email address to use for SMTP MAIL command.

##### mail.smtp.fromName config user

The personal name for the email from address.

##### sms.http.url config

SMS API service full URL. Enables SMS commands and notifications.

##### sms.http.authorizationHeader config

SMS API authorization header name. Default value is 'Authorization'.

Default value: "Authorization"

##### sms.http.authorization config

SMS API authorization header value. This value takes precedence over user and password.

##### sms.http.user config

SMS API basic authentication user.

##### sms.http.password config

SMS API basic authentication password.

##### sms.http.template config

SMS API body template. Placeholders {phone} and {message} can be used in the template. If value starts with '{' or '[', server automatically assumes JSON format.

##### sms.aws.access config

AWS Access Key with SNS permission.

##### sms.aws.secret config

AWS Secret Access Key with SNS permission.

##### sms.aws.region config

AWS Region for SNS service. Make sure to use regions that are supported for messaging.

##### command.sender device

Command sender type for the device. This overrides standard data or text commands with API-based commands. For example, it can be Traccar Client push commands.

##### command.client.serviceAccount config

Firebase service account JSON for push commands.

##### command.findHub.url device

Google Find Hub service URL.

##### command.findHub.key device

Google Find Hub service API key.

##### notificator.types config

Enabled notification options. Comma-separated string is expected. Example: web,mail,sms

Default value: "web,mail,command"

##### notificator.timeThreshold config

If the event time is too old, we should not send notifications. This parameter is the threshold value in milliseconds. Default value is 15 minutes.

Default value: 15 \* 60 \* 1000L

##### notificator.traccar.key config

Traccar notification API key.

##### notificator.firebase.serviceAccount config

Firebase service account JSON for push notifications.

##### notificator.firebase.mode config

Firebase message delivery mode. Supported values are {@code direct} (notification payload only, default and current behavior), {@code data} (data-only payload for the client app to handle), and {@code mixed} (both notification and data payloads).

Default value: "direct"

##### notificator.pushover.user config

Pushover notification user name.

##### notificator.pushover.token config

Pushover notification user token.

##### notificator.telegram.key config

Telegram notification API key.

##### notificator.telegram.chatId config

Telegram notification chat id to post messages to.

##### notificator.telegram.sendLocation config user

Telegram notification send location message.

##### notificator.telegram.proxy.url config

Telegram notification proxy URL.

##### notificator.whatsapp.token config

WhatsApp Cloud API permanent access token.

##### notificator.whatsapp.phoneNumberId config

WhatsApp Cloud API phone number id.

##### notificator.whatsapp.templateName config

WhatsApp Cloud API template name.

##### notificator.whatsapp.templateLanguage config

WhatsApp Cloud API template language code. Default value is "en\_US".

Default value: "en\_US"

##### notification.expiration.user config

Enable user expiration email notification.

##### notification.expiration.user.reminder config

User expiration reminder. Value in milliseconds.

##### notification.expiration.device config

Enable device expiration email notification.

##### notification.expiration.device.reminder config

Device expiration reminder. Value in milliseconds.

##### notification.block.users config

Block notifications for specific users. The value should be a comma-separated list of internal user ids.

##### report.periodLimit config

Maximum time period for reports in seconds. Can be useful to prevent users from requesting unreasonably long reports. By default, there is no limit.

##### report.fastThreshold config

Time threshold for fast reports. Fast reports are more efficient, but less accurate and missing some information. The value is in seconds. One day by default.

Default value: 86400L

##### report.trip.newLogic config

Enable new trips calculation logic.

Default value: true

##### report.trip.minDistance config device

Distances above the minimum are considered trips.

Default value: 200L

##### report.trip.minDuration config device

If the device doesn't move for the minimum duration, it is considered a stop.

Default value: 180L

##### report.trip.stopGap config device

Gaps of more than specified time are treated as stop/trip/stop based on average speed. Default value is one hour.

Default value: 3600L

##### report.trip.minimalTripDistance config device

Trips less than minimal duration and minimal distance are ignored. 300 seconds and 500 meters are default.

Default value: 500L

##### report.trip.minimalTripDuration config device

Trips less than minimal duration and minimal distance are ignored. 300 seconds and 500 meters are default.

Default value: 300L

##### report.trip.minimalParkingDuration config device

Parking shorter than the minimum duration does not split a trip. Default is 300 seconds.

Default value: 300L

##### report.trip.minimalNoDataDuration config device

Gaps of more than specified time are counted as stops. Default value is one hour.

Default value: 3600L

##### report.trip.useIgnition config device

Flag to enable ignition use for trips calculation.

Default value: false

##### report.ignoreOdometer config device

Ignore odometer value reported by the device and use server-calculated total distance instead. This is useful if device reports invalid or zero odometer values.

Default value: false

##### filter.enable config

Boolean flag to enable or disable position filtering.

Default value: true

##### filter.invalid config device

Filter invalid (valid field is set to false) positions.

##### filter.zero config device

Filter zero coordinates. Zero latitude and longitude are theoretically valid values, but in practice they usually indicate invalid GPS data.

##### filter.duplicate config device

Filter duplicate records (duplicates are detected by time value).

##### filter.outdated config device

Filter messages that do not have GPS location. If they are not filtered, they will include the last known location.

##### filter.future config device

Filter records with fix time in the future. The value is specified in seconds. Records that have fix time more than the specified number of seconds later than current server time would be filtered out.

Default value: 86400L

##### filter.past config device

Filter records with fix time in the past. The value is specified in seconds. Records that have fix time more than the specified number of seconds before current server time would be filtered out.

##### filter.accuracy config device

Filter positions with accuracy less than specified value in meters.

##### filter.approximate config device

Filter cell and wifi locations that are coming from geolocation provider.

##### filter.static config device

Filter positions with exactly zero speed values.

##### filter.distance config device

Filter records by distance. The value is specified in meters. If the new position is closer than this value to the last one, it gets filtered out.

##### filter.maxSpeed config device

Filter records by Maximum Speed value in knots. Can be used to filter jumps to far locations even if Position appears valid or if Position `speed` field reported by the device is also within limits. Calculates speed from the distance to the previous position and the elapsed time. Tip: Shouldn't be too low. Start testing with values at about 25000.

##### filter.minPeriod config device

Filter position if time from previous position is less than specified value in seconds.

##### filter.dailyLimit config device

Throttle positions if the daily limit is exceeded for the device.

##### filter.dailyLimitInterval config device

Throttling interval if the limit is exceeded. The value is in seconds.

##### filter.skipLimit config device

Time limit for filtering in seconds. If the time difference between when the last position was received by the server and when a new position is received by the server is greater than this limit, the new position will not be filtered out.

##### filter.skipAttributes.enable config device

Enable attributes skipping. Attribute skipping can be enabled in the config or device attributes. If position contains any attribute mentioned in "filter.skipAttributes" config key, position is not filtered out.

##### filter.skipAttributes config device

Attribute skipping can be enabled in the config or device attributes. If position contains any attribute mentioned in "filter.skipAttributes" config key, position is not filtered out.

Default value: ""

##### time.override config

Override device time. Possible values are 'deviceTime' and 'serverTime'

##### protocols.enable config

List of protocols to enable. If not specified, Traccar enables all protocols that have port numbers listed. The value is a comma-separated list of protocol names. Example value: teltonika,osmand

##### time.protocols config

List of protocols for overriding time. If not specified, override is applied globally. The list consists of protocol names that can be separated by a comma or a single space character.

##### coordinates.filter config

Replaces coordinates with the last known coordinates if the change is less than 'coordinates.minError' meters or more than 'coordinates.maxError' meters. Helps avoid coordinate jumps during parking periods or jumps to zero coordinates.

##### coordinates.minError config

Distance in meters. Distances below this value get handled as explained in 'coordinates.filter'.

##### coordinates.maxError config

Distance in meters. Distances above this value get handled as explained in 'coordinates.filter'.

##### processing.remoteAddress.enable config

Enable saving device IP address information. Disabled by default.

##### processing.useLinkedDriver config

Use linked driver id for positions if a device does not send driver id.

##### processing.copyAttributes.enable config

Enable copying of missing attributes from last position to the current one. Might be useful if device doesn't send some values in every message.

##### processing.copyAttributes config device

List of attributes to copy. Attributes should be separated by a comma without any spacing. For example: alarm,ignition

##### processing.computedAttributes.deviceAttributes config

Include device attributes in the computed attribute context.

##### processing.computedAttributes.lastAttributes config

Include last position attributes in the computed attribute context.

##### processing.computedAttributes.localVariables config

Enable local variables declaration.

##### processing.computedAttributes.loops config

Enable loop processing.

##### processing.computedAttributes.newInstanceCreation config

Enable new instance creation. When disabled, parsing a script/expression using 'new(...)' will throw a parsing exception.

##### geocoder.enable config

Boolean flag to enable or disable reverse geocoder.

Default value: true

##### geocoder.type config

Reverse geocoder type. Check reverse geocoding documentation for more info.

Default value: "locationiq"

##### geocoder.url config

Geocoder server URL. Applicable only to Nominatim and Gisgraphy providers.

##### geocoder.key config

Provider API key. Most providers require API keys.

Default value: "pk.689d849289c8c63708068b2ff1f63b2d"

##### geocoder.language config

Language parameter for providers that support localization (e.g. Google and Nominatim).

##### geocoder.format config

Address format string. Default value is %h %r, %t, %s, %c. See AddressFormat for more info.

##### geocoder.cacheSize config

Cache size for geocoding results.

##### geocoder.ignorePositions config

Disable automatic reverse geocoding requests for all positions.

Default value: true

##### geocoder.reuseDistance config

Optional parameter to specify minimum distance for new reverse geocoding request. If distance is less than specified value (in meters), then Traccar will reuse last known address.

##### geocoder.onRequest config

Perform geocoding when preparing reports and sending notifications.

Default value: true

##### mapMatcher.enable config

Boolean flag to enable map matcher. When enabled, position coordinates are aligned to the nearest road segment before further processing.

##### mapMatcher.type config

Map matcher provider type. Currently only "traccar" is supported.

Default value: "traccar"

##### mapMatcher.url config

Map matcher service URL.

##### mapMatcher.key config

Map matcher provider API key.

##### geolocation.enable config

Boolean flag to enable LBS location resolution. Some devices send cell tower information and Wi-Fi points when GPS location is not available. Traccar can determine coordinates based on that information using third-party services. Default value is false.

##### geolocation.type config

Provider to use for LBS location. Available options: google, unwired and opencellid. By default, google is used. You have to supply a key that you get from the corresponding provider. For more information, see LBS geolocation documentation.

##### geolocation.url config

Geolocation provider API URL address. Not required for most providers.

##### geolocation.key config

Provider API key. OpenCellID service requires API key.

##### geolocation.processInvalidPositions config

Boolean flag to apply geolocation to invalid positions.

##### geolocation.reuse config

Reuse last geolocation result if network details have not changed.

##### geolocation.requireWifi config

Process geolocation only when Wi-Fi information is available. This makes the result more accurate.

##### geolocation.mcc config

Default MCC value to use if device doesn't report MCC.

##### geolocation.mnc config

Default MNC value to use if device doesn't report MNC.

##### speedLimit.enable config

Boolean flag to enable speed limit API to get speed limit values depending on location. Default value is false.

##### speedLimit.type config

Provider to use for speed limit. Available options: overpass. By default overpass is used.

##### speedLimit.url config

Speed limit provider API URL address.

##### speedLimit.accuracy config

Search radius for speed limit. Value is in meters. Default value is 100.

Default value: 100

##### location.latitudeHemisphere config

Override latitude sign / hemisphere. Useful in cases where value is incorrect because of device bug. Value can be N for North or S for South.

##### location.longitudeHemisphere config

Override longitude sign / hemisphere. Useful in cases where value is incorrect because of device bug. Value can be E for East or W for West.

##### web.requestLog.path config

Jetty request log path. The path must include the string "yyyy\_mm\_dd", which is replaced with the actual date when creating and rolling over the file. Example: ./logs/jetty-yyyy\_mm\_dd.request.log

##### web.requestLog.retainDays config

Set the number of days before rotated request log files are deleted.

##### web.disableHealthCheck config

Disable systemd health checks.

##### web.healthCheck.dropThreshold config

If this parameter is set, Traccar will monitor drops in the number of stored messages. If it drops by more than the threshold, it will mark the service as failing for systemd. Threshold is a value from 0.0 to 1.0. For example, value 0.7 means that the number of messages in the last period is only 70% of what it was in the previous period.

##### web.sameSiteCookie config

Sets SameSite cookie attribute value. Supported options: Lax, Strict, None.

##### web.persistSession config

Enables persisting Jetty sessions to the database.

##### web.url config

Public URL for the web app. Used for notifications, report links, and OpenID Connect. If not provided, Traccar will attempt to get a URL from the server IP address, but it might be a local address.

##### web.showUnknownDevices config

Show logs from unknown devices.

##### web.shareDevice.commands config

Enable commands for a shared device.

##### web.shareDevice.reports config

Enable reports for a shared device.

##### web.mcp.enable config

Enable MCP service.

##### logger.console config

Output logging to the standard terminal output instead of a log file.

##### logger.queries config

Log executed SQL queries.

##### logger.file config

Log file name. For rotating logs, a date is added to the end of the file name for non-current logs.

Default value: "./logs/tracker-server.log"

##### logger.level config

Logging level. Default value is 'info'. Available options: off, severe, warning, info, config, fine, finer, finest, all.

Default value: "info"

##### logger.fullStackTraces config

Print full exception traces. Useful for debugging. By default shortened traces are logged.

##### logger.rotate config

Create a new log file daily. Helps with log management. For example, downloading and cleaning logs. Enabled by default.

Default value: true

##### logger.decodeTextData config

If all bytes are printable characters, log network data as text instead of HEX.

Default value: true

##### logger.rotate.interval config

Log file rotation interval. The default rotation interval is once a day. This option is ignored if 'logger.rotate' = false. Available options: day, hour.

Default value: "day"

##### logger.attributes config

A list of position attributes to log.

Default value: "time,position,speed,course,accuracy,result"

##### broadcast.type config

Broadcast method. Available options are "multicast" and "redis". By default, (if the value is not specified or does not match available options) server disables broadcast.

##### broadcast.interface config

Multicast interface. It can be either an IP address or an interface name.

##### broadcast.address config

Multicast address or Redis URL for broadcasting synchronization events.

##### broadcast.port config

Multicast port for broadcasting synchronization events.

##### broadcast.secondary config

Flag to mark secondary servers. Some tasks, like scheduled reports, will be executed on the main server only.

================================================================================
# Source URL: https://www.traccar.org/notifications/
================================================================================

# Notifications Configuration

Notifications is a way to notify users about important events. Check [events and notifications](/events/) page for more details about different types of events. Traccar supports various channels for delivering notifications, including email, push notifications and many others. This page focuses on the channel configuration.

By default, Traccar has web and email channels enabled. Note that for the email to work, SMTP server needs to be configured per user or globally in the config file. See the corresponding section below for more details on how to configure each channel.

* [Email Configuration](#email)
* [Push Configuration](#push)
* [SMS Configuration](#sms)
* [Telegram Configuration](#telegram)
* [WhatsApp Configuration](#whatsapp)
* [Web Configuration](#web)

To change the URL in notifications, set or change the [web.url](/configuration-file/) configuration parameter.

You can test notification channels from the settings when creating or editing a notification:

![](/images/documentation/notifications/test.png)

## Email Configuration

There are two ways to configure email notifications:

* Server-wide parameters in the server config file
* Per-user configuration via user attributes

All available parameters are listed in the [configuration page](/configuration-file/) (see **mail.smtp.**\* parameters).

Tips:

* Some SMTP servers require the **From** field even if authorization is used. Make sure to set the **mail.smtp.from** parameter.
* If you want to use your Gmail email for notifications please check [Gmail in Traccar](/gmail/) configuration guide.

SSL configuration example:

```
<entry key='mail.smtp.host'>smtp.gmail.com</entry>
<entry key='mail.smtp.port'>465</entry>
<entry key='mail.smtp.ssl.enable'>true</entry>
<entry key='mail.smtp.from'>traccar@gmail.com</entry>
<entry key='mail.smtp.auth'>true</entry>
<entry key='mail.smtp.username'>traccar@gmail.com</entry>
<entry key='mail.smtp.password'>password</entry>
```

STARTTLS configuration example:

```
<entry key='mail.smtp.host'>smtp.gmail.com</entry>
<entry key='mail.smtp.port'>587</entry>
<entry key='mail.smtp.starttls.enable'>true</entry>
<entry key='mail.smtp.from'>traccar@gmail.com</entry>
<entry key='mail.smtp.auth'>true</entry>
<entry key='mail.smtp.username'>traccar@gmail.com</entry>
<entry key='mail.smtp.password'>password</entry>
```

## Push Configuration

Push notifications are sent through the Firebase platform. There are two options for push notifications in Traccar:

* Traccar notifications to the official Traccar Manager apps
* Direct Firebase notifications to your own mobile app

To receive notifications to the official Traccar Manager app from Google Play all you need to do is configure Traccar with the following parameters:

```
<entry key='notificator.types'>traccar,...</entry>
<entry key='notificator.traccar.key'>TRACCAR_KEY</entry>
```

You can find your personal API key on [the Account page](/my-account/). You need to be registered on the website.

To receive Firebase push notifications directly, you need a custom mobile app that is built with your Firebase keys. Server configuration for direct Firebase notifications should look like this:

```
<entry key='notificator.types'>firebase,...</entry>
<entry key='notificator.firebase.serviceAccount'>
  {
    "type": "service_account",
    ...
  }
</entry>
```

You can create a service account from your Firebase console and download the JSON from there as well.

## SMS Configuration

Text notifications are delivered to the phone number in the user account settings. The user must have correctly configured the **Phone** field according to your SMS API provider requirements.

Traccar supports a flexible format to work with many SMS API providers. Check the [HTTP SMS API](/http-sms-api/) documentation for more details and examples.

We also provide a mobile app [Traccar SMS Gateway](/sms-gateway/) for Android. You can use it to send SMS directly from your own mobile phone without using an external service.

An example configuration to work with Traccar SMS Gateway app:

```
<entry key='notificator.types'>sms,...</entry>
<entry key='sms.http.url'>https://www.traccar.org/sms/</entry>
<entry key='sms.http.authorization'>TOKEN</entry>
<entry key='sms.http.template'>
    {
        "to": "{phone}",
        "message": "{message}"
    }
</entry>
```

## Telegram Configuration

Telegram is a popular mobile messaging app. Traccar supports Telegram as one option for delivering notifications.

You need to create a bot first. You can do it using **BotFather** contact in Telegram. It will provide you with a key.

Then you need to create a chat group with your bot and get the chat id. You can do it using the [instructions here](https://stackoverflow.com/a/38388851/2548565).

Traccar configuration example for Telegram:

```
<entry key='notificator.types'>telegram,...</entry>
<entry key='notificator.telegram.key'>BOT_KEY</entry>
<entry key='notificator.telegram.chatId'>CHAT_ID</entry>
```

Chat id can also be configured per user. See user attributes in settings.

## WhatsApp Configuration

WhatsApp notifications are delivered through the WhatsApp Cloud API using a pre-approved message template.

You need to create a WhatsApp business app, configure a phone number, create and approve a template, and generate a permanent access token.

Traccar configuration example for WhatsApp:

```
<entry key='notificator.types'>whatsapp,...</entry>
<entry key='notificator.whatsapp.token'>ACCESS_TOKEN</entry>
<entry key='notificator.whatsapp.phoneNumberId'>PHONE_NUMBER_ID</entry>
<entry key='notificator.whatsapp.templateName'>TEMPLATE_NAME</entry>
<entry key='notificator.whatsapp.templateLanguage'>en_US</entry>
```

Recipient phone number is taken from the user **Phone** field in account settings. Make sure the number is in international format (for example, `+15551234567`) and can receive WhatsApp messages.

## Web Configuration

Web notifications are delivered directly to the web or mobile app when it's open and connected to the server. It does not require any additional configuration and it's enabled by default.

Web notifications are displayed as toasts and are also available in the events panel on the main screen:

![](/images/documentation/notifications/web.png)

Web notifications are delivered instantly without any delay. It is also possible to enable sound for some critical notifications.

## Templates

Traccar uses [Velocity Engine](https://velocity.apache.org/engine/1.7/user-guide.html) for notification templates. Emails use the content and most other channels use the digest string from templates.

Templates can be easily adjusted to your needs or translated. Template **\*.vm** files must be in the UTF-8 encoding.

================================================================================
# Source URL: https://www.traccar.org/troubleshooting/
================================================================================

# Server Troubleshooting

If you have problems with connecting your device to the server please follow the steps below.

1 Find HEX messages in the logs

Check tracking server logs:

* On Linux and Mac it should be in `/opt/traccar/logs`
* On Windows it's usually in `C:\Program Files\Traccar\logs`

If server receives any data from device (even if the port is wrong), you should be able to see HEX messages in the log. If there are HEX messages in the log, go to [point 2](#point-2). If there are no HEX messages in the log, go to [point 4](#point-4).

2 Check if HEX messages are decoded

If there are HEX messages in the logs, you need to check the following things:

* If HEX messages are followed by `Unknown device` warning, then it means that you haven't registered a device on the server. To fix this issue, add a device with the unique ID you see in the log through the web console.
* If HEX messages are followed by `INFO` record with coordinates, then it means that data is decoded correctly. Possibly you need to wait a little bit for device to show up in the web console.
* In any other case proceed to [point 3](#point-3).

3 Investigate why HEX messages are not decoded

There are two main possibilities:

* Your device doesn't send GPS location. One of the reasons might be that it doesn't have a GPS fix. Note that GPS doesn't work indoors and it can take up to 15 or more minutes for device to get a first fix. Be patient.
* You are using wrong port. Proceed to [point 5](#point-5).

4 Investigate why server does not receive data

If you don't see HEX message in the log, it means that server doesn't receive any data from your device. Possible reasons:

* Server is not visible from the Internet. Use [port check tool](/port-check/) to verify. Proceed to [point 6](#point-6) if port is closed.
* Device is configured incorrectly or there is some problem with the device. Contact device vendor for more information.

5 Find correct port for your device

Please read following information:

* If you are using Traccar Client app, the port is always 5055.
* If you are using a Chinese device, you must read about [clones](/clones/).
* For all other devices, please read [protocol identification guide](/identify-protocol/).
* If you were unable to identify the port for your device, please contact [support](/support/). You must provide protocol documentation and/or HEX message samples. A link to a store listing or user manual for a device is not sufficient.

6 Make sure required ports are open

First of all you should check if Traccar process is running and it's listening for ports locally. On Linux and Mac you can use `netstat` and `ps` command line tools. On Windows you can use [Process Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer).

If process is running and listening on ports, but port check tool still shows that ports are inaccessible, then there must be some network issue. It's impossible to cover every possible network problem, but here is a list of most common issues you might want to check for:

* Router is not configured to forward external ports to your server.
* Firewall on your server or router is blocking connections.
* You do not have a public IP address. Contact your ISP or IT department to confirm.

================================================================================
# Source URL: https://www.traccar.org/client-troubleshooting/
================================================================================

# Traccar Client Troubleshooting

If you have problems with connecting Traccar Client app to the server please follow the steps below.

1 Check status screen

Check status screen in the Traccar Client app:

* If you don't see "location update" messages proceed to [point 2](#point-2).
* If you see "send error" messages proceed to [point 3](#point-3).
* If you see location updates without errors, but no data on your server, it might mean that your device is configured with wrong server. Check app configuration.

If application stops working after some time check [point 6](#point-6).

2 Check accuracy parameter

If you don't see "location update" messages, it means that operating system doesn't provide app with location information.

* If accuracy in the app is set to medium or low check [point 4](#point-4).
* If accuracy in the app is set to high check [point 5](#point-5).

3 Check network and server

If you get "send error" messages, it means that server is not accessible or it doesn't accept your message. Possible reasons:

* Your device id is not registered on the server. Make sure you use correct id on the server.
* Network connection issues on your mobile device. Check network connectivity.
* Wrong server URL details. Double-check app configuration.
* Some other server issue. Try [server troubleshooting guide](/troubleshooting/).

4 Check network location provider

When medium (default) or low accuracy is selected in the app, it uses network location provider which determines location based on cell towers and wifi access points around device. Possible reasons for no location update:

* Network location provider requires internet access. Check network connectivity.
* Application doesn't have required permissions. Check system settings.
* Location services are disabled completely. Check system settings.
* On Android check that network location provider is enabled in system settings.

5 Check GPS location provider

When high accuracy is selected in the app, it uses GPS location provider which determines location based on signal for GPS satellites. Possible reasons for no location update:

* No GPS signal. Remember that a GPS fix takes time and only works with clear sky visibility. GPS does not work indoors.
* Application doesn't have required permissions. Check system settings.
* Location services are disabled completely. Check system settings.
* On Android check that GPS location provider is enabled in system settings.

6 Common background issues

Both Apple and Google are trying to restrict background processes to give user better battery life, but it affects application like Traccar Client which require persistent background execution. Common issues to check for:

* On iOS if you swipe app off the screen, operating system actually kills the process and app will no longer be able to report.
* Make sure that in iOS settings background execution is allowed and Traccar Client has permission to always access location services.
* On recent versions of Android make sure that you have added Traccar Client to battery optimization exceptions.
* Some Android vendors have their own battery optimization in addition to standard Android system. Make sure those are disabled or the app is added to exceptions.
* Check the [Don't Kill My App](https://dontkillmyapp.com/) website for more vendor-specific details.

If you still have issues, please collect operating system logs.

================================================================================
# Source URL: https://www.traccar.org/reverse-geocoding/
================================================================================

# Reverse Geocoding

Reverse geocoding is the process of converting a location into a street address. Traccar supports several reverse geocoding providers, including some open-source options.

#### Traccar Geocoder

Traccar Geocoder is our own [open source](https://github.com/traccar/traccar-geocoder) reverse geocoding service built on OpenStreetMap data. It is hyper-optimized for speed and uses the Nominatim response format. A [hosted option](/product/geocoder/) is also available.

Configuration parameters:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>nominatim</entry>
<entry key='geocoder.url'>https://geocoder.traccar.org/reverse</entry>
<entry key='geocoder.key'>YOUR API KEY</entry>
```

#### LocationIQ

LocationIQ combines multiple datasets (OpenAddress, GNAF, OSM, Geonames, etc.) to provide affordable geolocation APIs worldwide. You need to sign up for an account at LocationIQ.com to get your API key.

There is a generous free monthly quota and several paid plans available.

Configuration parameters for LocationIQ:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>locationiq</entry>
<entry key='geocoder.key'>YOUR API KEY</entry>
```

Note: if Europe is geographically closer to you, change the URL to `https://eu1.locationiq.com/v1/reverse.php`.

#### Google

Google reverse geocoding is a part of Geocoding API. It provides one of the best data quality, but it's also one of the highest prices among all available providers.

Google provides a monthly credit, so the API can be used for free up to some limit. After that you get charged a fee per 1000 requests.

Configuration parameters:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>google</entry>
<entry key='geocoder.key'>YOUR API KEY</entry>
```

#### Nominatim

Nominatim is an open source software for geocoding services. Several companies provide hosted instances of Nominatim that you can query via an API. You can host your own Nominatim server as well.

Please DO NOT use OSM hosted version of Nominatim. It should only be used for low volume testing purposes.

Configuration parameters for Nominatim server example:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>nominatim</entry>
<entry key='geocoder.url'>https://nominatim.openstreetmap.org/reverse</entry>
```

#### Gisgraphy

Gisgraphy is a free and open source framework that offers the ability to do geolocalisation and geocoding via Java APIs or REST webservices. There are number of free and paid cloud providers or you can self host a Gisgraphy server.

Configuration parameters example:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>gisgraphy</entry>
<entry key='geocoder.url'>http://services.gisgraphy.com/reversegeocoding/search</entry>
```

#### Geocode Farm

Geocode Farm is another geocoding provider. They have a small free daily quota and there several paid plans available.

Configuration parameters:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>geocodefarm</entry>
<entry key='geocoder.key'>YOUR API KEY</entry>
```

#### OpenCage Geocoder

OpenCage Geocoder combines multiple geocoding systems in the background. Each is optimized for different parts of the world and types of requests. You need to have an API key to use this type of geocoder.

Configuration parameters:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>opencage</entry>
<entry key='geocoder.url'>http://api.opencagedata.com/geocode/v1</entry>
<entry key='geocoder.key'>YOUR API KEY</entry>
```

#### Base Adresse Nationale

Base Adresse Nationale is a free service limited to French addresses and 10 requests per IP per second.

Configuration parameters:

```
<entry key='geocoder.enable'>true</entry>
<entry key='geocoder.type'>ban</entry>
```

#### Other providers

Other reverse geocoding providers supported by Traccar:

* MapQuest
* Bing Maps
* Geocode.xyz
* Here
* MapMyIndia
* TomTom
* PositionStack

## Additional parameters

By default reverse geocoding is only done when user requests reports or explicitly clicks "show address" button in the app. You can disable this behavior using the following parameters:

```
<entry key='geocoder.onRequest'>false</entry>
<entry key='geocoder.ignorePositions'>false</entry>
```

You can also save on the reverse geocoding API cost by avoiding requests if location hasn't changed significantly from the last one. Distance threshold is set in meters:

```
<entry key='geocoder.reuseDistance'>10</entry>
```

================================================================================
# Source URL: https://www.traccar.org/secure-connection/
================================================================================

# Secure Connection

By default, Traccar serves its web interface and API over the standard HTTP protocol, which does not provide encryption. This guide explains how to configure Traccar to use HTTPS with SSL/TLS encryption for secure traffic. While the examples focus on Ubuntu Linux, the same approach applies to other platforms.

Traccar does not natively support secure connections, but you can enable them by running it behind a proxy server. In this guide, we will use the Caddy server.

To obtain a valid SSL certificate, you need a registered domain name. See our documentation on how to register and configure a [custom domain name](/domain-name/) for Traccar. If you don't yet own a domain, we recommend [Namesilo](https://www.namesilo.com/?rid=a0ca198nr), which offers consistently low prices for both new registrations and renewals.

First, install the latest version of Caddy. The commands below are for Ubuntu and Debian. For other platforms, refer to the [Caddy installation documentation](https://caddyserver.com/docs/install).

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
chmod o+r /usr/share/keyrings/caddy-stable-archive-keyring.gpg
chmod o+r /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

Next, update the Caddy configuration file:

```
sudo nano /etc/caddy/Caddyfile
```

Paste the following content into the file, replacing **demo.traccar.org** with your domain:

```
demo.traccar.org {
  reverse_proxy localhost:8082
}
:80 {
  reverse_proxy localhost:8082
}
```

Restart the Caddy service:

```
sudo systemctl restart caddy
```

If your domain is configured correctly, Caddy will automatically obtain and install an SSL certificate for it using Let's Encrypt.

================================================================================
# Source URL: https://www.traccar.org/openid-sso/
================================================================================

# OpenID Connect (Single Sign-On)

Traccar can authenticate users with a third-party identity provider supporting the OpenID Connect protocol. This includes enterprise directories such as Okta, self-hosted apps like Authelia and public services like Google.

The authentication process is as follows:

* Traccar redirects the user to the identity provider
* The user authenticates with the identity provider
* The identity provider redirects the user to Traccar with an authorization code
* Traccar validates the callback and requests the users info from the identity provider
* If a user is found with a matching email, they are authenticated, if not a new user is created

As with the LDAP integration, an internal representation of the user still exists, and admin rights can automatically be given using the **adminGroup** configuration option.

## Configuration

* **openid.force** - disables internal authentication, only OpenID users can login.
* **openid.clientId** - This is a unique ID assigned to each application you register with your identity provider and is required to enable SSO (required).
* **openid.clientSecret** - This is a secret assigned to each application you register with your identity provider and is required to enable SSO (required).
* **openid.issuerUrl** - The base URL of your identity provider (required if openid.authUrl, openid.tokenUrl and openid.userInfoUrl are not set).
* **openid.authUrl** - The endpoint users are forwarded to during the login process (required if openid.issuerUrl is not set).
* **openid.tokenUrl** - The endpoint providing a token to Traccar (required if openid.issuerUrl is not set).
* **openid.userInfoUrl** - The endpoint which provides user information to Traccar (required if openid.issuerUrl is not set).
* **openid.allowGroup** - A group to restrict access to.
* **openid.adminGroup** - The group to grant admin access to.

Additionally, you will most likely need to set the **web.url** property to provide the identity provider with the correct redirect URL.

In nearly all cases, the required configuration URLs will automatically be fetched from the "well-known" endpoint if openid.issuerUrl is set.

You can also configure OpenID manually by setting openid.authUrl, openid.tokenUrl and openid.userInfoUrl and omitting openid.issuerUrl.

An example of the well-known endpoint for Google is **<https://accounts.google.com/.well-known/openid-configuration>**, with the issuer URL of **<https://accounts.google.com>**.

## Provider-specific example - Google

* First, create a project in the [Google Cloud Console](https://console.cloud.google.com/)
* Go to APIs and services > OAuth consent screen and configure according to your needs. The important settings are:
  + Authorized domains - any domains you will use to host Traccar must be added here
  + Scopes - Add openid, userinfo.profile and userinfo.email
  + Test users - Add yourself and your users. Until the app is published, these are the only users who will have access to Traccar.
* Go to APIs and services > Credentials and create an OAuth client ID. It's important to add an authorized redirect URL to /api/session/openid/callback on your Traccar instance.![](/images/documentation/openid/openid-a.png)
* Configure Traccar with the details from Google Cloud:
  + openid.clientId - Google Client ID
  + openid.clientSecret - Google Client Secret
  + openid.issuerUrl - <https://accounts.google.com>
  + web.url - URL of your Traccar instance![](/images/documentation/openid/openid-b.png)
* Restart Traccar and click **Login with OpenID**
* You should see a Google login consent screen and be able to access your Traccar instance

Note: The Google OpenID endpoint does not provide a "groups" claim, so openid.adminGroup will not work with it. You will need to configure a local admin user before enabling SSO in Traccar.

## Implementation details

Traccar uses the OpenID Connect code flow and requires the following scopes:

* openid
* email
* profile
* groups (if openid.adminGroup) is set

The Traccar callback URLs are

* **{web-url}/api/session/openid/callback** for web
* **org.traccar.manager://api/session/openid/callback** for mobile

================================================================================
# Source URL: https://www.traccar.org/optimization/
================================================================================

# Optimization

This page provides information on configuring Traccar and operating system to work with a large number of devices and users.

In addition to the server optimization, we recommend a regular clean up of old location history and logs. Check [clear history](/clear-history/) page for more details on how to configure it.

## Increasing connection limit on Linux

You can check hard and soft limits for current user using the following command:

```
cat /proc/$(pidof java)/limits | grep "open files"
```

On most systems by default the limit is 1024 connections.

To increase the limit add the following lines to **/etc/security/limits.conf**:

```
* soft  nofile 50000
* hard  nofile 50000
```

On some versions of Ubuntu there is an issue and you need to do the following:

* Edit **/etc/systemd/user.conf** and add **DefaultLimitNOFILE=50000**
* Edit **/etc/systemd/system.conf** and add **DefaultLimitNOFILE=50000**

Make sure you use a number higher than your number of devices because when device reconnects it might consume two or even more connection for some period of time.

A system restart is required for the changes to take effect.

## Scaling beyond 65k connections

After following instructions above you might find that connection limit doesn't increase past 65k or even smaller number. This is because of the **vm.max\_map\_count** variable.

To fix the issue you need to modify "/etc/sysctl.conf":

```
vm.max_map_count = 250000
fs.file-max = 250000
net.ipv4.ip_local_port_range = 1024 65535
```

The first two parameters allow to increase total limit of connections to 250k. The last parameter is important to increase the local port range, which would allow more connections on a single port. By default the range is usually around 32k.

Don't forget to restart the system after modifying the file.

## Setting connection timeout in Traccar

Operating systems have a timeout for all TCP connections, but it's usually very high. For example, on Linux it's common to have a 2 hours timeout by default. It means that if your device re-connects without gracefully closing connection, then it will leave a stale connection on the server that consumes server resources and is counted against the total connection limit. When network connection is poor, a single device can easily create tens or even hundreds of connections within 2 hour period.

To avoid the problem, it is recommended to set the connection timeout in Traccar server. You can use **server.timeout** or **protocol.timeout** option in [the config file](/configuration-file/). It is recommended to set the value (in seconds) to slightly higher than your device reporting interval.

## Service configuration parameters

Traccar uses Java virtual machine, so it has restrictions on the amount of memory it can use on the system.

You can change Java heap size by adding following parameters to the service config:

* **-Xmx** to specify the maximum heap size
* **-Xms** to specify the initial Java heap size

On Linux the service config located at **/etc/systemd/system/traccar.service**:

```
ExecStart=/opt/traccar/jre/bin/java -Xmx1G -jar tracker-server.jar conf/traccar.xml
```

In above example the maximum heap size is set to 1GB.

## Selecting database engine

By default Traccar uses embedded H2 database system. It's used to simplify initial set up and configuration of the server software, but for any production environment it's strongly recommended to use a fully-featured database engine. For large installations we recommend [PostgreSQL](/postgresql/), optionally with the TimescaleDB extension for time-series data. Traccar also supports MySQL, MariaDB and Microsoft SQL Server.

Make sure that the database is configured appropriately to handle the amount of data and traffic you plan to have. Default cache size and other configuration parameters might not be the best for your use case.

================================================================================
# Source URL: https://www.traccar.org/advanced/
================================================================================

# Advanced Documentation

|  |  |
| --- | --- |
| [Custom Domain Name](/domain-name/) | Point a registered domain at your Traccar server. |
| [Secure Connection (Windows)](/secure-connection-windows/) | Set up HTTPS on Windows using the Caddy reverse proxy. |
| [MySQL Optimization](/mysql-optimization/) | Tune MySQL for high device counts and reporting frequency. |
| [Clear History (MySQL)](/clear-history/) | Automate cleanup of old positions, events, and logs on Linux. |
| [Clear History (MS SQL)](/clear-history-ms-sql/) | Scheduled cleanup job for Microsoft SQL Server. |
| [Data Forwarding](/forward/) | Forward positions, events, and raw protocol traffic to external systems. |
| [LBS Geolocation](/lbs-geolocation/) | Resolve device location from Wi-Fi and cell tower data when GPS is unavailable. |
| [Web App Branding](/branding-web/) | Customize logos and colors of the web app from server settings. |
| [Calendars](/calendars/) | Define recurring schedules using standard iCal files. |
| [Traccar Client Configuration](/client-configuration/) | Reference for the Traccar mobile client settings. |
| [Forwarder-Only Setup](/forwarder-only/) | Minimal install that decodes and forwards data with most features disabled. |

================================================================================
# Source URL: https://www.traccar.org/install-digitalocean/
================================================================================

# Install Traccar on DigitalOcean VPS

Use following link to register a DigitalOcean account, receive $200 credit and support our open source project:

* <https://m.do.co/c/9c8f727601b1>

## Commands

Install unzip utility and MySQL server

```
apt update && apt -y install unzip mysql-server
```

Set database password and create a new database

```
mysql -u root --execute="ALTER USER 'root'@'localhost' IDENTIFIED BY 'root'; GRANT ALL ON *.* TO 'root'@'localhost' WITH GRANT OPTION; FLUSH PRIVILEGES; CREATE DATABASE traccar;"
```

Download the latest installer

```
wget https://www.traccar.org/download/traccar-linux-64-latest.zip
```

Unzip the file and run the installer

```
unzip traccar-linux-*.zip && ./traccar.run
```

Update the configuration file to use MySQL database

```
cat > /opt/traccar/conf/traccar.xml << EOF
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE properties SYSTEM 'http://java.sun.com/dtd/properties.dtd'>

<properties>

    <entry key='database.driver'>com.mysql.cj.jdbc.Driver</entry>
    <entry key='database.url'>jdbc:mysql://localhost/traccar?zeroDateTimeBehavior=round&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true&amp;useSSL=false&amp;allowMultiQueries=true&amp;autoReconnect=true&amp;useUnicode=yes&amp;characterEncoding=UTF-8&amp;sessionVariables=sql_mode=''</entry>
    <entry key='database.user'>root</entry>
    <entry key='database.password'>root</entry>

</properties>
EOF
```

Start Traccar service

```
service traccar start
```

Note that you can use above commands on any Ubuntu server, not just DigitalOcean.

================================================================================
# Source URL: https://www.traccar.org/linux/
================================================================================

# Linux Installation

Linux package should work with most Linux distributions that use **systemd**. Currently x64 (x86\_64) and ARM versions of the installer are available.

If you are using Ubuntu, check the [Installation on DigitalOcean](/install-digitalocean/) guide for more detailed installation instructions and a video.

## Install

* [Download](/download/) and extract the installer package
* Execute **traccar.run** file
  + `sudo ./traccar.run`
* Start systemd service
  + `sudo systemctl start traccar`

## Uninstall

* Stop systemd service
  + `sudo systemctl stop traccar`
* Remove systemd service
  + `sudo systemctl disable traccar`
  + `sudo rm /etc/systemd/system/traccar.service`
  + `sudo systemctl daemon-reload`
* Remove traccar directory
  + `sudo rm -R /opt/traccar`

By default Traccar is executed with root user privileges. If you want to run it as a regular user, check [run as non-root](/run-as-non-root/) documentation.

================================================================================
# Source URL: https://www.traccar.org/windows/
================================================================================

# Windows Installation

Windows package has been tested on the most recent versions of Microsoft Windows, including server editions.

## Install

* [Download](/download/) and extract the installer package
* Run **traccar-setup.exe** and follow instructions
* Start **traccar** service from the system **Services** console

## Uninstall

* Stop **traccar** service from the system **Services** console
* Remove Traccar program from the Control Panel

================================================================================
# Source URL: https://www.traccar.org/docker/
================================================================================

# Docker

Traccar provides official Docker images that are available directly on Docker Hub.

Recommended tags:

* 6.14.5, 6.14, 6, 6.14.5-alpine, 6.14-alpine, 6-alpine, latest
* 6.14.5-debian, 6.14-debian, 6-debian, debian
* 6.14.5-ubuntu, 6.14-ubuntu, 6-ubuntu, ubuntu

Multi-platform images available:

* Alpine: linux/amd64, linux/arm64
* Debian: linux/amd64, linux/arm64
* Ubuntu: linux/amd64, linux/arm64

## Container creation example

Command to create a container using the default database and configuration:

```
docker run \
--name traccar \
--hostname traccar \
--detach --restart unless-stopped \
--publish 80:8082 \
--publish 5000-5300:5000-5300 \
--publish 5000-5300:5000-5300/udp \
traccar/traccar:latest
```

Publishing the full `5000-5300` range starts a separate `docker-proxy` process for every port (for both TCP and UDP), which consumes a large amount of memory and can leak connections under load. To avoid this, disable the userland proxy so Docker uses iptables NAT instead by adding the following to `/etc/docker/daemon.json` and restarting Docker:

```
{
  "userland-proxy": false
}
```

Alternatively, publish only the ports for the protocols that you actually use.

It is usually a good idea to mount the logs, data, and configuration file from the host. Make sure to create the directories and the configuration file first.

```
--volume /opt/traccar/logs:/opt/traccar/logs:rw
--volume /opt/traccar/traccar.xml:/opt/traccar/conf/traccar.xml:ro
--volume /opt/traccar/data:/opt/traccar/data:rw
```

## Production deployment

The default H2 database is not recommended for production use. For non-testing deployments, we recommend TimescaleDB for large installations or MySQL for smaller servers.

The following Docker Compose examples can serve as a starting point for your deployment with a production-grade database:

* [Traccar with MySQL](https://github.com/traccar/traccar/blob/master/docker/compose/traccar-mysql.yaml)
* [Traccar with TimescaleDB (PostgreSQL)](https://github.com/traccar/traccar/blob/master/docker/compose/traccar-timescaledb.yaml)

================================================================================
# Source URL: https://www.traccar.org/upgrading-traccar/
================================================================================

# Upgrading Traccar

Upgrade steps:

1. Back up the database.
2. Back up the **traccar.xml** configuration file (if you have modified it).
3. Back up the **media** folder (if it exists).
4. Uninstall the previous version of Traccar.
5. Install the new version of Traccar.
6. Restore the **media** folder (if applicable).
7. Restore the **traccar.xml** configuration file (if applicable).
8. Start the Traccar service.

These instructions assume that only the main configuration file has been modified. If you have changed other files (for example, templates), re-apply those changes to the new version. Older templates may not be compatible with newer releases.

================================================================================
# Source URL: https://www.traccar.org/horizontal-scaling/
================================================================================

# Horizontal Scaling

Starting from version 5.4, it is possible to run several instances of Traccar in parallel to handle large number of devices. It was not possible on older versions because of an internal cache. There is still a cache, but it is simplified only stores data about connected devices and can be synchronized between distributed instances.

Traccar utilizes multicast messages for synchronization. You need to enable it in the configuration file like this:

```
<entry key='broadcast.interface'>en0</entry>
<entry key='broadcast.address'>230.3.3.3</entry>
<entry key='broadcast.port'>7001</entry>
```

Replace interface, address and port if needed to reflect your network setup.

In addition to the above, make sure all instances are configured to work with the same database.

================================================================================
# Source URL: https://www.traccar.org/postgresql/
================================================================================

# PostgreSQL

By default, Traccar uses an embedded H2 database, but we don't recommend it for production use. If you want to use another SQL database instead, replace the following lines in the configuration file:

```
<entry key='database.driver'>org.h2.Driver</entry>
<entry key='database.url'>jdbc:h2:./data/database</entry>
<entry key='database.user'>sa</entry>
<entry key='database.password'></entry>
```

Configuration parameters for PostgreSQL (replace [HOST], [DATABASE], [USER], and [PASSWORD] with the appropriate values; for a local database use **localhost** as the HOST):

```
<entry key='database.driver'>org.postgresql.Driver</entry>
<entry key='database.url'>jdbc:postgresql://[HOST]/[DATABASE]</entry>
<entry key='database.user'>[USER]</entry>
<entry key='database.password'>[PASSWORD]</entry>
```

## TimescaleDB

TimescaleDB is an open-source time-series database. It is a PostgreSQL extension that optimizes storage and query performance for time-based data such as the location history in Traccar. We recommend using it for large-scale Traccar deployments.

The configuration is the same as the PostgreSQL sample above. If you want to deploy Traccar pre-configured with TimescaleDB, you can use this sample Docker Compose configuration:

* [Traccar with TimescaleDB](https://github.com/traccar/traccar/blob/master/docker/compose/traccar-timescaledb.yaml)

================================================================================
# Source URL: https://www.traccar.org/build/
================================================================================

# Build from Source

Traccar server consists of a [server](https://github.com/traccar/traccar) and a [web interface](https://github.com/traccar/traccar-web). Make sure you get both repositories if you want web interface to work. Web interface is a git submodule in the main Traccar repository, so you can checkout everything you need with a single git command:

```
git clone --recursive https://github.com/traccar/traccar.git
```

There are three parts for building the full package:

* Build Server
* Build Web App
* Generate Installers (optional)

In most cases all you need is build Traccar from source code and replace the original files on the server with the newly compiled ones.

The following instructions are only for building the server and web apps. Android and iOS apps are standard Android Studio and Xcode projects, so there are no special instructions required.

For the IDE (Integrated Development Environment) we recommend using [IntelliJ IDEA](/build-in-intellij-idea/) for the server code and VS Code for the web app.

## Build Server

To build the server binary file (JAR) use following command:

```
./gradlew assemble
```

It will automatically download all dependencies and generate **tracker-server.jar** in the **target** subfolder. Now you can just replace the file in your Traccar installation and restart the service for changes to take effect. Make sure you have a compatible version of Traccar installed. Use the latest release when using the **master** branch for development.

## Build Web App

The web app is located in the root folder of the **traccar-web** repository.

It uses React and Material UI frameworks. It relies on Vite for running and building the project with minimal additional configuration.

To run the project in development mode:

* Make sure you have Traccar backend running locally
* Install dependencies using **npm install** command
* Run development server using **npm start** command
* We recommend using Visual Studio Code as the IDE

To compile a release build, run **npm run build**.

## Generate Installers (optional)

You almost never need to do this. The only reason you would want to generate an installer is to distribute your version of Traccar server.

You need to have a Linux system as the script is designed for Linux. We recommend the latest version of Ubuntu. If you are using Windows, you can install Ubuntu in a virtual machine.

To generate an installer, execute the **setup/package.sh** shell script in the terminal. The script will check all requirements and show if anything is missing and where to download the missing package.

================================================================================
# Source URL: https://www.traccar.org/traccar-api/
================================================================================

# Traccar API

Traccar provides an API to access all service functionality. Detailed documentation can be found in the [API Reference](/api-reference/) or the [OpenAPI spec file](https://raw.githubusercontent.com/traccar/traccar/master/openapi.yaml).

There are several user authorization options:

* Using session cookies (see **session** endpoint)
* [Standard HTTP authorization](https://en.wikipedia.org/wiki/Basic_access_authentication#Client_side) header
* Bearer token authentication

Official Traccar API documentation:

* [API Reference](/api-reference/)

To submit positions to the server, devices and apps use one of the many supported tracking protocols. [OsmAnd](/osmand/) is a simple and common one, which makes it a good starting point.

## Access token

As an alternative to the email and password login, there is an option to use an account token for authorization. A token can be generated from the web app or via an API call.

To create a session use **token** query parameter in session get request:

```
/api/session?token=USER_TOKEN
```

## WebSocket API

In addition to the REST API, we provide an access to a WebSocket endpoint for live updates. Endpoint for the WebSocket connection:

```
/api/socket
```

Session cookie is the only authorization option for the WebSocket connection.

Each message in the WebSocket stream uses the same universal JSON format:

```
{
  "devices": [...],
  "positions: [...],
  "events": [...]
}
```

Each array contains standard JSON objects:

* [Device](https://github.com/traccar/traccar/blob/master/src/main/java/org/traccar/model/Device.java)
* [Position](https://github.com/traccar/traccar/blob/master/src/main/java/org/traccar/model/Position.java)
* [Event](https://github.com/traccar/traccar/blob/master/src/main/java/org/traccar/model/Event.java)

If a message does not contain any objects of a certain type, the key would not be included in the JSON structure. In most cases a message contain a single type of objects.

================================================================================
# Source URL: https://www.traccar.org/architecture/
================================================================================

# Traccar Architecture

![](/images/architecture.svg)

## Netty Pipeline

Traccar system is based on the Netty framework.

For each network channel or connection Traccar creates a pipeline of event handlers. Incoming messages from GPS devices are received as binary buffers and are separated into frames, decoded into an internal position model and eventually stored in the database. Outgoing messages (commands) are going in the opposite direction and are starting as an internal command model and eventually get converted into binary buffers that are sent to devices. Each protocol has its own pipeline, but most of the supported protocol include following main handlers in their pipelines:

* Frame decoder (only TCP protocols) - splits incoming buffer into full single frames/messages, which can contain one or more location data points
* String encoder (only text-based protocols) - converts outgoing text message into a network buffer
* String decoder (only text-based protocols) - converts incoming network buffer into a text message ready for parsing
* Protocol encoder - converting universal command model into a message format specific to the protocol
* Protocol decoder - converting message in protocol-specific format into a one or several universal position objects
* Utility handlers - range from handlers calculating travelled distance to handlers that perform reverse geocoding
* Event handlers - generate events based on the information in the position object
* Data handler - stores decoded data in the database
* Main handler - responsible for error handling, logging and network connection management

Pipeline for each protocol is configured in the corresponding protocol class, which also defines additional protocol-specific configuration, including a list of supported commands.

## Events and Notifications

Events can be reported directly by a GPS device, or events can be generated on the server side based on specific data and conditions. All events, except for the connection status events, are generated in network pipeline handlers based on the data in the position model.

Generated events are delivered to appropriate users via following channels:

* Email - via SMTP service
* SMS - via SMS API service
* Web notification - via an active WebSocket connection
* Push notification - via Firebase API
* Telegram - via the Telegram Bot API
* WhatsApp - via the WhatsApp Business API

Delivery logic for each channel is implemented in a corresponding Notificator class. Notificators can be enabled and disabled in the configuration file.

## Database

Server can be configured to work with most popular SQL database engines, including PostgreSQL, MySQL, MariaDB and Microsoft SQL Server.

Initial database schema creation and subsequent database schema upgrade migrations are done using Liquibase library. It allows to define schema in universal format in changelog XML files and generate database-specific SQL queries at runtime.

For fast access to the data, most of the current information for connected devices is cached internally in memory. To avoid outdated cache issues, it is recommended to make any changes using the API instead of direct database queries.

## Web Server

Traccar system includes an embedded Jetty web server. Jetty is a stable and mature implementation of Java HTTP server and Servlet container. It is used in many popular products and frameworks.

In Traccar system Jetty is used to serve the following main components:

* Web API
* Web application

Following sections describe each component in more details.

## Web API

Interface consists of the two main parts:

* REST API
* WebSocket

REST API is implemented using standard Java EE API for RESTful web services. Each model in the system has its own dedicated endpoint. Most URL endpoints accept following methods:

* GET - retrieve a collection of objects based on user access and / or query parameters
* POST - add new object
* PUT - update existing object
* DELETE - remove object

In addition to the REST API, which only allows client driven request-based usage, Traccar provides WebSocket endpoint to get instant updates from the server. It is used for device status updates, location updates and event notifications.

Both REST API and WebSocket connection use same authentication and authorisation mechanisms. For more information about API and security see the [API documentation](/traccar-api/).

## Web Application

Traccar system includes a web application for managing users, devices and other information. The web app relies on the API described above.

Application is based on the React framework with Materia UI for the user interface. Data flow is managed by the Redux. Navigation is done using React Router framework.

The web app is served as a set of static files. There are some external dependencies for the map display.

================================================================================
# Source URL: https://www.traccar.org/implement-protocol/
================================================================================

# Implementing a New Protocol

This page contains information about implementing a new protocol support for Traccar server.

The project is written in Java, using gradle build system. For more details on development environment setup please check the [build documentation](/build/).

To implement a protocol you would need at least a class extending **BaseProtocol** and a class extending **BaseProtocolDecoder**. In some complicated cases you might also need a class extending **FrameDecoder**. In most cases for frame decoder you can use classes provided by the Netty framework (e.g. **LineBasedFrameDecoder**).

There are two main types of protocols: text and binary.

As an example for binary protocol you can use GT02 protocol decoder: [Gt02ProtocolDecoder](https://github.com/traccar/traccar/blob/master/src/main/java/org/traccar/protocol/Gt02ProtocolDecoder.java).

As an example for text protocol you can use GPS103 protocol decoder: [Gps103ProtocolDecoder](https://github.com/traccar/traccar/blob/master/src/main/java/org/traccar/protocol/Gps103ProtocolDecoder.java).

To enable protocol you also need to add the port to **PortConfigSuffix** or the configuration file.

To test that everything is working you can send a test message using our [hex.sh](https://github.com/traccar/traccar/blob/master/tools/hex.sh) shell script.

================================================================================
# Source URL: https://www.traccar.org/protocols/
================================================================================

# Protocols

A collection of protocol documentation for various GPS tracking devices. All of the provided protocols are supported by Traccar. For the Traccar Client format documentation please check the [OsmAnd protocol](/osmand/).

| Protocol | Documents |
| --- | --- |
| gps103 | [GPRS data protocol.xls](/protocol/5001-gps103/GPRS data protocol.xls) |
| [PROTOCOL123456 out.doc](/protocol/5001-gps103/PROTOCOL123456 out.doc) |
| tk103 | [KX402\_GPRS\_V15\_EN.pdf](/protocol/5002-tk103/KX402_GPRS_V15_EN.pdf) |
| [TK103 GPS Vehicle tracker user manual3.doc](/protocol/5002-tk103/TK103 GPS Vehicle tracker user manual3.doc) |
| gl200 | [GL200 @Tracker Air Interface Protocol\_V1.02.pdf](/protocol/5004-gl200/GL200 @Tracker Air Interface Protocol_V1.02.pdf) |
| [GT300 @Track Air Interface Protocol V4.02.pdf](/protocol/5004-gl200/GT300 @Track Air Interface Protocol V4.02.pdf) |
| [GT500MA&GT501 Interface Protocol V1.27.pdf](/protocol/5004-gl200/GT500MA&GT501 Interface Protocol V1.27.pdf) |
| [GV350M Series @Track Air Interface Protocol\_V3.11.pdf](/protocol/5004-gl200/GV350M Series @Track Air Interface Protocol_V3.11.pdf) |
| [GV55W @Track Air Interface Protocol V1.01.pdf](/protocol/5004-gl200/GV55W @Track Air Interface Protocol V1.01.pdf) |
| [p61-interface-protocol\_v119.pdf](/protocol/5004-gl200/p61-interface-protocol_v119.pdf) |
| t55 | [NMEA Reference Manual-Rev2.1-Dec07.pdf](/protocol/5005-t55/NMEA Reference Manual-Rev2.1-Dec07.pdf) |
| xexun | [COMMUNICATIONPROTOCOL.pdf](/protocol/5006-xexun/COMMUNICATIONPROTOCOL.pdf) |
| totem | [AT03 GPRS Protocol v1.0.doc](/protocol/5007-totem/AT03 GPRS Protocol v1.0.doc) |
| [AVL08GPRSProtocolv1.0.1.pdf](/protocol/5007-totem/AVL08GPRSProtocolv1.0.1.pdf) |
| enfora | [ENF0000CB001 - API Reference.pdf](/protocol/5008-enfora/ENF0000CB001 -  API Reference.pdf) |
| [GSM2228CB-\_Enfora\_Mini-MT\_Cook\_Book\_-\_Revision\_1.01.pdf](/protocol/5008-enfora/GSM2228CB-_Enfora_Mini-MT_Cook_Book_-_Revision_1.01.pdf) |
| meiligao | [Meiligao\_GPRS\_Communication\_Protocol\_GT30i\_GT60\_VT300\_VT310\_VT400\_V2.2.pdf](/protocol/5009-meiligao/Meiligao_GPRS_Communication_Protocol_GT30i_GT60_VT300_VT310_VT400_V2.2.pdf) |
| suntech | [OperationDescription\_ST215\_ST240\_STADV\_SUNTECHBR\_V103.pdf](/protocol/5011-suntech/OperationDescription_ST215_ST240_STADV_SUNTECHBR_V103.pdf) |
| progress | [Протокол Прогресс-01 v1.3.doc](/protocol/5012-progress/Протокол Прогресс-01 v1.3.doc) |
| h02 | [GPS+Tracker+Platform+Communication+Protocol-From+Winnie+HuaSunTeK-V1.0.5-2017.pdf](/protocol/5013-h02/GPS+Tracker+Platform+Communication+Protocol-From+Winnie+HuaSunTeK-V1.0.5-2017.pdf) |
| [GPS杞﹁浇璁惧鍗忚(澶╃惔鍏ㄦ枃)瑙勭▼.doc](/protocol/5013-h02/GPS杞﹁浇璁惧鍗忚(澶╃惔鍏ㄦ枃)瑙勭▼.doc) |
| [H02 protocol EN.doc](/protocol/5013-h02/H02 protocol EN.doc) |
| [ICARGPSProtocol.xlsx](/protocol/5013-h02/ICARGPSProtocol.xlsx) |
| [LKGPS protocol.pdf](/protocol/5013-h02/LKGPS protocol.pdf) |
| [document.pdf](/protocol/5013-h02/document.pdf) |
| [h02\_cantrack\_secumore.pdf](/protocol/5013-h02/h02_cantrack_secumore.pdf) |
| jt600 | [The Protocol Manual of GP6000-V1.7.pdf](/protocol/5014-jt600/The Protocol Manual of GP6000-V1.7.pdf) |
| [The protocol of JT600 V1.8.pdf](/protocol/5014-jt600/The protocol of JT600 V1.8.pdf) |
| jt808 | [Communication Protocol of VL502 V1.0.7\_eng.pdf](/protocol/5015-jt808/Communication Protocol of VL502  V1.0.7_eng.pdf) |
| [JT705 protocol 2020.pdf](/protocol/5015-jt808/JT705 protocol 2020.pdf) |
| [Smart GPS Tracker Communication Protocol(Rev.1.1).docx](/protocol/5015-jt808/Smart GPS Tracker Communication Protocol(Rev.1.1).docx) |
| v680 | [GPRS protocol between tracker and server.pdf](/protocol/5016-v680/GPRS protocol between tracker and server.pdf) |
| tr20 | [protocol v3.3.8 for TR20.doc](/protocol/5018-tr20/protocol v3.3.8 for TR20.doc) |
| navis | [ver3.5\_Текстовый\_и\_бинарный\_протоколы\_передачи\_данных\_\_для\_систем\_элемент\_и\_сигнал.pdf](/protocol/5019-navis/ver3.5_Текстовый_и_бинарный_протоколы_передачи_данных__для_систем_элемент_и_сигнал.pdf) |
| meitrack | [MEITRACK\_GPRS\_PROTOCOL\_V1.6.pdf](/protocol/5020-meitrack/MEITRACK_GPRS_PROTOCOL_V1.6.pdf) |
| skypatrol | [TT8750+AN004 - SkyPatrol Position Report Decoding Rev 1\_0.pdf](/protocol/5021-skypatrol/TT8750+AN004 - SkyPatrol Position Report Decoding Rev 1_0.pdf) |
| gt02 | [GT02 protocol \_en\_neutral.pdf](/protocol/5022-gt02/GT02 protocol _en_neutral.pdf) |
| gt06 | [20170830 Communication Protocol of JC100.pdf](/protocol/5023-gt06/20170830 Communication Protocol of JC100.pdf) |
| [AZ735 Device Bike.pdf](/protocol/5023-gt06/AZ735 Device Bike.pdf) |
| [Benway Protocol.pdf](/protocol/5023-gt06/Benway Protocol.pdf) |
| [FM08ABCGT06protocol.pdf](/protocol/5023-gt06/FM08ABCGT06protocol.pdf) |
| [GK310 Communication Protocol.docx](/protocol/5023-gt06/GK310 Communication Protocol.docx) |
| [GT06E with magnetic card reader.pdf](/protocol/5023-gt06/GT06E with magnetic card reader.pdf) |
| [GT06\_GPS\_Tracker\_Communication\_Protocol\_v1.8.1.pdf](/protocol/5023-gt06/GT06_GPS_Tracker_Communication_Protocol_v1.8.1.pdf) |
| [GT07\_GPS\_Tracker\_Communication\_Protocol\_v1.4neutral.pdf](/protocol/5023-gt06/GT07_GPS_Tracker_Communication_Protocol_v1.4neutral.pdf) |
| [JI09 Communication Protocol.pdf](/protocol/5023-gt06/JI09 Communication Protocol.pdf) |
| [JM-LL301 GPS Tracker Communication Protocol\_v1.0\_20210823.pdf](/protocol/5023-gt06/JM-LL301 GPS Tracker Communication Protocol_v1.0_20210823.pdf) |
| [Protocol.S5E.S5.docx](/protocol/5023-gt06/Protocol.S5E.S5.docx) |
| [Protocolo SL42 SL44 SL48.pdf](/protocol/5023-gt06/Protocolo SL42 SL44 SL48.pdf) |
| [S20WanWayProtocol2021.03.22V21.pdf](/protocol/5023-gt06/S20WanWayProtocol2021.03.22V21.pdf) |
| [Space10X-wifi-Protocol.xls](/protocol/5023-gt06/Space10X-wifi-Protocol.xls) |
| [WD-209\_GT06 Protocol.pdf](/protocol/5023-gt06/WD-209_GT06 Protocol.pdf) |
| [X1-GPRS Communication Protocol V1.1.pdf](/protocol/5023-gt06/X1-GPRS Communication Protocol V1.1.pdf) |
| [ZhongXun Topin Locator Communication Protocol.pdf](/protocol/5023-gt06/ZhongXun Topin Locator Communication Protocol.pdf) |
| megastek | [369S GPRS Communication Protocol-V1.00.040113.pdf](/protocol/5024-megastek/369S GPRS Communication Protocol-V1.00.040113.pdf) |
| [Communication Protocol V-A 1.1.pdf](/protocol/5024-megastek/Communication Protocol V-A 1.1.pdf) |
| [GPRS Uploading Data.pdf](/protocol/5024-megastek/GPRS Uploading Data.pdf) |
| navigil | [Navigil\_application\_protocol\_R8.003.pdf](/protocol/5025-navigil/Navigil_application_protocol_R8.003.pdf) |
| gpsgate | [GpsGateServerProtocol200.pdf](/protocol/5026-gpsgate/GpsGateServerProtocol200.pdf) |
| teltonika | [FMXXXX Protocols v2\_7.pdf](/protocol/5027-teltonika/FMXXXX Protocols v2_7.pdf) |
| [GH3000 Data protocol v1.08.pdf](/protocol/5027-teltonika/GH3000 Data protocol v1.08.pdf) |
| mta6 | [МТА6.zip](/protocol/5028-mta6/МТА6.zip) |
| [Тип объекта МТА6CAN + CANlog (M330\_440) + 6F v3.2.doc](/protocol/5028-mta6/Тип объекта МТА6CAN + CANlog (M330_440) + 6F v3.2.doc) |
| tzone | [TZ-DataProtocol-School Bus AVL19-v1.5.en.xls](/protocol/5029-tzone/TZ-DataProtocol-School Bus AVL19-v1.5.en.xls) |
| tlt2h | [TLT-2H protocol.pdf](/protocol/5030-tlt2h/TLT-2H protocol.pdf) |
| taip | [DCT\_SYRUS\_LISTENER\_RECOMMENDATIONS.pdf](/protocol/5031-taip/DCT_SYRUS_LISTENER_RECOMMENDATIONS.pdf) |
| wondex | [Part of VT300 Protocol Document V101 (20110310).pdf](/protocol/5032-wondex/Part of VT300 Protocol Document V101 (20110310).pdf) |
| cellocator | [Cellocator-Wireless-Communication-Protocol.pdf](/protocol/5033-cellocator/Cellocator-Wireless-Communication-Protocol.pdf) |
| galileo | [UserManual0129.pdf](/protocol/5034-galileo/UserManual0129.pdf) |
| [UserManual\_En\_Lite\_0192.pdf](/protocol/5034-galileo/UserManual_En_Lite_0192.pdf) |
| ywt | [YWT-Protocol(vehicle device).pdf](/protocol/5035-ywt/YWT-Protocol(vehicle device).pdf) |
| tk102 | [TK-102B communication protocol.pdf](/protocol/5036-tk102/TK-102B communication protocol.pdf) |
| [TK102项目服务器协议\_121224.doc](/protocol/5036-tk102/TK102项目服务器协议_121224.doc) |
| intellitrac | [IntelliTrac X1 Plus Protocol v1 05.pdf](/protocol/5037-intellitrac/IntelliTrac X1 Plus Protocol v1 05.pdf) |
| [intellitrac\_x1\_protocol\_v1\_10.pdf](/protocol/5037-intellitrac/intellitrac_x1_protocol_v1_10.pdf) |
| gpsmta | [gpsmta.html](/protocol/5038-gpsmta/gpsmta.html) |
| wialon | [Wialon IPS.pdf](/protocol/5039-wialon/Wialon IPS.pdf) |
| [Wialon IPS\_en.pdf](/protocol/5039-wialon/Wialon IPS_en.pdf) |
| carscop | [Communication Protocol for Carscop CCTR-800 GPS\_en.doc](/protocol/5040-carscop/Communication Protocol for Carscop CCTR-800 GPS_en.doc) |
| apel | [Tracker Message Service. Version 0.10.pdf](/protocol/5041-apel/Tracker Message Service. Version 0.10.pdf) |
| globalsat | [GTR-128\_GTR-129 Development Document Version V0.4\_20120921.pdf](/protocol/5043-globalsat/GTR-128_GTR-129 Development Document Version V0.4_20120921.pdf) |
| [TR-206 Development Document Version 0.7\_20110111.pdf](/protocol/5043-globalsat/TR-206 Development Document Version 0.7_20110111.pdf) |
| atrack | [AT1 Track Manual Completo.pdf](/protocol/5044-atrack/AT1 Track Manual Completo.pdf) |
| pt3000 | [PT3000.pdf](/protocol/5045-pt3000/PT3000.pdf) |
| ruptela | [server protocol.ods](/protocol/5046-ruptela/server protocol.ods) |
| topflytech | [TOPFLYTECH T880X Protocol V5 6.xlsx](/protocol/5047-topflytech/TOPFLYTECH T880X Protocol V5 6.xlsx) |
| laipac | [Bracelet\_Protocol\_Syncronization.pdf](/protocol/5048-laipac/Bracelet_Protocol_Syncronization.pdf) |
| aplicom | [S100300\_Aplicom\_D\_protocol.pdf](/protocol/5049-aplicom/S100300_Aplicom_D_protocol.pdf) |
| gotop | [GPRS-protocol-V-5.0-TL201-TL206-VT1081.pdf](/protocol/5050-gotop/GPRS-protocol-V-5.0-TL201-TL206-VT1081.pdf) |
| gator | [Gator vehicle tracker communication protocol.doc](/protocol/5052-gator/Gator vehicle tracker communication protocol.doc) |
| [S208 S228 protocolENGLISH-st.doc](/protocol/5052-gator/S208 S228 protocolENGLISH-st.doc) |
| noran | [GPRS-Vehicle-Tracker-Protocol-NR006-NR024.pdf](/protocol/5053-noran/GPRS-Vehicle-Tracker-Protocol-NR006-NR024.pdf) |
| [UM02&UT04\_GPRS Vehicle Tracker Protocol.doc](/protocol/5053-noran/UM02&UT04_GPRS Vehicle Tracker Protocol.doc) |
| m2m | [Протокол.xls](/protocol/5054-m2m/Протокол.xls) |
| easytrack | [Easy-Track communication protocol.doc](/protocol/5056-easytrack/Easy-Track communication protocol.doc) |
| gpsmarker | [GPS.pdf](/protocol/5057-gpsmarker/GPS.pdf) |
| khd | [KHDtrack Standard Protocol V2.pdf](/protocol/5058-khd/KHDtrack Standard Protocol V2.pdf) |
| stl060 | [STL060.doc](/protocol/5060-stl060/STL060.doc) |
| cartrack | [ITP AVL Command List GPRS.pdf](/protocol/5061-cartrack/ITP AVL Command List GPRS.pdf) |
| minifinder | [EV601FullProtocols.doc](/protocol/5062-minifinder/EV601FullProtocols.doc) |
| [MiniFinder\_Pico\_Protocol.pdf](/protocol/5062-minifinder/MiniFinder_Pico_Protocol.pdf) |
| haicom | [GPRS.xls](/protocol/5063-haicom/GPRS.xls) |
| eelink | [EELINK Device Protocol V2.0 (A4).pdf](/protocol/5064-eelink/EELINK Device Protocol V2.0 (A4).pdf) |
| [EELINK\_Terminal\_Communication\_Protocol\_V1.8.4\_EN.pdf](/protocol/5064-eelink/EELINK_Terminal_Communication_Protocol_V1.8.4_EN.pdf) |
| box | [BOX-tracker Protocol v1 5.pdf](/protocol/5065-box/BOX-tracker Protocol v1 5.pdf) |
| trackbox | [protocol.pdf](/protocol/5068-trackbox/protocol.pdf) |
| orion | [ORION Binary Data Format V7\_130328.pdf](/protocol/5070-orion/ORION Binary Data Format V7_130328.pdf) |
| riti | [ENG\_PRL11\_SLS-00886 Air Communication Protocol\_1\_0\_5 # 2013-02-20.pdf](/protocol/5071-riti/ENG_PRL11_SLS-00886 Air Communication Protocol_1_0_5 # 2013-02-20.pdf) |
| ulbotech | [Ulbotech Communication Protocol V1.2.pdf](/protocol/5072-ulbotech/Ulbotech Communication Protocol V1.2.pdf) |
| tramigo | [T23\_GPRSS\_GPRS\_Specification\_ENG\_v0.94.pdf](/protocol/5073-tramigo/T23_GPRSS_GPRS_Specification_ENG_v0.94.pdf) |
| tr900 | [PROTOCOLO NEO1\_2\_TR900\_21042014.pdf](/protocol/5074-tr900/PROTOCOLO NEO1_2_TR900_21042014.pdf) |
| ardi01 | [Message\_Parameter\_Format.doc](/protocol/5075-ardi01/Message_Parameter_Format.doc) |
| autofon | [GPRS\_M10\_M11.doc](/protocol/5077-autofon/GPRS_M10_M11.doc) |
| [Прокол АвтоФон Маяк версий 5.х и выше.xls](/protocol/5077-autofon/Прокол АвтоФон Маяк версий  5.х и выше.xls) |
| bce | [DT7Version3 (Data structures).pdf](/protocol/5080-bce/DT7Version3 (Data structures).pdf) |
| [SimpleBCEdeviceprotocol.pdf](/protocol/5080-bce/SimpleBCEdeviceprotocol.pdf) |
| xirgo | [XT-2150 X1z1-1131B9 2015.pdf](/protocol/5081-xirgo/XT-2150 X1z1-1131B9 2015.pdf) |
| castel | [NEW OBD SMART Communication Protocol\_CB212-C1005 Rev. 4.25\_.pdf](/protocol/5086-castel/NEW OBD SMART Communication Protocol_CB212-C1005 Rev. 4.25_.pdf) |
| [OBD Smart Communication Samples V4.28.pdf](/protocol/5086-castel/OBD Smart Communication Samples V4.28.pdf) |
| mxt | [MXT-1XX Protocol Standard.pdf](/protocol/5087-mxt/MXT-1XX Protocol Standard.pdf) |
| cityeasy | [Protocol.pdf](/protocol/5088-cityeasy/Protocol.pdf) |
| adm | [protocol\_ADM\_1.06.pdf](/protocol/5092-adm/protocol_ADM_1.06.pdf) |
| watch | [Communication Protocol.doc](/protocol/5093-watch/Communication Protocol.doc) |
| [比电协议V1.8（20160926）.doc](/protocol/5093-watch/比电协议V1.8（20160926）.doc) |
| t800x | [TOPFLYTECH T880X Protocol V6.2.xlsx](/protocol/5094-t800x/TOPFLYTECH T880X Protocol V6.2.xlsx) |
| kenji | [KJ-8501\_Protocol-061107.pdf](/protocol/5102-kenji/KJ-8501_Protocol-061107.pdf) |
| huasheng | [OBD Device Protocol\_V1.1.4\_160526.docx](/protocol/5111-huasheng/OBD Device Protocol_V1.1.4_160526.docx) |
| fifotrack | [fifotrack A03 GPRS Protocol.pdf](/protocol/5124-fifotrack/fifotrack A03 GPRS Protocol.pdf) |
| globalstar | [SmartOneC\_V\_2.1.x.pdf](/protocol/5185-globalstar/SmartOneC_V_2.1.x.pdf) |
| [SmartOne\_C\_User\_Manual\_v2.0\_Rev\_C.pdf](/protocol/5185-globalstar/SmartOne_C_User_Manual_v2.0_Rev_C.pdf) |
| minifinder2 | [Eview Pet Tracker Communication Protocol V20230506.pdf](/protocol/5187-minifinder2/Eview Pet Tracker Communication Protocol V20230506.pdf) |
| mobilogix | [EI\_2021\_0022\_\_MOBILOGIX\_\_MT2000\_PROTOCOL\_OVER\_THE\_AIR\_\_27\_09\_2021\_\_V1\_3\_8.pdf](/protocol/5216-mobilogix/EI_2021_0022__MOBILOGIX__MT2000_PROTOCOL_OVER_THE_AIR__27_09_2021__V1_3_8.pdf) |
| startek | [iStartekGPSTrackerCommunicationProtocolv1.32.pdf](/protocol/5222-startek/iStartekGPSTrackerCommunicationProtocolv1.32.pdf) |
| thinkpower | [Tracker Protocol V1.3.pdf](/protocol/5228-thinkpower/Tracker Protocol V1.3.pdf) |
| xexun2 | [XEXUN Devices and Server Data Protocol for GPS tracker 20210520.pdf](/protocol/5233-xexun2/XEXUN Devices and Server Data Protocol for GPS tracker 20210520.pdf) |
| t622iridium | [MEITRACK\_T622G-F9\_Iridium\_Protocol\_V1.0--20230630.pdf](/protocol/5248-t622iridium/MEITRACK_T622G-F9_Iridium_Protocol_V1.0--20230630.pdf) |
| nto | [NTOdeviceprotocol.docx](/protocol/5250-nto/NTOdeviceprotocol.docx) |
| ramac | [RAMAC MULTIEVENT CALLBACK.pdf](/protocol/5251-ramac/RAMAC MULTIEVENT CALLBACK.pdf) |
| kotonlink | [LoRa Cattle Tracker Payload protocol\_V2.0\_Eng[1].pdf](/protocol/5256-kotonlink/LoRa Cattle Tracker Payload protocol_V2.0_Eng[1].pdf) |
| jmak | [J\_R11\_Protocolo\_de\_comunicacao.pdf](/protocol/5259-jmak/J_R11_Protocolo_de_comunicacao.pdf) |
| [J\_R12\_Protocolo\_de\_comunicacao.pdf](/protocol/5259-jmak/J_R12_Protocolo_de_comunicacao.pdf) |
| r16h | [R16H Ankle Bracelet Protocol Document.pdf](/protocol/5266-r16h/R16H Ankle Bracelet Protocol Document.pdf) |