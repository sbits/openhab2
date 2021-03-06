Documentation of the CometVisu Backend for openHAB2

## Introduction

This adds a backend for the web based visualization CometVisu (http://www.cometvisu.org). The CometVisu is a highly customizable visualization, that runs in any browser. Despite the existing browser based UI´s in openHAB, the CometVisu does not rely on sitemaps (although they can be used). The layout is defined with an XML-based configuration file, which allows a multi column layout.

This is just a short overview, more details will be added soon!

## Requirements

* openHAB 2.0 or greater<br>
* CometVisu >=0.8.6? (not released yet) or greater (https://github.com/CometVisu/CometVisu/releases).<br>

## Installation

* Copy the addon org.openhab.ui.cometvisu*.jar to the openHAB addon folder
* Then just extract the "release" folder (the one which contains the index.html file) of the downloaded CometVisu archive in the runtime folder off your openHAB2 server. 
Note: if you want to store the CometVisu client in another folder you have to set the webfolder in your config file

## Configuration
### Server configuration
You can customize some setting of the CometVisu backend in a configuration file named 'cometvisu.cfg' which has to be created in the 'conf/services/' folder of your openHAB2 instance.
The following settings are available:

Relative path to the folder the CometVisu client can be found:
```
cometvisu:webFolder=runtime/web/src/
```

The alias where you can access the client e.g http://\<openhab-server\>:8080/\<webAlias\>/
```
cometvisu:webAlias=/cometvisu
```

Enable icon mapping from openHAB-items to CometVisu-items (Note this is only needed if you use the automatic sitemap->config generation feature)
```
cometvisu:icons>enableMapping=true
```

List of mappings of structure: icons.mapping\>\<openhab-icon\>=\<cometvisu-icon\>
```
cometvisu:icons.mapping>firstfloor=control_building_int_og
cometvisu:icons.mapping>groundfloor=control_building_int_eg
```
The list of icons in the CometVisu is available at:
```
http://<openhab-server>:8080/<webAlias>/icon/knx-uf-iconset/showicons.php
```

### Override CometVisu files
You can create a folder called 'cometvisu' in openHAB's 'conf/' folder and create files there which should be used instead of CometVisu's own. For example you can put all your customizations there (e.g. you config, custom.css, etc.). So the CometVisu itself stays untouched and can easily be replaced with a new release.

Please use the same folder structure and file names as they are used in the CometVisu package. For example if you want to replace the original config with your own copy the file to the following path:
```
'conf/cometvisu/config/visu_config.xml'
```

Or for design customizations use:
```
'conf/cometvisu/designs/metal/custom.css'
```

### Client configuration
If you don´t use the given example below, please make sure that you correctly configure openHAB as backend in the CometVisu-Config by adding `backend="oh2"` to the root pages-element, like `<pages...backend="oh2"...>`

It is a good start to use the new sitemap support, to get a working example config, which you can store an customize for your needs.
Open a sitemap in the CometVisu client like this:
```
http://<openhab-server>:8080/<webAlias>/?config=<sitemap-name>
```

You can use the editor to change and store this config. You can start with the demo-sitemap to get a quick overview about how the CometVisu looks like.

## New features (compared to the 1.x version)
* PHP support: Editor is working, rrslog-plugin can be used (see examples)
* Persistence support: Any persisted item can be used to create a chart
* GroupItem support: as known from the openHAB UI´s the group-functions like the number of open windows in a group of contacts
can be shown in the visu
* ...


## Examples
### Show how many lights are switched on 
* Items: uses the demo.items provided by openHAB2
* Config (please add the infoaction-plugin in pages->meta->plugins in your config in order to use this):
```
<infoaction>
	<label>Lights</label>
	<widgetinfo>
		<info format="%d">
			<address transform="OH:number" mode="read" variant="">number:Lights</address>
		</info>
	</widgetinfo>
	<widgetaction>
		<switch mapping="OnOff" styling="Red_Green">
			<address transform="OH:switch" variant="">Lights</address>
		</switch>
	</widgetaction>
</infoaction>
```
  * use it in navbar:
```
<pagejump target="Lights">
	<layout colspan="0" />
	<label>
		<icon name="control_building_empty" />Übersicht
	</label>
	<widgetinfo>
		<info format="%d">
			<layout colspan="0" />
			<address transform="OH:number" mode="read" variant="">number:Lights</address>
		</info>
	</widgetinfo>
</pagejump>
```
###Charts:
* Items: see demo.items
* Config:
```
<diagram height="300px" series="day" refresh="600">
<rrd consolidationFunction="AVERAGE">Weather_Temperature</rrd>
<rrd consolidationFunction="AVERAGE">Weather_Temp_Max</rrd>
<rrd consolidationFunction="AVERAGE">Weather_Temp_Min</rrd>
</diagram>
```
some notes:
  * consolidationFunction is only obeyed, when the item is persisted by the rrd4f persistence service
  * altough you have to define <rrd...>Item_name</rrd> for every line in the config, the used items doe not have to be persisted by the rrd4j persistence service, any other service will work too

###RSS-Log:
* Items: no special item needed
* Config:
```
<rsslog src="plugins/rsslog/rsslog_pdo.php" refresh="60" mode="last">
	<label>Events</label>
	<layout colspan="12" rowspan="9"/>
</rsslog>
``` 
  * Fill the log from a rule:
```
var content = "Call recevied from 123456",encode("UTF-8")
var tag = "Call".encode("UTF-8")
sendHttpGetRequest("http://<openhab-server>:8080/<webAlias>/plugins/rsslog_pdo.php?c="+content+"&t="+tag)
```
###RSS-Log from persisted item:
* Items:
```
String Logger
```
* Config:
```
<rsslog src="plugins/rsslog/rsslog_oh.php" refresh="60" filter="Logger">
	<label>Events</label>
	<layout colspan="12" rowspan="9"/>
</rsslog>
``` 
  * Fill the log from a rule:
```
# Message structure is <title>|<content>|<state>|<tags>
sendCommand(Logger,"Call received|Received call from 123456789|0|Call,Incoming")
# but you can also just use any string like
sendCommand(Logger,"Received call from 123456789")
```
###ColorItem (supported since CometVisu-Release 0.8.2) => 
```
<colorchooser>
  <label>Color</label>
  <address transform="OH:color" variant="rgb">ITEM_NAME</address>
</colorchooser>
```
Please note: You have to add the colorchooser plugin in the meta>plugins section of you config

## Known problems
Not all of the PHP-based functions in the CometVisu client have been tested so far. The untested features are:
* Automatic configuration upgrade, when the library version has changed
* calendarlist plugin
* upnpcontroller plugin

The sitemap support can only be used as a starting point for own customizations, e.g. you open an automatic generated config and store it in by CometVisu by using the editor. Then you start to customize it to your own needs. New items must by added by hand from the moment you stored and changed the generated config.

### 403 error 
If you get an 403 - Access Denied error, when you try to open the CometVisu in your browser you have not copied the correct release folder into the \<webFolder\> folder. Please check if there is a subfolder with the exact name "release/", which contains an index.html file and copy the content to the folder defined in the \<webFolder\>-property.

## TODO
Maybe it is possible to define a general structure (in addition to a sitemap), that maps and groups items based on their context, e.g. which floor/room/subsection the belong to
```
<floor>
	<room navbar="top">
		<all-lights-items colspan="12"> 
		<all-rollershutter-items colspan="6"> 
		<all-contact-items colspan="6"> 
		<all-other-items colspan="12">
	</room>
</floor>
```
Something like that could help to improve the sitemap->config generation.

## Screenshots

some screenshots can be found here:
- http://knx-user-forum.de/forum/supportforen/cometvisu/26809-allgemeine-frage-wie-sieht-eure-cv-startseite-aus

## Links

* German CometVisu Support Forum: http://knx-user-forum.de/forum/supportforen/cometvisu
* User documentation for the CometVisu: http://www.cometvisu.org/
* GitHub project page of the CometVisu: https://github.com/CometVisu/CometVisu
* Current development version of the openHAB2 compatible CometVisu: https://github.com/peuter/CometVisu/tree/openhab2
