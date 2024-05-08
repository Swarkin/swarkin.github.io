# making a discord bot in godot 4

> swarkin, may 9 2024

**as stupid as that idea sounds, it already is in
a semi-usable state.**

while i was creating a discord bot in python for a friend's server, a thought came to my head: "how difficult would it be to actually code a discord bot from scratch?"

ive immediately delved into the discord docs and started reading for quite a bit. at first, everything was very confusing as a bunch of important aspects are spread around many pages, but after a short while i got the hang of it, finding an extremely helpful page about [gateway communication](https://discord.com/developers/docs/topics/gateway) in the process which was a perfect starting point.

## connecting to the gateway

in order to be notified about various events like messages or commands, it is needed to do a http request to the [/gateway/bot](https://discord.com/developers/docs/topics/gateway#get-gateway-bot) endpoint, returning a valid websocket url to connect to. with this information, we can use godot's websocketpeer class to connect.

upon connection we receive a [hello event](https://discord.com/developers/docs/topics/gateway#hello-event) which contains the requested interval of [heartbeat](https://discord.com/developers/docs/topics/gateway#heartbeat-interval) events that have to be sent back to discord in order to keep the connection alive. this step is quite trivial as it can be done by just using a timer node.

to complete the handshake we must now send an [identify](https://discord.com/developers/docs/topics/gateway#identifying) event to discord which contains our bot token as well as its [intents](https://discord.com/developers/docs/topics/gateway#gateway-intents) integer, indicating the events we want to receive.

discord now sends the [ready](https://discord.com/developers/docs/topics/gateway#ready-event) event meaning everything was successful. with it comes some metadata about the bot such as partial guild objects, the bot's user object, an url to reconnect for recovering from connection issues (and receiving any missed events) among some other data. following this are a bunch of guild create events that contain additonal info about the guilds that the bot is in.

this is the point where the initial connection is done and we start receiving all events based on the intents we sent during the handshake.

## receiving events

all packets discord sends have an [opcode](https://discord.com/developers/docs/topics/opcodes-and-status-codes) to identify its type. if this code is equal to the dispatch opcode, we start parsing it to find out what kind of event we just received and extract its data.

ive created a dataclass for each currently implemented event that takes in the json dictionary in its constructor and parses it.

replying to command callbacks or incoming messages is out of scope for this post and involves using discord's rest api, which is not something ive dealt with *yet*.

## the interface

currently i have two ideas on how to design the interface between my "framework" and its user.

i will most likely end up implementing both approaches, as the first example below can be used as a base to create the second example later on.

### exposing signals for command callbacks and received events

this approach is much simpler to get working initially, static typing will work better, but is less intuitive overall. the only extra node required is the user's script to connect all needed callbacks and run logic. the node tree is barely utilized.

### using the node tree to register commands and listen to events

with this approach, adding new nodes that extend the command class are automatically detected and assigned as commands internally. adding child nodes that extend the command argument class to it will specify the arguments this command takes in.

nodes that extend an event-specific class will receive the corresponding event's information. this will involve creating a class for each event to retain the option of static typing.

_example tree visualization:_

- DiscordBot
  - Commands
    - foo
    - bar
      - baz
  - Events
    - message_create

this would register the commands "foo" and "bar", where the latter takes in an argument of the type specified in the export variable of "baz", and an event callback for message_create events is registered.

this way, it is also possible to automatically calculate the required intents integer, which is used to specify which events to receive, without the dev manually figuring it out.

i may add another level of nesting to make command groups possible. this would allow multiple commands to inherit the same prefix as well as other options from the command group node.

---

## notes

i appreciate you reading trough all of this! its the first time im making a blog-like post and i plan on roughly documenting the entire process, as well as describe some technical aspects in case it inspires you to give it a try as well.

i will update this post whenever i make notable progress. the code for this is currently private, but i plan on releasing it on github.

if you want to contact me, my user is `swark1n` on discord.
