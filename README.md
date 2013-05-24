
Lightstreamer Portfolio Demo Adapter
====================================


This project include the implementation of the Metadata and Data Adapter in Java managing the functionalities required by Lightstreamer Portfolio demos.

Java Data Adapter and Metadata Adapter
--------------------------------------

This Data Adapter accepts subscriptions to Items representing stock portfolios and inquiries a (simulated) portfolio feed, getting the current portfolio contents and waiting for update events.
The  Metadata Adapter is suitable for managing client requests to both the sample Quote Data Adapter and the sample Portfolio Data Adapter.
It inherits from the LiteralBasedProvider and in addition, it implements the NotifyUserMessage method, in order to handle "sendMessage" requests from the Portfolio Demo client.
This allows the Portfolio Demo client to use "sendMessage" in order to submit buy/sell orders to the Portfolio Feed Simulator. The communication to the Portfolio Feed Simulator, through the Portfolio Data Adapter, is handled here.

Configure Lightstreamer
-----------------------

After you have Downloaded and installed Lightstreamer, please go to the "adapters" folder of your Lightstreamer Server installation. You should find a "Demo" folder containing some adapter ready-made for several demo including the Portfolio ones, please note that the MetaData Adapter jar installed is a mixed one that combines the functionality of several demos. If this is not your case because you have removed the "Demo" folder or you want to install the Portfolio adapter set alone, please follow this steps to configure the Portfolio adapter properly:

1. You have to create a new folder to deploy the portfolio adapters, let's call it "portfolio", and a "lib" folder inside it.
2. Create an "adapters.xml" file inside the "portfolio" folder and use the following contents (this is an example configuration, you can modify it to your liking):
```xml      
<?xml version="1.0"?>
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
3. Get the ls-adapter-interface.jar, ls-generic-adapters.jar, and log4j-1.2.15.jar files from the [Lightstreamer 5 Colosseo distribution](http://www.lightstreamer.com/download).
4. Copy into "lib" folder the jars LS_portfolio_metadata_adapter.jar, LS_portfolio_feed_simulator.jar, and LS_portfolio_data_adapter.jar created for something like these commands
```sh
 >javac -source 1.7 -target 1.7 -nowarn -g -classpath compile_libs/log4j-1.2.15.jar -sourcepath src/src_feed -d tmp_classes src/src_feed/portfolio_demo/feed_simulator/Portfolio.java
 
 >jar cvf LS_portfolio_feed_simulator.jar -C tmp_classes src_feed
 
 >javac -source 1.7 -target 1.7 -nowarn -g -classpath compile_libs/log4j-1.2.15.jar;compile_libs/ls-adapter-interface/ls-adapter-interface.jar;compile_libs/ls-generic-adapters/ls-generic-adapters.jar;LS_portfolio_feed_simulator.jar -sourcepath src/src_portfolio -d tmp_classes src/src_portfolio/portfolio_demo/adapters/PortfolioDataAdapter.java
 
 >jar cvf LS_portfolio_data_adapter.jar -C tmp_classes src_portfolio
 
 >javac -source 1.7 -target 1.7 -nowarn -g -classpath compile_libs/log4j-1.2.15.jar;compile_libs/ls-adapter-interface/ls-adapter-interface.jar;compile_libs/ls-generic-adapters/ls-generic-adapters.jar;LS_portfolio_feed_simulator.jar;LS_portfolio_data_adapter.jar -sourcepath src/src_metadata -d tmp_classes src/src_metadata/portfolio_demo/adapters/PortfolioMetadataAdapter.java
 
 >jar cvf LS_portfolio_metadata_adapter.jar -C tmp_classes src_metadata
```

Please note that to work with fully functionality the [Portfolio Demo](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript) you have to deploy on your Lightstreamer instance the QUOTE_ADAPTER adapter too.
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

See Also
--------

* TODO: add link to GitHub project of [Lightstreamer Basic Portfolio Demo Client for JavaScript]
* [Lightstreamer Portfolio Demo Client for JavaScript](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-javascript)
* [Lightstreamer Portfolio Demo Client for Dojo](https://github.com/Weswit/Lightstreamer-example-Portfolio-client-dojo)
* [Lightstreamer Stock-List Demo Adapter](https://github.com/Weswit/Lightstreamer-example-StockList-adapter-java)

Lightstreamer Compatibility Notes
---------------------------------

- Compatible with Lightstreamer SDK for Java Adapters since 5.1