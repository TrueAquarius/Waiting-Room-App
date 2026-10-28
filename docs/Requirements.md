# Waiting Room Application

The **Waiting Room Application** manages queues in waiting rooms (for example in a doctor's office). It issues new *Waiting Numbers* for arriving *Visitors*, it displays the current queue on a screen in the waiting room and it allows *Agents* (for example doctors) to call the next visitors to a *Service Point* (for example a treatment room in a doctor's office).

The application can manage multiple Waiting Rooms.

On a high level, it consists of the following main screens:

- **Pick Number Screen** is a single page which runs in terminals in a waiting room. Its purpose is to generate a new *Waiting Number* for an arriving *Visitor*.

- **Waiting Queue Screen** is a single page which runs on a screen in a waiting room. It shows the current queue so that the *Visitors* know when it is their turn and which *Service Point* to go.

- **Service Point Screen** is a single page which runs at each *Service Point*. It allows an *Agent* to call the next *Visitor* to that *Service Point*.


## Definitions

| Term | Definition |
|------|-----------|
| *Waiting Number* | A unique 3-digit sequence number which is issued for an arriving *Visitor*. It counts from 001 to 999, then it re-starts at 001. Number 000 is never issued to avoid confusion. The counter is **per Waiting Room** and **persisted durably**; a server restart resumes counting from the last issued number. Because the cycle (001–999) is long relative to a waiting room's daily throughput, roll-over collisions (a wrapped number still being in the queue) are accepted as out of scope and ignored. |
| *Visitor* | A person who visits the waiting room (for example a patient in a doctor's office). |
| *Agent* | A person who processes visitors' requests at a *Service Point* (for example a doctor in a doctor's office working in a certain treatment room). |
| *Service Point* | A service point uniquely identified by a *ServicePointID*. For example a treatment room in a doctor's office. |
| *ServicePointID* | A unique technical identifier for a *Service Point*. Example: "TR3" |
| *ServicePointName* | A human readable name of a *Service Point*. Example: "Treatment Room 3" |
| *Waiting Room* | A Waiting Room. Has a name and a unique *WaitingRoomID*. |
| *WaitingRoomID* | A unique technical identifier of a *Waiting Room*. Example: "WR1" |
| *WaitingRoomName* | A human readable name of a *Waiting Room*. Example: "Waiting Room North" |


## Configuration

All configuration is held in a single configuration file on the server, in YAML format (JSON is accepted as a fallback). The configuration defines, per Waiting Room:

- `id` (*WaitingRoomID*)
- `name` (display name)
- `logo` (path or URL to the logo image; default applies to all rooms if not specified)
- `sound` (path or URL to the call-sound file; default applies to all rooms if not specified)
- `timeFormat` (time format string for the footer; default `HH:mm` = 24-hour `hh:mm`)
- `language` / locale (for localized UI strings; per Waiting Room)
- `servicePoints`: list of `id` (*ServicePointID*) and `name` (*ServicePointName*)

Global configuration:

- default `logo` and `sound` (used by any Waiting Room that does not specify its own)
- default `timeFormat`
- default `language`

Configuration is loaded at server start. (Reloading without a restart is out of scope unless trivial.)


## Screens

### Screen Layout: Header, Content Area, Footer

All screens shall show a header, a content area and a footer.

The header shall have a logo in the upper left corner, the Service Point Name in the center (if applicable) and the Waiting Room Name in the right corner (if applicable).

The content area shall vertically span between header and footer. Together (header, footer, content area) shall fill the entire available screen space. No scrolling allowed.

The footer shows the current time in the right corner.

```
+--------------------------------------------------------+
| {Logo}    {Service Point Name}     {Waiting Room Name} |
+--------------------------------------------------------+
|                                                        |
|                                                        |
|                                                        |
|                       {Content Area}                   |
|                                                        |
|                                                        |
|                                                        |
+--------------------------------------------------------+
|                                   {Current Time hh:mm} |
+--------------------------------------------------------+
```


### Home Screen

The **Home Screen** is the entry point to the application. It contains hyperlinks to all available screens.

The following schematic shows the **Home Screen** layout for a scenario with two Waiting Rooms named "Waiting Room 1" (with *Service Points* named "Counter 1", "Counter 2", "Counter 3") and "Waiting Room 2" (with *Service Points* named "Treatment 1", "Treatment 2").

```
+--------------------------------------------------------+
| {Logo}                                                 |
+--------------------------------------------------------+
|                                                        |
|  Waiting Room 1:  Pick | Queue                         |
|                   Counter 1 | Counter 2 | Counter 3    |
|                                                        |
|  Waiting Room 2:  Pick | Queue                         |
|                   Treatment 1 | Treatment 2           |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+
```

"Pick" and "Queue" are hyperlinks to the corresponding **Pick Number Screen** and **Waiting Queue Screen**.

"Counter x" and "Treatment y" are hyperlinks to the corresponding **Service Point Screens**. The *Service Points* of a Waiting Room are listed **horizontally**.


### Pick Number Screen

The **Pick Number Screen** is a single page which runs in terminals in a waiting room. Its purpose is to generate a new *Waiting Number* for an arriving *Visitor*.

It contains a button *New Number*.

```
+--------------------------------------------------------+
| {Logo}                             {Waiting Room Name} |
+--------------------------------------------------------+
|                                                        |
|     Please obtain your personal waiting number here    |
|                                                        |
|                   +------------+                       |
|                   | New Number |                       |
|                   +------------+                       |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+
```

Upon pressing that button, a new waiting number is issued and shown on the screen for a given period of time (default 5 seconds). Example: "Your waiting number is 225".

```
+--------------------------------------------------------+
| {Logo}                             {Waiting Room Name} |
+--------------------------------------------------------+
|                                                        |
|              Your personal waiting number is           |
|                                                        |
|                         225                            |
|                                                        |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+
```

After the display period (default 5 seconds) the number **fades out** over the dissolve time (default 2 seconds). Thereafter, the button *New Number* appears again for the next arriving visitor.

If a *Visitos* clicks the screen again during presentation time of dissolve time, the cycle starts new, giving the *Visitor* more time to not down the number.

There may be several **Pick Number Screens** running in the same Waiting Room. The application guarantees that concurrent Pick terminals in the same Waiting Room never issue the same number to different visitors (atomic counter increment on the server).

If the server is unreachable when the button is pressed, the **Connection Lost** display mode is shown; no number is issued.


### Waiting Queue Screen

The **Waiting Queue Screen** is a single page which runs on a screen in a waiting room. It shows the current queue so that the *Visitors* know when it is their turn and which *Service Point* to go.

There may be several **Waiting Queue Screens** running in the same Waiting Room.

The content area shows the current waiting queue on the left side (latest arrival at the bottom) and the *Visitors* currently being serviced on the right side (newest on top, ordered by call time).

```
+--------------------------------------------------------+
| {Logo}                             {Waiting Room Name} |
+--------------------------------------------------------+
|                                                        |
|  Waiting:                       Being serviced:        |
|  225                            222     Treatment 2    |
|  226                            221     Treatment 1    |
|  227                            220     Treatment 3    |
|  228                                                   |
|  229                                                   |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+
```

The **Waiting Queue Screen** is view-only. There is no way to interact.

The screen refreshes automatically as the queue status changes (real-time update from the server).

It plays the Waiting Room's configured sound when a *Visitor* is called to a *Service Point*.

If the list of waiting numbers or the list of being-serviced entries exceeds the available space, the text size **shrinks** so that all entries remain visible without scrolling.


### Service Point Screen

The **Service Point Screen** is a single page which runs at each *Service Point*. It allows an *Agent* to call the next *Visitor* to that *Service Point*.

```
+--------------------------------------------------------+
| {Logo}    {Service Point Name}     {Waiting Room Name} |
+--------------------------------------------------------+
|                  +--------+                            |
|  Waiting:        |  Next  |     Being serviced:  222   |
|  225             +--------+                            |
|  226                                                   |
|  227             +--------+                            |
|  228             | Repeat |                            |
|  229             +--------+                            |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+
```

The screen is operated by an *Agent*. The *Agent* clicks the button **Next** to call the next *Visitor*.

- **Next** calls the **oldest** waiting number in the Waiting Room's queue. All *Service Points* of a *Waiting Room* draw from the **same queue**.
- When **Next** is clicked while a visitor is still being serviced at this *Service Point*, the previous being-serviced visitor is **replaced** by the newly called one (i.e. being-serviced ends when the Agent clicks Next).
- **Repeat** replays the Waiting Room's call sound for the currently being-serviced visitor at this *Service Point*. **Repeat** has a cooldown (default 2 seconds) to prevent spamming.
- If a *Visitor* does not show up, the *Agent* simply clicks **Next** (no-show handling: the missing visitor is dropped from "Being serviced" without re-queue).
- Re-queue of an already-called visitor is **not** supported.

If no visitor is waiting and/or nothing is currently being serviced, the corresponding list is **empty**.

When the queue is empty and the first visitor arrives, the Waiting Room's configured sound shall play. The sound shall not play is there are already people in the queue.

If the list of waiting numbers or the being-serviced entry exceeds the available space, the text size **shrinks** so that all entries remain visible without scrolling.


### Connection Lost (display mode)

If the web client loses the connection to the server, a **Connection Lost** display mode is shown. This is **not a separate screen** — it is a display mode of whichever screen is currently open (header and footer may remain visible; the content area is replaced by the message below). The client **auto-recovers**: as soon as the connection is back, the original screen content returns and live updates resume.

```
+--------------------------------------------------------+
|                       {Header}                         |
+--------------------------------------------------------+
|                                                        |
|                                                        |
|                                                        |
|                    CONNECTION LOST                     |
|                                                        |
|                                                        |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+
```


### Paths

| Path | Screen |
|------|--------|
| `/` | Home Screen |
| `/pick/{WaitingRoomID}/` | Pick Number Screen |
| `/queue/{WaitingRoomID}/` | Waiting Queue Screen |
| `/service/{WaitingRoomID}/{ServicePointID}/` | Service Point Screen |

The **Connection Lost** display mode is not a path; it overlays whichever screen is currently open.


## Behavior Summary

### Waiting Number lifecycle
- 3-digit, per Waiting Room, persisted durably, restart-safe.
- 001–999, then restart at 001; 000 is never issued.
- Roll-over collisions are out of scope (accepted as highly unlikely).
- Atomic issuance guarantees uniqueness across concurrent Pick terminals in the same Waiting Room.

### "Being serviced" lifecycle
- **Starts** when an Agent calls a Visitor (clicks **Next**).
- **Ends** when the Agent at that *Service Point* clicks **Next** again (the visitor is replaced by the next one).
- Newest on top, ordered by call time.

### Repeat
- Replays the Waiting Room's call sound for the currently being-serviced visitor at this *Service Point*.
- Cooldown: default 2 seconds.

### No-show
- Handled by the Agent clicking **Next**; the missing visitor is dropped from "Being serviced".
- Re-queue is not supported.


## Non-functional Requirements

### Scale (sizing target)
- Up to **10 Waiting Rooms** per server.
- Up to **10 Service Points** per Waiting Room.
- Average **2 minutes** between an Agent clicking **Next** (per Service Point).
- Several Pick Number Screens and Waiting Queue Screens may run concurrently in the same Waiting Room.

### Deployment
- One central web application (server) can handle several Waiting Rooms.
- All screens run in closed terminals in kiosk mode. It is assumed that the terminals are configured to autostart the appropriate screen.
- No offline mode: when the connection to the server is lost, the **Connection Lost** display mode is shown; no number is issued and no actions are accepted until the connection is back.

### Authentication & Security
- No authentication or security is required. (All screens are open; deployment is assumed to be on a trusted network.)

### Auditing
- No audit history is required.

### Browsers
- Target browsers: **Chrome** and **Edge** (latest stable).

### Input
- Both **touch** and **mouse** input shall work.

### Internationalization
- UI strings shall be **localized**, **per Waiting Room** (configured in the configuration file).
- The time format is configurable per Waiting Room; default is `hh:mm` (24-hour).

### Sound
- A call sound is played on the **Waiting Queue Screen** when a Visitor is called to a Service Point, and replayed on **Repeat**.
- The sound is **specific to each Waiting Room** (configured in the configuration file). Default: all Waiting Rooms use the same sound.
- Default sound: a free, royalty-free sound sourced from the internet (e.g. Mixkit / Freesound, CC0). *(Concrete file to be picked during implementation.)*

### Logo
- The logo is **specific to each Waiting Room** (configured in the configuration file). Default: all Waiting Rooms use the same logo.

### Display overflow
- When a list (waiting queue or being-serviced) does not fit on the screen, the text size **shrinks** so that all entries remain visible without scrolling.


## High Level Architecture

The **Waiting Room Application** will be implemented as a central web application.

All screens will run in closed terminals. It is assumed that the terminals are configured to autostart the appropriate screen.

The technology stack is defined in a separate document: [TechStack.md](TechStack.md).
