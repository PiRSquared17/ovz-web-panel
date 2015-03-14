

# Introduction #

Remote API allows to integrate OpenVZ Web Panel with third-party systems, e.g. billing systems. API exposes business logic objects and operations and gives the ability to manipulate with them using HTTP requests with XML responses.

Typical API entry points:
```
http://<host>/api/hardware_servers/get?id=38
```
where "hardware\_servers" - name of controller for manipulating with physical servers, "get" - particular method of controller, "id=38" - additional request parameters.

The response for such call is:
```
<?xml version="1.0" encoding="UTF-8"?>
<hardware_server>
  <daemon_port type="integer">7767</daemon_port>
  <default_os_template>debian</default_os_template>
  <default_server_template>vps.basic2</default_server_template>
  <description></description>
  <host>host</host>
  <id type="integer">38</id>
  <use_ssl type="boolean">true</use_ssl>
</hardware_server>
```

# Playing with API #

The most simple way to become familiar with API is to play with it using browser. It is recommended to use Mozilla Firefox with WebDeveloper extension installed. Firefox shows XML responses in human-readable manner. WebDeveloper extension allows to perform re-login procedure (Miscellaneous -> Clear Private Data -> HTTP Authentication).

# Authorization #

Remote API uses HTTP basic authorization mechanism.

# Error Handling #

In case of invalid requests or due to some other problems you can get errors instead of expected XML output.

Attempt to access to non-existing virtual server:
```
http://<host>/api/virtual_servers/get?id=10000
```

Response:
```
<?xml version="1.0" encoding="UTF-8"?>
<error>
  <code>object_not_found</code>
  <message>Object not found.</message>
</error>
```

Attempt to access location, which is not accesible by current user:
```
http://<host>/api/hardware_servers/list
```

Response:
```
<?xml version="1.0" encoding="UTF-8"?>
<error>
  <code>access_denied</code>
  <message>Access denied.</message>
</error>
```

Very rarely you can see internal error. In most cases this means that system doesn't work correctly and maybe you found a bug. Typical response in such case is shown below:
```
<?xml version="1.0" encoding="UTF-8"?>
<error>
  <code>internal_error</code>
  <message>Internal error.</message>
  <details>Test error</details>
</error>
```

# Provisioning Scenarios #

Before provisioning you should prepare infrastructure:
  * connect physical servers
  * install OS templates
  * create/modify server templates
  * create user roles

Typical provisioning scenario may look like following:
  * create user to be able to login
  * create virtual server and assign user as owner
  * show/send notification to user that server is ready

# API Client Code Samples #

Below are the client code samples to interacting with Remote API. Each sample tries to perform API call to fetch list of physical servers, dumps fetched XML and prints hostname for each physical server found.

Client code samples are located at repository by the URL:
http://code.google.com/p/ovz-web-panel/source/browse/#svn%2Ftrunk%2Ftest%2Futils%2Fremote-api

## Ruby Client ##

```
require 'net/http'
require 'rexml/document'

host = 'host'
port = 3000
user = 'admin'
password = 'password'
api_method = '/api/hardware_servers/list'

Net::HTTP.start(host, port) do |http|
  request = Net::HTTP::Get.new(api_method)
  request.basic_auth user, password
  response = http.request(request)
  result = response.body

  print result

  doc = REXML::Document.new(result)
  doc.elements.each('//hardware_server/host') do |element|
    print "server: #{element.text}\n"
  end
end
```

## PHP Client ##

```
$host = 'host';
$port = 3000;
$user = 'admin';
$password = 'password';
$api_method = '/api/hardware_servers/list';

$context = stream_context_create(array(
    'http' => array(
        'header'  => "Authorization: Basic " . base64_encode("$user:$password")
    )
));

$result = file_get_contents("http://$host:$port$api_method", false, $context);

echo $result;

$doc = simplexml_load_string($result);

foreach($doc->xpath('//hardware_server/host') as $node) {
    echo "server: $node\n";
}
```

# Methods #

## Physical Servers ##

### list ###

List of connected physical servers.

Request URL:
```
http://<host>/api/hardware_servers/list
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<hardware_servers type="array">
  <hardware_server>
    <daemon_port type="integer">7767</daemon_port>
    <default_os_template>ubuntu-9.10-x86</default_os_template>
    <default_server_template>vps.new</default_server_template>
    <description></description>
    <host>host</host>
    <id type="integer">4</id>
    <use_ssl type="boolean">false</use_ssl>
  </hardware_server>
  <hardware_server>
    ...
  </hardware_server>
</hardware_servers>
```

Parameters: absent.

### get ###

Detailed information about particular physical server.

Request URL:
```
http://<host>/api/hardware_servers/get?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<hardware_server>
  <daemon_port type="integer">7767</daemon_port>
  <default_os_template>debian</default_os_template>
  <default_server_template>vps.basic2</default_server_template>
  <description></description>
  <host>host</host>
  <id type="integer">38</id>
  <use_ssl type="boolean">true</use_ssl>
</hardware_server>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |

### server\_templates ###

List of server templates available at physical server.

Request URL:
```
http://<host>/api/hardware_servers/server_templates?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<server_templates type="array">
  <server_template>
    <id type="integer">108</id>
    <name>light</name>
  </server_template>
  <server_template>
    ...
  </server_template>
</server_templates>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |

### os\_templates ###

List of installed OS templates at physical server.

Request URL:
```
http://<host>/api/hardware_servers/os_templates?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<os_templates type="array">
  <os_template>
    <id type="integer">477</id>
    <name>centos-5-x86_64</name>
    <size type="integer">182</size>
  </os_template>
  <os_template>
    ...
  </os_template>
</os_templates>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |

### connect ###

Connect new physical server.

Request URL:
```
http://<host>/api/hardware_servers/connect?host=<host>&root_password=<password>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
  <details>
    <id type="integer">45</id>
  </details>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| host | + | IP address or host name of new server |
| root\_password | + | SSH root user password |
| description |  | server description |

### sync ###

Synchronize information about virtual servers at physical server.

Request URL:
```
http://<host>/api/hardware_servers/sync?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |

### update ###

Update connection settings for physical server.

Request URL:
```
http://<host>/api/hardware_servers/update?id=<id>&host=<host>&root_password=<password>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
  <details>
    <id type="integer">45</id>
  </details>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |
| host | + | IP address or host name of new server |
| root\_password | + | SSH root user password |
| description |  | server description |

### disconnect ###

Disconnect physical server.

Request URL:
```
http://<host>/api/hardware_servers/disconnect?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |

### virtual\_servers ###

List of virtual servers at physical server.

Request URL:
```
http://<host>/api/hardware_servers/virtual_servers?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<virtual_servers type="array">
  <virtual_server>
    <cpu_limit type="integer" nil="true"></cpu_limit>
    <cpu_units type="integer">1000</cpu_units>
    <cpus type="integer" nil="true"></cpus>
    <description nil="true"></description>
    <diskspace type="integer">2000</diskspace>
    <expiration_date type="date" nil="true"></expiration_date>
    <hardware_server_id type="integer">38</hardware_server_id>
    <host_name>ldap.lan</host_name>
    <id type="integer">299</id>
    <identity type="integer">131</identity>
    <ip_address>192.168.0.131</ip_address>
    <memory type="integer">272</memory>
    <nameserver>192.168.0.4</nameserver>
    <orig_os_template>ubuntu-10.04-x86</orig_os_template>
    <orig_server_template>vps.basic</orig_server_template>
    <search_domain>lan</search_domain>
    <start_on_boot type="boolean">true</start_on_boot>
    <state>running</state>
    <user_id type="integer">0</user_id>
  </virtual_server>
  <virtual_server>
    ...
  </virtual_server>
 <virtual_servers>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | physical server ID |

## Virtual Servers ##

### get ###

Detailed information about virtual server.

Request URL:
```
http://<host>/api/virtual_servers/get?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<virtual_server>
  <cpu_limit type="integer" nil="true"></cpu_limit>
  <cpu_units type="integer">1000</cpu_units>
  <cpus type="integer" nil="true"></cpus>
  <description nil="true"></description>
  <diskspace type="integer">2000</diskspace>
  <expiration_date type="date" nil="true"></expiration_date>
  <hardware_server_id type="integer">38</hardware_server_id>
  <host_name>ldap.lan</host_name>
  <id type="integer">299</id>
  <identity type="integer">131</identity>
  <ip_address>192.168.0.131</ip_address>
  <memory type="integer">272</memory>
  <nameserver>192.168.0.4</nameserver>
  <orig_os_template>ubuntu-10.04-x86</orig_os_template>
  <orig_server_template>vps.basic</orig_server_template>
  <search_domain>lan</search_domain>
  <start_on_boot type="boolean">true</start_on_boot>
  <state>running</state>
  <user_id type="integer">0</user_id>
</virtual_server>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |

### delete ###

Delete virtual server.

Request URL:
```
http://<host>/api/virtual_servers/delete?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |

### get\_advanced\_limits ###

List of advanced limits current values for virtual servers.

Request URL:
```
http://<host>/api/virtual_servers/get_advanced_limits?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<limits type="array">
  <limit>
    <name>KMEMSIZE</name>
    <soft_limit>11055923</soft_limit>
    <hard_limit>11377049</hard_limit>
  </limit>
  <limit>
    ...
  </limit>
</limits>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |

### stop ###

Stop virtual server.

Request URL:
```
http://<host>/api/virtual_servers/stop?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |

### own\_servers ###

List of virtual server controlled by current user.

Request URL:
```
http://<host>/api/virtual_servers/own_servers
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<virtual_servers type="array">
  <virtual_server>
    <cpu_limit type="integer" nil="true"></cpu_limit>
    <cpu_units type="integer">1000</cpu_units>
    <cpus type="integer" nil="true"></cpus>
    <description></description>
    <diskspace type="integer">2048</diskspace>
    <expiration_date type="date" nil="true"></expiration_date>
    <hardware_server_id type="integer">4</hardware_server_id>
    <host_name>117.ip.lan</host_name>
    <id type="integer">202</id>
    <identity type="integer">117</identity>
    <ip_address>192.168.0.117</ip_address>
    <memory type="integer">512</memory>
    <nameserver>192.168.0.4</nameserver>
    <orig_os_template>ubuntu-9.04-x86</orig_os_template>
    <orig_server_template>vps.new</orig_server_template>
    <search_domain nil="true"></search_domain>
    <start_on_boot type="boolean">true</start_on_boot>
    <state>stopped</state>
    <user_id type="integer">21</user_id>
  </virtual_server>
  <virtual_server>
    ...
  </virtual_server>
</virtual_servers>
```

Parameters: absent.

### create ###

Create new virtual server.

Request URL:
```
http://<host>/api/virtual_servers/create?hardware_server_id=<hardware_server_id>&identity=<identity>...
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
  <details>
    <id type="integer">350</id>
  </details>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| hardware\_server\_id | + | physical server ID |
| identity | + | virtual server identity (container id) |
| orig\_os\_template | + | OS template name |
| orig\_server\_template | + | Server template name |
| description |  | server description |
| ip\_address |  | server IP address |
| host\_name |  | server host name |
| password |  | SSH root user password |
| user\_id |  | user ID (server owner) |
| diskspace |  | disk space, Mb |
| memory |  | RAM memory, Mb |
| cpu\_units |  | number of CPU units |
| cpus |  | number of allowed CPUs |
| cpu\_limit |  | percent of CPU time |
| expiration\_date |  | expiration date |
| nameserver |  | DNS server |
| search\_domain |  | search domain (for DNS lookup) |
| start\_on\_boot |  | start virtual server after physical server reboot |

### start ###

Start virtual server.

Request URL:
```
http://<host>/api/virtual_servers/start?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |

### restart ###

Restart virtual server.

Request URL:
```
http://<host>/api/virtual_servers/restart?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |

### update ###

Update virtual server attributes.

Request URL:
```
http://<host>/api/virtual_servers/update?id=<id>password=<password>...
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
  <details>
    <id type="integer">350</id>
  </details>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | virtual server ID |
| description |  | server description |
| ip\_address |  | server IP address |
| host\_name |  | server host name |
| password |  | SSH root user password |
| user\_id |  | user ID (server owner) |
| diskspace |  | disk space, Mb |
| memory |  | RAM memory, Mb |
| cpu\_units |  | number of CPU units |
| cpus |  | number of allowed CPUs |
| cpu\_limit |  | percent of CPU time |
| expiration\_date |  | expiration date |
| nameserver |  | DNS server |
| search\_domain |  | search domain (for DNS lookup) |
| start\_on\_boot |  | start virtual server after physical server reboot |

## Users ##

### get ###

Detailed information about user.

Request URL:
```
http://<host>/api/users/get?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<user>
  <contact_name>John</contact_name>
  <created_at type="datetime">2011-01-19T16:26:26Z</created_at>
  <email>john@example.com</email>
  <enabled type="boolean">true</enabled>
  <id type="integer">21</id>
  <login>john</login>
  <role_id type="integer">2</role_id>
</user>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | user ID |

### delete ###

Delete user.

Request URL:
```
http://<host>/api/users/delete?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | user ID |

### list ###

List of panel users.

Request URL:
```
http://<host>/api/users/list
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<users type="array">
  <user>
    <contact_name>John</contact_name>
    <created_at type="datetime">2011-01-19T16:26:26Z</created_at>
    <email>john@example.com</email>
    <enabled type="boolean">true</enabled>
    <id type="integer">21</id>
    <login>john</login>
    <role_id type="integer">2</role_id>
  </user>
  <user>
    ...
  </user>
</users>
```

Parameters: absent.

### create ###

Create new user.

Request URL:
```
http://<host>/api/users/create?login=<login>&password=<password>&role_id=<role_id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
  <details>
    <id type="integer">34</id>
  </details>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| login | + | user login |
| password | + | user password |
| role\_id |  | role ID |
| contact\_name |  | contact name |
| email |  | e-mail address |

### enable ###

Enable user (ability to login).

Request URL:
```
http://<host>/api/users/enable?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | user ID |

### disable ###

Disable user (ability to login).

Request URL:
```
http://<host>/api/users/disable?id=<id>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | user ID |

### update ###

Update user attributes.

Request URL:
```
http://<host>/api/users/update?id=<id>&password=<password>
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<result>
  <status type="boolean">true</status>
  <details>
    <id type="integer">34</id>
  </details>
</result>
```

Parameters:
| **Parameter** | **Required** | **Description** |
|:--------------|:-------------|:----------------|
| id | + | user ID |
| password |  | user password |
| role\_id |  | role ID |
| contact\_name |  | contact name |
| email |  | e-mail address |

## Roles ##

### list ###

List of available user roles.

Request URL:
```
http://<host>/api/roles/list
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<roles type="array">
  <role>
    <built_in type="boolean">true</built_in>
    <id type="integer">1</id>
    <name>superadmin</name>
  </role>
  <role>
    ...
  </role>
</roles>
```

Parameters: absent.

## Event Log ##

### list ###

List of lastest events.

Request URL:
```
http://<host>/api/event_log/list
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<events type="array">
  <event>
    <created_at type="datetime">2011-01-30T07:21:24Z</created_at>
    <id type="integer">1945</id>
    <ip_address>192.168.0.11</ip_address>
    <level type="integer">1</level>
    <message>user.updated</message>
    <t_message>User john was updated.</t_message>
  </event>
  <event>
    ...
  </event>
</events>
```

Parameters: absent.

## Tasks ##

### list ###

List of lastest finished and running tasks.

Request URL:
```
http://<host>/api/tasks/list
```

Example of response:
```
<?xml version="1.0" encoding="UTF-8"?>
<tasks type="array">
  <task>
    <description>os_templates.install</description>
    <id type="integer">46</id>
    <status type="integer">0</status>
    <t_description>OS templates installation on the server rock.lan.</t_description>
  </task>
  <task>
    ...
  </task>
</tasks>
```

Parameters: absent.