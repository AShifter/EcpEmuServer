# EcpEmuServer

Trigger webhooks from your Logitech Harmony (or other Roku ECP compatible) Universal Remote

EcpEmuServer is a small ASP.NET application which allows you to add an emulated Roku ECP (external control protocol) device to your compatible universal remote in order to trigger webhooks. It is designed as a spiritual successor to [HarmonySpan](https://github.com/ashifter/harmony-span) which provided the same functionality in a Node.js application.

Compared to HarmonySpan, EcpEmuServer aims to be;

 - Faster & Lighter (the .NET 6.0 binary shows significantly less resource usage than its self-contained HarmonySpan equivalent);
 - Easier to deploy, simple to host multiple instances across individual containers on the network (Docker Template coming soon);
 - Easier to configure, with an intuitive rule-based architecture that allows many endpoints to be bound to a single button;
 - Easier to expand and maintain - It does not depend on any third party libraries like the JS version did, and .NET is much more appropriate for this type of application.

![EcpEmuServer](https://raw.githubusercontent.com/AShifter/EcpEmuServer/main/docs/Window.png)

## Setup
Ensure your target system has a recent installation of the [.NET Core 6.0 Runtime and ASP.NET Core Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/6.0) which are available from Microsoft's website for many different platforms.

Download the latest EcpEmuServer binary [from the releases page](https://github.com/AShifter/EcpEmuServer/releases) for your platform.

Extract the zip file and run the main executable. *On Linux/macOS machines you might have to run `chmod +x ./EcpEmuServer` on the binary itself to enable the executable flag for that file.* If you are asked for permission to let EcpEmuServer through your OS' firewall, allow it for both private and public networks.

![Windows Defender Firewall Permission Dialogue](https://raw.githubusercontent.com/AShifter/EcpEmuServer/main/docs/Firewall.png)

On first startup, a simple ``rules.xml`` file will be created that should look something like this **(please close the app before trying to write to the ``rules.xml`` file)**; 

```xml
<?xml version="1.0" encoding="utf-8"?>
<ecpemuserver xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <rules>
    <rule>
      <Name>New Rule</Name>
      <Button>None</Button>
      <Action>GET</Action>
      <EndPoint>https://www.example.com/</EndPoint>
    </rule>
  </rules>
</ecpemuserver>
```

This is where you will be determining how EcpEmuServer should respond to specific button presses on your remote. The fields are defined as;

 - ``<Name>`` - the Name of your rule. Use this to keep track of all the different rules you create.
 - ``<Button>`` - the [Roku ECP button](https://developer.roku.com/docs/developer-program/debugging/external-control-api.md) bound to this rule. These are the virtual buttons that EcpEmuServer emulates, pretending to be a real Roku device. You can use any of these 15 buttons;
```
  Home
  Rev
  Fwd
  Play
  Select
  Left
  Right
  Down
  Up
  Back
  InstantReplay
  Info
  Backspace
  Search
  Enter
```
These will be available as buttons you can add to Sequences in the Harmony app, and that is how you are meant to integrate rules with your universal remote.
 - ``<Action>`` - The type of HTTP request you wish to make with this rule. For most APIs, an HTTP ``GET`` request is appropriate, but some endpoints require you to make ``POST`` requests.
 - ``<EndPoint>`` - the URL of the webhook/endpoint you want to trigger. This can be anything from an IFTTT webhook or a Discord bot to an ESP8266 board or Home Assistant instance within your local network.

Keep in mind that all requests from rules are made from the machine running EcpEmuServer, *not from the remote itself.*

Example configuration;
```xml
<?xml version="1.0" encoding="utf-8"?>
<ecpemuserver xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <rules>
    <rule>
      <Name>MainTheaterLightsOn</Name>
      <Button>Pause</Button>
      <Action>GET</Action>
      <EndPoint>https://www.fakeapi.com/trigger?key=0123456789&device=MainLightGroup&action=on</EndPoint>
    </rule>
    <rule>
      <Name>DiscordStatusUpdate</Name>
      <Button>Pause</Button>
      <Action>GET</Action>
      <EndPoint>https://www.fakediscord.com/webhook?token=j89d32jlknd0&whid=md9032m09g94</EndPoint>
    </rule>
    <rule>
      <Name>MainTheaterLightsOff</Name>
      <Button>Play</Button>
      <Action>GET</Action>
      <EndPoint>https://www.fakeapi.com/trigger?key=0123456789&device=MainLightGroup&action=off</EndPoint>
    </rule>
    <rule>
      <Name>CloseBlinds</Name>
      <Button>Play</Button>
      <Action>GET</Action>
      <EndPoint>https://www.fakeapi.com/trigger?key=0123456789&device=WindowBlinds&action=close</EndPoint>
    </rule>
  </rules>
</ecpemuserver>
```
To make changes at any time, ensure you have stopped EcpEmuServer before writing to ``rules.xml``.

Once you have configured EcpEmuServer, and have it running, you can add it to your Harmony Hub just the same as any other IP device. 
