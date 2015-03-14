# Introduction #

Panel has environment checker, which helps to detect common problems with panel installation.

# Details #

Environment checker is available at http://{your-host}/check/env address. But due to security reasons it is available only in debug mode. So you sould have the following lines inside {panel-files-root}/application/config.ini:
```
[debug]
enabled = on
```

Sometimes it's helpful to check PHP configuration in details. To get fast and easy access to phpinfo function output you can look at http://{your-host}/check/phpinfo