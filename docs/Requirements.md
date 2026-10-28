# Waiting Room Application

The **Waiting Room Application** manages queues in waiting rooms (for example in a doctor's office). It issues new *Waiting Numbers* for arriving *Visitors*, it displays the current queue on a screen in the waiting room and it allows *Agents* (for example doctors) to call the next visitors to a *Service Point* (for example a treatment room in a doctor's office).

The application can manage multiple Waiting Rooms.

On a high level, it consists of the following main screens:

- **Pick Number Screen** is a single page which runs in terminals in a waiting room. Its purpose is to generate a new *Waiting Number* for an arriving *Visitor*. 

- **Waiting Queue Screen** is a single page which runs on a screen in a waiting room. It shows the current Queue so that the *Visitors* know when it is their turn and which *Service Point* to go. 

- **Service Point Screen** is a single page which runs at each *Service Point*. It allows an *Agent* to call the next *Visitor* to that *Service Point*.

## Definitions

| Term | Definition |
|------|------------|
| *Waiting Number* | A unique 3-digit sequence number which is issues for an arriving *Visitor*. It counts from 001 to 999, then it re-starts at 001. Number 000 is never issued to avoid confusion. |
| *Visitor* | A person who visites the waiting room (for example a patient in a doctor's office). |
| *Agent * | A person who processes visitors' requests at a *Service Point* (for examle a doctor in a doctor's office working in a certain treament room) |
| *Service Point* | A service point uniquely identified by a *ServicePointID*. For example a treatment room in a doctor's office |
| *ServicePointID* | A unique identified for a *Service Point* |
| *Waiting Room* | A Waiting Room. Has a name and a unique *WaitingRoomID* |
| *WaitingRoomID* | A unique identifier of a *Waiting Room*. |


## Screens

### Screen Layout: Header, Content Area, Footer

All screens shall show a header, a content area and a footer. 

The header shall have a Logo in the upper left corner, The Service Point Name in the center (if applicable) and the Waiting Room Name in the reight corner (if applicable).

The content area shall vertically span between header and footer. Together (header, footer, content area) shall fill the entire available screen scace. No scrolling allowed. 

The footer with the current time in the right corner.

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


### Home Screen

The **Home Screen** is the entry point to the application. It contains hyperlinks to all available screens.

The following schematic show the **Home Screen** layout for a scenario with two Waiting Rooms named "Waiting Room 1" (with *Service Points* named "Counter 1", "Counter 2", "Counter 3") and "Waiting Room 2" (with *Service Points* named "Treatment 1", "Treatment 2* ). 

+--------------------------------------------------------+
| {Logo}                                                 |
+--------------------------------------------------------+
|                                                        |
|  Waiting Room 1:  Pick, Queue                          |
|                   Counter 1, Counter 2, Counter 3      |
|                                                        |
|  Waiting Room 2:  Pick, Queue                          |
|                   Treatment 1, Treatment 2             |
|                                                        |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+

"Pick" and "Queue" are hyperlinks to the corresponding **Pick Number Screen** and **Wait Queue Screen**. 

"Counter x" and "Treatment y" are hyperlinks to the corresponding **Service Point Screens**


### Pick Number Screen

The **Pick Number Screen** is a single page which runs in terminals in a waiting room. Its purpose is to generate a new *Waiting Number* for an arriving *Visitor*. 

It contains a button *New Number*. 

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
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+


Upon pressing that button, a new waiting number is issued and shown on the screen for a given perod of time (default 5 seconds). Example: "Your waiting number is 225".

+--------------------------------------------------------+
| {Logo}                             {Waiting Room Name} |
+--------------------------------------------------------+
|                                                        |
|              Your personal waiting number is           |
|                                                        |
|                         225                            |
|                                                        |
|                                                        |
|                                                        |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+

Then the number slowly disappears / dissolves (default desolve time is 2 seconds). Thereafter, the button *New Number* appears again for the next arriving visitor.

There may be several **Pick Number Screens** running in the same Waiting Room.

### Waiting Queue Screen

**Waiting Queue Screen** is a single page which runs on a screen in a waiting room. It shows the current Queue so that the *Visitors* know when it is their turn and which *Service Point* to go. 

There may be several **Waining Queue Screens** running in the same Waiting Room.

The content are show the current wait queue on the left side (latest arrival at the bottom) and the currently being serviced on the right side (newest on top).

+--------------------------------------------------------+
| {Logo}                             {Waiting Room Name} |
+--------------------------------------------------------+
|                                                        |
|  Waiting:                       Being serviced:        |
|  225                            222     Treatment  2   |
|  226                            221     Treatment  1   |
|  227                            220     Treatment  3   |
|  228                                                   |
|  229                                                   |
|                                                        |
+--------------------------------------------------------+
|                      {Footer}                          |
+--------------------------------------------------------+

The **Waiting Queue Screen** is view-only. There is no way to interact.

The screen refreshes automatically as the queue status changes.

It plays a sound when a *Visior* is called to a *Service Point*.

### Service Point Screen

**Service Point Screen** is a single page which runs at each *Service Point*. It allows an *Agent* to call the next *Visitor* to that *Service Point*.

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

The screen is operated by an *Agent*. The *Agent* clicks on botton "Next" to call the next *Visitor*. 

If the *Visitor* does not arrive, the *Agent* kicks "Repeat" to call the *Visitor* again.

### Connection Lost Screen

If the web client looses the connection to the server, a **Connection Lost Screen** shall apprear.

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


### Paths

| Path | Screen |
|------|--------|
| /    | Home Screen |
| pick/{WaitingRoomID}/ | Pick Number Screen |
| queue/{WaitingRoomID}/ | Waiting Queue Screen |
| service/{WaitingRoomID}/{ServicePointID} | Service Point Screen |



## High level architecture

The **Waiting Room Application** will be implemented as central Web Application.

All Screens will run in closed terminals. It is assumed that the terminals are configured to autostart the appropriate screen.