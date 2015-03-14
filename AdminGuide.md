# Introduction #

OpenVZ Web Panel is a panel for control servers with OpenVZ virtualization technology using web browser only. Panel allows to install OS templates, create and manage virtual servers, fine tune server settings, grant access for particular users to selected virtual servers, manage several physical servers at ones.

# Dashboard #

Dashboard is a first page shown after login. It contains information about logged user and statistics of panel usage.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide1.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide1.png)

# OS Templates #

Before creating virtual servers need to have OS templates installed on physical server. It easy to install new OS templates using panel. Need to select physical server (localhost by default), go to OS templates list and click "Install New OS Template" button.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide2.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide2.png)

After some time OS templates will be installed on the server (depending on size of templates). Refresh of OS templates list should show them.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide3.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide3.png)

# Virtual Servers #

Virtual servers creation will be available if OS templates present on the server. Creation form is shown below:

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide4.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide4.png)

Need to select VEID, OS and server template, IP address (or addresses) and root password. Also it's possible to define diskspace and RAM on "Advanced settings" tab like some other additional parameters.

Several IP addresses or DNS servers can be added separating them by spaces.

Quality of service limits can be changed later. By default they will be copied from server template.

# Server Templates #

Server templates are used as a source of quality of service limits during virtual server creation. They could be used to simplify new virtual servers creation for different purposes.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide5.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide5.png)

It's possible to create, edit and remove templates. One of templates is a default template for physical server and can't be removed.

# Physical Servers #

Physical servers page contains the list of all connected physical servers. There is localhost by default if panel was installed on physical server with OpenVZ. Also it's possible to connect more physical servers without need to install panel there.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide6.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide6.png)

New virtual servers or OS templates can be installed directly on the server using command line tools. In such case need to select physical server and click "Synchronization" button to perform synchronization between panel's database and actual state of the server.

# Users #

Panel has users & roles management.

It's possible to create new users and assign roles to them. By default two roles present right after installation: infrastructure admin and virtual server owner.

Any user can become the owner of particular virtual server. This action is performed during virtual server creation or editing on "Main settings" tab by selecting owner inside drop down.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide7.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide7.png)

# Events Logs #

Most of actions are tracked in the events log. Each record has a status: success or failure. Need to check logs if something goes wrong or panel's behavior looks like inadequate.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide8.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide8.png)

# Tasks #

Long running tasks execution is tracked at Tasks page. Examples of such tasks are OS templates installation or backup creation. Currently active tasks have a spinner icon in status field. Finished tasks have success of failure icons according to result.

![http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide9.png](http://ovz-web-panel.googlecode.com/svn/wiki/guides/guide9.png)