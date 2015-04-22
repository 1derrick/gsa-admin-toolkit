

# Introduction #
The Python connector framework provides a simple, quick way to write connectors for the GSA. It uses the same underlying protocol as the [official Java connector framework](http://code.google.com/apis/searchappliance/documentation/connectors/200/connector_dev/cdg_intro.html). However, this code is not supported by Google. If there is any change to the underlying protocol, then connectors written using the Python framework will stop working.

# Writing a connector #
## Prerequisites ##
  * Python 2.4
  * [CherryPy 3](http://cherrypy.org/)
The sample files for this documentation are available in the repository.

## Example ##
Here is a simple connector that fetches the contents of a URL and then pushes it to the GSA as a content feed:

```
import connector
import urllib2

class URLConnector(connector.TimedConnector):
  CONNECTOR_TYPE = 'url-connector'
  CONNECTOR_CONFIG = {
      'url': { 'type': 'text', 'label': 'URL to fetch' },
      'delay': { 'type': 'text', 'label': 'Fetch delay' }
  }

  def init(self):
    self.setInterval(int(self.getConfigParam('delay')))

  def run(self):
    # fetch the contents of the URL
    url = self.getConfigParam('url')
    req = urllib2.Request(url)
    response = urllib2.urlopen(req)
    content = response.read()

    # push it to the GSA as a content feed
    feed = connector.Feed('incremental')
    feed.addRecord(url=url, action='add', mimetype='text/html', content=content)
    self.pushFeed(feed)
```

Here are a few things to note about this connector:
  * The `URLConnector` extends the `TimedConnector` class. This `TimedConnector` is a special type of connector that implements a simple scheduling scheme--when it is started, it will call its `run` method periodically, with a delay interval specified by the `setInterval` method.
  * All connectors must have a unique `CONNECTOR_TYPE` field that identifies the particular type of connector.
  * The `CONNECTOR_CONFIG` dictionary specifies configuration fields for the connector. When the connector's configuration form is brought up in the GSA administration interface, this `CONNECTOR_CONFIG` will be used to construct the form.
  * The `init` method, which is available to all connectors, is called when the connector is created or when its configuration is modified. It's the perfect place to load and process configuration parameters.
  * The `Feed` class in the `connector` module represents a data feed. Any data placed in the feed with `addRecord` will be converted to an XML representation readable by the GSA when it is sent by the `pushFeed` method. Here, the `URLConnector` sends the content as HTML to be added to the GSA's index.

## Running the connector ##
Now that we have a connector written, it's time to use the connector manager to expose this connector to the GSA.

  * Start the connector manager.
```
python connectormanager.py --debug --connectors=url_connector.URLConnector
```
> The `--debug` flag causes the script to print debug information as it executes (more specifically, all calls to `log()` from connectors are printed). The `--connectors` flag lists the connector classes to be loaded by the connector manager. Each class must be specified with its Python package name relative to the location of `connectormanager.py`. In the above example, the `URLConnector` class is located in the file `url_connector.py` in the same directory as `connectormanager.py`.

> If desired, the `--use_ssl` flag can be specified to use SSL. The `--port` flag can be used to change the default port, which is 38080.

> It's also important to note that multiple connector types can be loaded. To do so, simply separate their names with commas. For example:
```
python connectormanager.py --debug --connectors=url_connector.URLConnector,other_connector.OtherConnector
```

> (For the purposes of the rest of this section, suppose that this script is running on the host `myconnector.mydomain.com`).
  * Next, open the GSA administration console. Create a new connector manager and set the host to `http://myconnector.mydomain.com:38080` (note: no '/' at the end).
  * Create a connector instance with the new connector manager, and configure the connector appropriately.
  * Ensure that the URLs that the connector feeds into the GSA match the patterns listed in the admin console in the Crawl URLs page. Otherwise, the GSA will reject the feed provided by the connector.

# The `Connector` abstract class #
The `URLConnector` above extends `TimedConnector` in order to use its scheduling scheme. However, the raw `Connector` class can be used to implement a custom scheduling scheme. It also provides a wealth of other methods essential in writing full-featured connectors.

The following is a description of key aspects of `Connector`.

## Identification ##
All concrete connector implementations must provide a unique `CONNECTOR_TYPE` string field that identifies the particular connector type.

## General ##
  * `init()`: An initialization method that a connector can implement if needed. This is called when the connector is constructed and right after its configuration or schedule is changed. Loading such configuration or scheduling parameters should be done here.
  * `startConnector()`: Called whenever the connector is created or after its configuration or schedule is changed. For example, the connector manager calls the `startConnector` methods of all its stored connectors whenever it starts up. This method must be implemented for the connector to work.

> While both `init` and `startConnector` may seem very similar, they do have a crucial difference: `init` is called right after the connector is constructed, whereas `startConnector` is called to kick off the connector's traversal. Therefore, `init` should be used to do initialization (like reading configuration values), whereas `startConnector` should be used to define behavior at the beginning of traversals (like starting timing threads). Consider the classes `TimedConnector` and `URLConnector`. `TimedConnector` is an abstract connector that provides a timed scheduling facility, and `URLConnector` is a concrete connector that extends from `TimedConnector. `TimedConnector` implements `startConnector` to spawn scheduling threads, but leaves `init` alone; on the other hand, `URLConnector` uses `init` to read configuration values from the GSA administration console, but leaves `startConnector` alone to be inherited from `TimedConnector`.

> It's guaranteed that `init` will be called before `startConnector`.

  * `stopConnector()`: Called when the connector manager shuts down, or right before configuration or scheduling is set. Should be used to stop the connector's traversal--`TimedConnector`, for example, uses this to kill its child threads. This method must be implemented for the connector to work.
  * `restartConnectorTraversal()`:
Note that the connector manager does not give any regard to what happens between `startConnector` and `stopConnector` calls--in other words, it's completely up to the connector to implement its own scheduling scheme. The `TimedConnector` implements an example scheduling scheme; see `connector.py` for details on how to do this.
  * `getStatus()`: Called when the GSA needs the status of the connector. Should return the status code as a string. The code '0' indicates that the connector is OK.

## Configuration and scheduling ##
The configuration and scheduling parameters for the connector are specified by the user in the GSA administration console. The following methods are provided to easily access these parameters.
  * `getConfigParam(param_name)`: Reads the connector's configuration and returns the value of a parameter contained within. The name of the configuration parameter is identical to the name of its field in the configuration HTML form.
  * `getLoad()`: Returns the load setting (documents to traverse per minute).
  * `getRetryDelay()`: Returns the retry delay in milliseconds. The retry delay is the time that the connector should wait after it finishes traversal and before it starts a new traversal. The default value is 300000 milliseconds.
  * `getTimeIntervals()`: Returns the time intervals over which to traverse. The method returns a string, with individual time intervals delimited by colons.

The `Connector` class provides more lower-level methods for retrieving configuration and scheduling parameters, but the above methods should suffice in most situations. See the `Connector` documentation for more details.

### Configuration forms ###
When a user opens the GSA administration page to configure a connector, it is up to the connector to provide its own configuration page as raw HTML text. The following connector methods are called by the connector manager to provide this HTML:
  * `@classmethod getConfigForm()`: Should return empty configuration form table rows encapsulated inside a `<CmResponse><ConfigureResponse><FormSnippet>` CDATA tag. This method is called before the connector object is ever instantiated.
  * `getPopulatedConfigForm()`: Like `getConfigForm()`, except with configuration fields already filled in. The rows returned should only be encapsulated inside a CDATA tag.

Writing code to generate HTML for configuration forms can be quite a hassle. Thus, `Connector` provides a configuration form generator that can create this HTML based on a configuration form specification. A connector can take advantage of this feature by providing a `CONNECTOR_CONFIG` field:
```
class ExampleConnector(connector.Connector):
  CONNECTOR_TYPE = 'example-connector'
  CONNECTOR_CONFIG = {
      'example_field': { 'type': 'text', 'label': 'Example field' }
  }
  # no need to implement getConfigForm() and getPopulatedConfigForm().

  # ... remaining code omitted ...
```
In this configuration specification, there is a single field named `example_field`. When the HTML is generated, a text input tag will be emitted, and a label with the text "Example field" will be placed to its left. After the connector is created, the value of this field can be accessed with `getConfigParam('example_field')`.
If a connector implementation does choose to use this configuration form generation feature, then it does not need to implement the `getConfigForm()` and `getPopulatedConfigForm()` methods.

## Data storage ##
`Connector` provides the capability to store data that persists between `stopConnector()` and `startConnector()` calls (for example, this data would persist when the connector manager is completely shut down). This data is stored locally along with the connector's configuration data within the connector manager configuration file. The data can be of any type than can be encoded by Pickle.
  * `getData()`: Returns the stored data.
  * `setData(data)`: Sets the data to be stored upon connector shutdown.
Note that the data is written to disk only when the connector manager saves its configuration file.

## Sending data to the GSA ##
Connectors must send data to the GSA in the XML feed format, which is fully described in the [Feeds Protocol Developer's Guide](http://code.google.com/apis/searchappliance/documentation/64/feedsguide.html). It's possible for connectors to manually construct this XML, but the `Feed` class provides a simple way to create feeds without directly dealing with any XML. These `Feed`s can be created and manipulated by the following methods:
  * `__init__(feedtype)`: Creates the feed. `feedtype` is a string that specifies the type of the feed, either `incremental`, `full`, or `metadata-and-url`.
  * `addRecord(**kwargs)`: Adds a record to the feed.
    * Arguments:
      * `url`, `displayurl`, `mimetype`, etc.: Attributes for the record tag, to be provided as needed (should be strings). The GSA requires the url and metadata arguments to be provided. See http://code.google.com/apis/searchappliance/documentation/64/feedsguide.html#defining_the_xml for a full list of record tag attributes.
      * `metadata`: Optional. Metadata tags to add to the feed, as a dictionary that maps strings to strings.
      * `content`: Optional; required if the feed is a content feed. Adds a content tag to the record. Should be a string.
    * Example usage of `addRecord`:
```
addRecord(url='http://example.com/index.html', action='add', mimetype='text/html', content='<html>...</html>')
```
> > This adds a record to a content feed, with the record element attributes `url`, `action`, and `mimetype`. A `<content>` element is also added. The following XML will be produced by `toXML` for this record:
```
<record url='http://example.com/index.html' action='add' mimetype='text/html'>
  <content encoding="base64binary">
    ... the base64-encoded version of the content ...
  </content>
</record>
```
  * `clear()`: Clears all the records added to the feed.

Once a feed is constructed and populated with records, it must be sent to the GSA. The following method in `Connector` accomplishes this very task:
  * `pushFeed(feed)`: Pushes the given `Feed` object to the GSA, and returns the GSA response string.

Both the `Connector` class and the `Feed` class provide additional methods for manipulating and sending feeds, but the most frequently-used are described above. See the documentation for the respective classes for more information about the other methods.

## Logging ##
The `Connector` class provides a method, `logger()`, that returns a Python Logger object designated for the purpose of the calling connector class. See the [documentation](http://docs.python.org/library/logging.html#loggers) for the Python logging library for details on how to use this Logger object.

# Provided connectors #
The following are some connectors provided in the repository:
  * `TimedConnector`: An abstract connector that runs continually with a specified time interval. A connector that implements this `TimedConnector` just needs to override the `run` method. The `getInterval` and `setInterval` methods are provided to manipulate the timer delay (the default is 15 minutes).

> If a TimedConnector implementation's interval comes from the configuration form, then `setInterval` should be called in an `init` method with the appropriate value from `getConfigParam`.
> For more information, see the documentation inside `connector.py`.
  * `ExampleConnector`: A bare-bones connector that does absolutely nothing. It serves as a template for writing new connectors.
  * `SitemapConnector`: A connector that crawls a site given a `sitemap.xml` file. It extends `TimedConnector` to use its scheduling facilities.
  * `URLConnector`: A connector that crawls a single URL, which is specified in the connector's configuration form in the admin interface. The contents of the URL are pushed to the GSA as a content feed. This connector extends `TimedConnector`.
  * `SMBConnector`: A connector that crawls a public SMB share. It uses [smbcrawler](http://gsa-admin-toolkit.googlecode.com/svn/trunk/smbcrawler.py) to fetch a list of files in the share, and then invokes `smbclient` to download each file locally and then pushes each file to the GSA as a content feed. This connector extends `TimedConnector`.