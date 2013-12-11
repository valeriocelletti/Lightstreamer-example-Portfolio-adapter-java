# Lightstreamer - Portfolio Demo - Java SE Adapter #

This project includes the resources needed to develop the Metadata and Data Adapters for [Lighstreamer - Basic Portfolio Demo - HTML Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript#basic-portfolio-demo) and [Lighstreamer - Portfolio Demo - HTML Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript#portfolio-demo) that is pluggable into Lightstreamer Server. Please refer [here](http://www.lightstreamer.com/latest/Lightstreamer_Allegro-Presto-Vivace_5_1_Colosseo/Lightstreamer/DOCS-SDKs/General%20Concepts.pdf) for more details about Lightstreamer Adapters.<br>
The Portfolio Demos simulate portfolio management. They show a list of stocks included in a portfolio and provide a simple order entry form. Changes to portfolio contents due to new orders are displayed on the page in real time.<br>
<br>
The project is comprised of source code and a deployment example. The source code is divided into three folders.

## src_feed ##
Contains the source code for a class that simulates a portfolio manager, which generates random portfolios and accepts buy and sell operations to change portfolio contents.

## src_portfolio ##
Contains the source code for the Basic Portfolio Demo Data Adapter, a demo Adapter that handles subscription requests by attaching to the simulated portfolio manager.
It can be referred to as a basic example for Data Adapter development.

## src_metadata ##
Contains the source code for a Metadata Adapter to be associated with the Portfolio Demo Data Adapter. This Metadata Adapter inherits from `LiteralBasedProvider` in [Lightstreamer - Reusable Metadata Adapters - Java SE Adapter](https://github.com/Weswit/Lightstreamer-example-ReusableMetadata-adapter-java) and just adds a simple support for order entry by implementing the NotifyUserMessage method, in order to handle "sendMessage" requests from the Portfolio Demo client.
The communication to the Portfolio Feed Simulator, through the Portfolio Data Adapter, is handled here.<br>
It should not be used as a reference for a real case of client-originated message handling, as no guaranteed delivery and no clustering support is shown.
<br>
<br>
See the source code comments for further details.


# Build #

If you want to skip the build process of this Adapter please note that in the [deploy release](https://github.com/Weswit/Lightstreamer-example-Portfolio-adapter-java/releases) of this project you can find the "deploy.zip" file that contains a ready-made deployment resource for the Lightstreamer server. <br>
Otherwise follow these steps:

* Get the ls-adapter-interface.jar, ls-generic-adapters.jar, and log4j-1.2.15.jar files from the [latest Lightstreamer distribution](http://www.lightstreamer.com/download) and put these files into lib folder.
* Create the jars LS_portfolio_metadata_adapter.jar, LS_portfolio_feed_simulator.jar, and LS_portfolio_data_adapter.jar created by something like these commands
```sh
 >javac -source 1.7 -target 1.7 -nowarn -g -classpath compile_libs/log4j-1.2.15.jar -sourcepath src/src_feed -d tmp_classes src/src_feed/portfolio_demo/feed_simulator/Portfolio.java
 
 >jar cvf LS_portfolio_feed_simulator.jar -C tmp_classes src_feed
 
 >javac -source 1.7 -target 1.7 -nowarn -g -classpath compile_libs/log4j-1.2.15.jar;compile_libs/ls-adapter-interface/ls-adapter-interface.jar;compile_libs/ls-generic-adapters/ls-generic-adapters.jar;LS_portfolio_feed_simulator.jar -sourcepath src/src_portfolio -d tmp_classes src/src_portfolio/portfolio_demo/adapters/PortfolioDataAdapter.java
 
 >jar cvf LS_portfolio_data_adapter.jar -C tmp_classes src_portfolio
 
 >javac -source 1.7 -target 1.7 -nowarn -g -classpath compile_libs/log4j-1.2.15.jar;compile_libs/ls-adapter-interface/ls-adapter-interface.jar;compile_libs/ls-generic-adapters/ls-generic-adapters.jar;LS_portfolio_feed_simulator.jar;LS_portfolio_data_adapter.jar -sourcepath src/src_metadata -d tmp_classes src/src_metadata/portfolio_demo/adapters/PortfolioMetadataAdapter.java
 
 >jar cvf LS_portfolio_metadata_adapter.jar -C tmp_classes src_metadata
```

# Deploy #

Now you are ready to deploy the Portfolio Demo Adapter into Lighstreamer server.
After you have Downloaded and installed Lightstreamer, please go to the "adapters" folder of your Lightstreamer Server installation. You should find a "Demo" folder containing some adapters ready-made for several demo including the Portfolio ones, please note that the MetaData Adapter jar installed is a mixed one that combines the functionality of several demos. If this is not your case because you have removed the "Demo" folder or you want to install the Portfolio adapter set alone, please follow this steps to configure the Portfolio adapter properly.

You have to create a specific folder to deploy the Portfolio Adapter otherwise get the ready-made "Portfolio" deploy folder from "deploy.zip" of the [latest release](https://github.com/Weswit/Lightstreamer-example-Portfolio-adapter-java/releases) of this project and skips the next three steps.<br>

1. You have to create a new folder to deploy the portfolio adapters, let's call it "portfolio", and a "lib" folder inside it.
2. Create an "adapters.xml" file inside the "portfolio" folder and use the following content (this is an example configuration, you can modify it to your liking):

```xml
<?xml version="1.0"?>

<!-- Mandatory. Define an Adapter Set and sets its unique ID. -->
<adapters_conf id="DEMO">

    <!-- Mandatory. Define the Metadata Adapter. -->
    <metadata_provider>

        <!-- Mandatory. Java class name of the adapter. -->
        <adapter_class>portfolio_demo.adapters.PortfolioMetadataAdapter</adapter_class>

        <!-- Optional for PortfolioMetadataAdapter.
             Configuration file for the Adapter's own logging.
             Logging is managed through log4j. -->
        <param name="log_config">adapters_log_conf.xml</param>
        <param name="log_config_refresh_seconds">10</param>

        <!-- Optional, managed by the inherited LiteralBasedProvider.
             See LiteralBasedProvider javadoc. -->
        <!--
        <param name="max_bandwidth">40</param>
        <param name="max_frequency">3</param>
        <param name="buffer_size">30</param>
        <param name="distinct_snapshot_length">10</param>
        <param name="prefilter_frequency">5</param>
        <param name="allowed_users">user123,user456</param>
        -->

        <!-- Optional, managed by the inherited LiteralBasedProvider.
             See LiteralBasedProvider javadoc. -->
        <param name="item_family_1">portfolio.*</param>
        <param name="modes_for_item_family_1">COMMAND</param>

    </metadata_provider>

    <!-- Mandatory. Define the Data Adapter. -->
    <data_provider name="PORTFOLIO_ADAPTER">

        <!-- Mandatory. Java class name of the adapter. -->
        <adapter_class>portfolio_demo.adapters.PortfolioDataAdapter</adapter_class>

        <!-- Optional for PortfolioDataAdapter.
             Configuration file for the Adapter's own logging.
             Leans on the Metadata Adapter for the configuration refresh.
             Logging is managed through log4j. -->
        <param name="log_config">adapters_log_conf.xml</param>

    </data_provider>

</adapters_conf>
```
<br>
3. Copy into /portfolio/lib the jars (LS_portfolio_metadata_adapter.jar, LS_portfolio_feed_simulator.jar, and LS_portfolio_data_adapter.jar) created in the previous section.

Now your "Portfolio" folder is ready to be deployed in the Lightstreamer server, please follow these steps:

1. Make sure you have installed Lightstreamer Server, as explained in the GETTING_STARTED.TXT file in the installation home directory.
2. Make sure that Lightstreamer Server is not running.
3. Copy the "portfolio" directory and all of its files to the "adapters" subdirectory in your Lightstreamer Server installation home directory.
4. Copy the "ls-generic-adapters.jar" file from the "lib" directory of the sibling "Reusable_MetadataAdapters" SDK example to the "shared/lib" subdirectory in your Lightstreamer Server installation home directory.
5. Lightstreamer Server is now ready to be launched.

Please test your Adapter with one of the clients in the [list](https://github.com/Weswit/Lightstreamer-example-Portfolio-adapter-java#clients-using-this-adapter) below.

## Portfolio and StockList Demo Adapters together in the same Adapter Set ##

Please note that to work with fully functionality the [Lightstreamer - Portfolio Demo - HTML Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript#portfolio-demo), you have to deploy on your Lightstreamer instance the QUOTE_ADAPTER adapter too (see [Lightstreamer - Stock-List Demo - Java SE Adapter](https://github.com/Weswit/Lightstreamer-example-StockList-adapter-java)).
To allow the two adapters to coexist within the same adapter set, please follow the instructions below:

1. Create a new folder in  "<LS_HOME>/adapters" , let's call it "demo".
2. Move the "portfolio" and "Stocklist" specific folders to the "demo" one.
3. Remove the adapter.xml files from respective directories and merge them togheter in a new one into new "demo" folder. Something like this:
```xml 
<?xml version="1.0"?>
  <adapters_conf id="DEMO">

    <!-- Mandatory. Define the Metadata Adapter. -->
    <metadata_provider>

      <install_dir>portfolio</install_dir>

      <!-- Mandatory. Java class name of the adapter. -->
      <adapter_class>portfolio_demo.adapters.PortfolioMetadataAdapter</adapter_class>

      <!-- Optional for PortfolioMetadataAdapter.
           Configuration file for the Adapter's own logging.
           Logging is managed through log4j. -->
      <param name="log_config">adapters_log_conf.xml</param>
      <param name="log_config_refresh_seconds">10</param>

      <!-- Optional, managed by the inherited LiteralBasedProvider.
           See LiteralBasedProvider javadoc. -->
      <!--
      <param name="max_bandwidth">40</param>
      <param name="max_frequency">3</param>
      <param name="buffer_size">30</param>
      <param name="distinct_snapshot_length">10</param>
      <param name="prefilter_frequency">5</param>
      <param name="allowed_users">user123,user456</param>
      -->

      <!-- Optional, managed by the inherited LiteralBasedProvider.
           See LiteralBasedProvider javadoc. -->
      <param name="item_family_1">portfolio.*</param>
      <param name="modes_for_item_family_1">COMMAND</param>
      
      <param name="item_family_2">item.*</param>
      <param name="modes_for_item_family_2">MERGE</param>

    </metadata_provider>

    <!-- Mandatory. Define the Data Adapter. -->
    <data_provider name="PORTFOLIO_ADAPTER">

      <install_dir>portfolio</install_dir>
    
      <!-- Mandatory. Java class name of the adapter. -->
      <adapter_class>portfolio_demo.adapters.PortfolioDataAdapter</adapter_class>

      <!-- Optional for PortfolioDataAdapter.
           Configuration file for the Adapter's own logging.
           Leans on the Metadata Adapter for the configuration refresh.
           Logging is managed through log4j. -->
      <param name="log_config">adapters_log_conf.xml</param>

    </data_provider>

    <!-- Mandatory. Define the Data Adapter. -->
    <data_provider name="QUOTE_ADAPTER">

      <install_dir>Stocklist</install_dir>
    
      <!-- Mandatory. Java class name of the adapter. -->
      <adapter_class>stocklist_demo.adapters.StockQuotesDataAdapter</adapter_class>

      <!-- Optional for StockQuotesDataAdapter.
           Configuration file for the Adapter's own logging.
          Logging is managed through log4j. -->
      <param name="log_config">adapters_log_conf.xml</param>
      <param name="log_config_refresh_seconds">10</param>

    </data_provider>

  </adapters_conf>
```

# See Also #

## Clients using this Adapter ##
* [Lightstreamer - Basic Portfolio Demo - HTML Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript#basic-portfolio-demo)
* [Lightstreamer - Portfolio Demo - HTML Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript#portfolio-demo)
* [Lightstreamer - Portfolio Demo - Flex Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-flex)
* [Lightstreamer - Portfolio Demo - Dojo Toolkit Client](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-dojo)

## Related projects ##
* [Lightstreamer - Reusable Metadata Adapters - Java SE Adapter](https://github.com/Weswit/Lightstreamer-example-ReusableMetadata-adapter-java)
* [Lightstreamer - Stock-List Demo - Java SE Adapter](https://github.com/Weswit/Lightstreamer-example-StockList-adapter-java)

## The same Demo Adapter with other technologies ##
* [Lightstreamer - Portfolio Demo - .NET Adapter](https://github.com/Weswit/Lightstreamer-example-Portfolio-adapter-dotnet)

# Lightstreamer Compatibility Notes #

- Compatible with Lightstreamer SDK for Java Adapters since 5.1
