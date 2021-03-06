# Lightstreamer - "Hello World" Tutorial - .NET Adapter

<!-- START DESCRIPTION lightstreamer-example-helloworld-adapter-dotnet -->

This project, of the "Hello World with Lightstreamer" series, will  focus on a .NET port of the Java Data Adapter illustrated in [Lightstreamer - "Hello World" Tutorial - Java Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-java). In particular, both a <b>C#</b> version and a <b>Visual Basic</b> version of the Data Adapter will be shown.
<!-- END DESCRIPTION lightstreamer-example-helloworld-adapter-dotnet -->

As example of [Clients Using This Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-dotnet#clients-using-this-adapter), you may refer to the ["Hello World" Tutorial - HTML Client](https://github.com/Weswit/Lightstreamer-example-HelloWorld-client-javascript) and view the corresponding [Live Demo](http://demos.lightstreamer.com/HelloWorld/).

## Details

First, please take a look at the previous installment [Lightstreamer - "Hello World" Tutorial - HTML Client](https://github.com/Weswit/Lightstreamer-example-HelloWorld-client-javascript), which provides some background and the general description of the application. Notice that the front-end will be exactly the same. We created a very simple HTML page that subscribes to the "greetings" item, using the "HELLOWORLD" Adapter. Now, we will replace the "HELLOWORLD" Adapter implementation based on Java with C# and Visual Basic equivalents. On the client side, nothing will change, as server-side Adapters can be transparently switched and changed, as long as they respect the same interfaces. Thanks to this decoupling provided by Lightstreamer Server, we could even do something different. For example, we could keep the Java Adapter on the server side and use Flex, instead of HTML, on the client side. Or we could use the C# Adapter on the server side and use Java, instead of HMTL or Flex, on the client side. Basically, all the combinations of languages and technologies on the client side and on the server side are supported.

Please refer to [General Concepts](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/General%20Concepts.pdf) for more details about Lightstreamer Adapters.

### .NET Interfaces

Lightstreamer Server exposes native Java Adapter interfaces. The .NET interfaces are added through the [Lightstreamer Adapter Remoting Infrastructure (**ARI**)](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/sdk_adapter_remoting_infrastructure/doc/ARI%20Protocol.pdf) and form the [Adapter Remoting Infrastructure for .NET](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/sdk_adapter_dotnet/doc/DotNet%20Adapters.pdf). 

*The Architecture of [Adapter Remoting Infrastructure for .NET](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/sdk_adapter_dotnet/doc/DotNet%20Adapters.pdf).*

![General architecture](general_architecture.png)

ARI is simply made up of two Proxy Adapters and a *Network Protocol*. The two Proxy Adapters, one implementing the Data Adapter interface and the other implementing the Metadata Adapter interface, are meant to be plugged into Lightstreamer Kernel.

Basically, the Proxy Data Adapter exposes the Data Adapter interface through TCP sockets. In other words, it offers a Network Protocol, which any remote counterpart can implement to behave as a Lightstreamer Data Adapter. This means you can write a remote Data Adapter in any language, provided that you have access to plain TCP sockets.
But if your remote Data Adapter is based on .NET, you can leverage a ready-made library that exposes a higher level *.NET interface*. So, you will simply have to implement this .NET interface. So the Proxy Data Adapter converts from a Java interface to TCP sockets, and the .NET library converts from TCP sockets to a .NET interface.

The full API references for the languages covered in this tutorial are available from [.NET API reference for Adapters](http://www.lightstreamer.com/docs/adapter_dotnet_api/index.html)

### Dig the Code

#### The C# Data Adapter

(Skip this section if you are only interested in [The Visual Basic Data Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-dotnet#the-visual-basic-data-adapter).)

The C# Data Adapter consists of two classes: the `DataAdapterLauncher` contains the application's <b>Main</b> and initializes the DataProviderServer (the provided piece of code that implements the Network Protocol); the `HelloWorldAdapter` implements the actual Adapter interface.

##### DataAdapterLauncher

The DataAdapterLauncher creates a DataProviderServer instance and assigns a HelloWorldAdapter instance to it. Then, it creates two TCP client sockets, because the Proxy Data Adapter, to which our remote .NET Adapter will connect, needs two connections (but as we said, after creating these sockets, you don't have to bother with reading and writing, as these operations are automatically handled by the DataProviderServer). Let's use TCP ports <b>6661</b> and <b>6662</b>. Assign the stream of the first socket to the RequestStream and ReplyStream properties of the DataProviderServer. Assign the stream of the second socket to the NotifyStream property of the DataProviderServer. Finally, you start the DataProviderServer.

##### HelloWorldAdapter

The HelloWorldAdapter class implements the <b>IDataProvider</b> interface (which is the .NET remote equivalent of the Java DataProvider interface).

Implement the <b>SetListener</b> method to receive a reference to the server's listener that you will use to inject events.

Then, implement the <b>Subscribe</b> method. When the "greetings" item is subscribed to by the first user, the Adapter receives that method call and starts a thread that will generate the real-time data. If more users subscribe to the "greetings" item, the Subscribe method is not called any more. When the last user unsubscribes from this item, the Adapter is notified through the <b>Unsubscribe</b> call. In this case we stop the publisher thread for that item. If a new user re-subscribes to "greetings", the Subscribe method is called again. As already mentioned in the previous installment, this approach avoids consuming processing power for items nobody is currently interested in.

The <b>Run</b> method is executed within the thread started by Subscribe. Its code is very simple. We create a Hashtable containing a message (alternating "Hello" and "World") and the current timestamp. Then we inject the Hashtable into the server through the listener. We wait for a random time between 1 and 3 seconds, and we are ready to generate a new event.

#### The Visual Basic Data Adapter

Now let's work with Visual Basic instead of C#.

It consists of two modules: the `DataAdapterLauncher`, which contains the application's <b>Main</b> and initializes the DataProviderServer (the provided piece of code that implements the Network Protocol); The `HelloWorldAdapter` that implements the actual Adapter interface.

##### DataAdapterLauncher

The `DataAdapterLauncher.vb` creates a DataProviderServer instance and assigns a HelloWorldAdapter instance (which we will define below) to it. Then, it creates two TCP client sockets, because the Proxy Data Adapter, to which our remote .NET Adapter will connect, needs two connections (but as we said, after creating these sockets, you don't have to bother with reading and writing, as these operations are automatically handled by the DataProviderServer). Let's use TCP ports <b>6661</b> and <b>6662</b>. Assign the stream of the first socket to the RequestStream and ReplyStream properties of the DataProviderServer. Assign the stream of the second socket to the NotifyStream property of the DataProviderServer. Finally, start the DataProviderServer.

##### HelloWorldAdapter

The HelloWorldAdapter class implements the <b>IDataProvider</b> interface (which is the .NET remote equivalent of the Java DataProvider interface).

Implement the <b>SetListener</b> subroutine to receive a reference to the server's listener that you will use to inject events.

Then, implement the <b>Subscribe</b> subroutine. When the "greetings" item is subscribed to by the first user, the Adapter receives that subroutine call and starts a thread that will generate the real-time data. If more users subscribe to the "greetings" item, the Subscribe subroutine is not called any more. When the last user unsubscribes from this item, the Adapter is notified through the <b>Unsubscribe</B> call. In this case we stop the publisher thread for that item. If a new user re-subscribes to "greetings", the Subscribe subroutine is called again.

The <b>Run</b> subroutine is executed within the thread started by Subscribe. Its code is very simple. We create a Hashtable containing a message (alternating "Hello" and "World") and the current timestamp. Then we inject the Hashtable into the server through the listener. We wait for a random time between 1 and 3 seconds, and we are ready to generate a new event.

#### The Adapter Set Configuration

For this demo we just use a Proxy Data Adapter, while instead, as Metadata Adapter, we use a default one: the [LiteralBasedProvider](https://github.com/Weswit/Lightstreamer-example-ReusableMetadata-adapter-java)).
This Adapter pair will be referenced by the clients as "**PROXY_HELLOWORLD**".
The `adapters.xml` file looks like:

```xml
<?xml version="1.0"?>
 
<adapters_conf id="PROXY_HELLOWORLD">
 
  <metadata_provider>
    <adapter_class>com.lightstreamer.adapters.metadata.LiteralBasedProvider</adapter_class>
  </metadata_provider>
 
  <data_provider>
    <adapter_class>com.lightstreamer.adapters.remote.data.NetworkedDataProvider</adapter_class>
    <classloader>log-enabled</classloader>
    <param name="request_reply_port">6661</param>
    <param name="notify_port">6662</param>
  </data_provider>
 
</adapters_conf>
```

## Install
If you want to install a version of this demo in your local Lightstreamer Server, follow these steps.
* Download *Lightstreamer Server* (Lightstreamer Server comes with a free non-expiring demo license for 20 connected users) from [Lightstreamer Download page](http://www.lightstreamer.com/download.htm), and install it, as explained in the `GETTING_STARTED.TXT` file in the installation home directory.
* Get the `deploy.zip` file of the [latest release](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-dotnet/releases) and unzip it
* Plug the Proxy Data Adapter into the Server: go to the `Deployment_LS` folder and copy the `ProxyHelloWorld` directory and all of its files to the `adapters` folder of your Lightstreamer Server installation.
* Alternatively you may plug the **robust** versions of the Proxy Data Adapter: go to the `Deployment_LS(robust)` folder and copy the `ProxyHelloWorld` directory and all of its files into `adapters`. The robust Proxy Data Adapter can handle the case in which a Remote Data Adapter is missing or fails, by suspending the data flow and trying to connect to a new Remote Data Adapter instance. 
* Launch Lightstreamer Server. The Server startup will complete only after a successful connection between the Proxy Adapters and the Remote Adapters.
* Launch the C# Remote .NET Adapter or the Visual Basic Remote .NET Adapter: the `adapter_csharp.exe` or the `adapter_vb.exe` files can be found under `Deployment_DotNet_Server` folder.
* Test the Adapter, launching the client listed in [Clients Using This Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-dotnet#clients-using-this-adapter).
    * In order to make the ["Hello World" Tutorial - HTML Client](https://github.com/Weswit/Lightstreamer-example-HelloWorld-client-javascript) front-end pages get data from the newly installed Adapter Set, you need to modify the front-end pages and set the required Adapter Set name to PROXY_HELLOWORLD when creating the LightstreamerClient instance. So edit the `index.htm` page of the Hello World front-end deployed under `Lightstreamer/pages/HelloWorld` and replace:<BR/>
`var client = new LightstreamerClient(null," HELLOWORLD");`<BR/>
with:<BR/>
`var client = new LightstreamerClient(null,"PROXY_HELLOWORLD");;`<BR/>
    * Open a browser window and go to: [http://localhost:8080/HelloWorld/]()

## Build

### Build The C# Data Adapter

To build your own version of `adapter_csharp.exe`, instead of using the one provided in the `deploy.zip` file from the [Install](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-dotnet#install) section above, follow these steps.
* Download this project.
* Create a new C# project (we used Microsoft's [Visual C# Express Edition](http://www.microsoft.com/express/)): 
from the "New Project..." wizard, choose the "Console Application" template and use "adapter_csharp" as project name
* From the "Solution Explorer", delete the default `Program.cs`, 
* Get the Lightstreamer .NET Adapter Server library `DotNetAdapter_N2.dll` from the `DOCS-SDKs/sdk_adapter_dotnet/lib` folder of the latest [Lightstreamer 6.0 (Beta)](http://download.lightstreamer.com/#next) distribution, and copy it into the `lib` directory.
* Get the Log4net library `log4net.dll` file from the `DOCS-SDKs/sdk_adapter_dotnet/bin` folder of the latest [Lightstreamer 6.0 (Beta)](http://download.lightstreamer.com/#next) distribution, and copy them into the `lib` directory.
* Add a reference to the Lightstreamer .NET library and the Log4net library : go to the "Browse" tab of the "Add Reference" dialog and point to the `DotNetAdapter_N2.dll` and the `log4net.dll` files in the `lib` folder. 
* Add the `DataAdapterLauncher.cs` and the `HelloWorld.cs` files from the "Add -> Exixting Item" dialog. 
* Build the `adapter_csharp.exe` file: from the Build menu, choose "Build Solution".

### Build The Visual Basic Data Adapter
To build your own version of `adapter_vb.exe`, instead of using the one provided in the `deploy.zip` file from the [Install](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-dotnet#install) section above, follow these steps.
* Download this project.
* Create a new VB project (we used Microsoft's [Visual Basic Express Edition](http://www.microsoft.com/express/)):
From the "New Project..." wizard, choose the "Console Application" template and use "adapter_vb" as project name.
* From the "Solution Explorer", delete the default Module1.vb.
* Get the Lightstreamer .NET Adapter Server library `DotNetAdapter_N2.dll` from the `DOCS-SDKs/sdk_adapter_dotnet/lib` folder of the latest [Lightstreamer 6.0 (Beta)](http://download.lightstreamer.com/#next) distribution, and copy it into the `lib` directory.
* Get the Log4net library `log4net.dll` file from the `DOCS-SDKs/sdk_adapter_dotnet/bin` folder of the latest [Lightstreamer 6.0 (Beta)](http://download.lightstreamer.com/#next) distribution, and copy them into the `lib` directory.
* Add a reference to the Lightstreamer .NET library and the Log4net library : go to the "Browse" tab of the "Add Reference" dialog and point to the `DotNetAdapter_N2.dll` and the `log4net.dll` files in the `lib` folder. 
* Add the `DataAdapterLauncher.vb` and the `HelloWorld.vb` module to the project from the "Add -> Exixting Item" dialog.
* Build the `adapter_vb.exe` file: from the Build menu, choose "Build Solution".

## See Also 

* [Adapter Remoting Infrastructure
Network Protocol Specification](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/sdk_adapter_remoting_infrastructure/doc/ARI%20Protocol.pdf)
* [.NET Adapter Development](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/sdk_adapter_dotnet/doc/DotNet%20Adapters.pdf)
* [.NET Adapter API Reference](http://www.lightstreamer.com/docs/adapter_dotnet_api/frames.html)
 
### Clients Using This Adapter
<!-- START RELATED_ENTRIES -->
* [Lightstreamer - "Hello World" Tutorial - HTML Client](https://github.com/Weswit/Lightstreamer-example-HelloWorld-client-javascript)

<!-- END RELATED_ENTRIES -->

### Related Projects

* [Lightstreamer - "Hello World" Tutorial - Java Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-java)
* [Lightstreamer - "Hello World" Tutorial - TCP Socket Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-socket)
* [Lightstreamer - "Hello World" Tutorial - Node.js Adapter](https://github.com/Weswit/Lightstreamer-example-HelloWorld-adapter-node)
* [Lightstreamer - Reusable Metadata Adapters - Java Adapter](https://github.com/Weswit/Lightstreamer-example-ReusableMetadata-adapter-java)

## Lightstreamer Compatibility Notes

- Compatible with Lightstreamer .NET Adapter SDK version 1.7 or newer.

## Final Notes
For more information, please [visit our website](http://www.lightstreamer.com/) and [post to our support forums](http://forums.lightstreamer.com) any feedback or question you might have. Thanks!
