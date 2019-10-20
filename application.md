# Application
## Introduction

The application needs to be configured in two ways. You have to tell it where to find all services
it needs to remote control and a few settings for said remote control and - in case the services
to be tested are also run via docker-compose - you need to make each docker-compose aware of the
other network(s).

Apart from this, you can play around with php, redis, MySQL settings to finetune your
experience - this will not be part of this manual, at least yet.
                   

## Configuration

### Docker Compose

The SOARCE main application needs to be able to send http requests to the applications and services
under test. If they are orchestrated by docker-compose in a closed network, you will have to make
it known to SOARCE's docker-compose. This can be achieved by copying/renaming the file
`docker-compose.override.yml.dist` to `docker-compose.override.yml` and changing/adding the
network name(s). It could look like this:

```yaml
version: '3.3'

services:
  app-soarce:
    networks:
      mycoolapp_default:
      thisotherapp_default:

networks:
  mycoolapp_default:
    external: true
  thisotherapp_default:
    external: true
```

### soarce.json

You will need to tell SOARCE a list of services, their URLs and a few config parameters, especially
when you don't want to stick with the defaults. Since it would be quite cumbersome to do this via
ENV vars, we decided to place it into a JSON-File that has to go by the name `soarce.json` in this
project's root directory.

```json
{
  "services": {
    "myAwesomeMainApplication": {
      "url": "http://myawesomemainapplication.local/",
      "parameter_name": "SOARCE",
      "common_path": "/var/www/",
      "preshared_secret": "abcdefg"
    },
    "mySneakyBackgroundService": {
      "url": "http://mysneakybackgroundservice.local/",
      "parameter_name": "SOARCE",
      "common_path": "/var/www/",
      "preshared_secret": "qwertyuiop"
    }
  }
}
```

The parameters so far in detail:
* The `key` of the `services` object is arbitrary but should be something you can recognize the
  service under test with by name. It will be used in the database and in the web frontend.
* The `url` parameter is the local network URL pointing to the application or service under test.
  If the server does not run on a standard port, include it, the same has to be done if the
  application is setup in a subdirectory
* The `parameter_name` parameter tells SOARCE to use a certain parameter as key for commands to
  the SOARCE client plugin. In most cases it should be fine to keep the default value `SOARCE`.
  You have to change it, if the application has a parameter `SOARCE` itself. Of course this has
  to match the value in client config.
* The `common_path` parameter is used to strip paths in the web frontend that will be common for
  all requests for that application to reduce visual noise. This is for example the document
  root of the web server or the application root.
* The `preshared_secret` parameter is sent as a HTTP header to the client/plugin as a form of
  authentication. It's value has to be identical in the client's config file.


## Usage

### Collecting Coverage, Traces and Sequences

After everything is configured, you will need to create and activate usecases. If you are using
a test tool like Selenium or Katalon, we recommend to do this in an automated way right with your
tests. If you are manually testing, you can use the web frontend as well.

Navigate to "Control" and then "usecases". You should see a probably empty list and form "Create
New Usecase". Simply enter a name for the usecase and hit the button "Create".

The list above should now have at least one entry. In order for SOARCE to collect the data you
need, there needs to be one active usecase. Incoming data will be associated with that usecase
automatically, so there can be only one active usecase. To activate a use case simply click on
the green button next to it.

You can of course delete a use case or delete all it's data if you for exmaple decide to re-run
the test.

### Analysis

All current views basically show all data by default. You can use filters (top form on the pages)
to narrow down the data displayed to your needs.

Within code coverage view you can click on lines to see which usecases and requests touched the
respective line.


## Known Issues

### Security
Do not use in production and do not place the application on a publicly available server as
people might be able to discover details about your services that are not suitable for the
public. 

### Performance
Do not use xdebug on this application itself, only in the applications and services which are
supposed to be tested need xdebug.

### Database Keys

We use Unique Key Constraints for certain tables so that we don't need transactions, nor limit
writing data to single threads nor spend a lot of CPU time with cleaning up and rewriting
hundreds of thousands of rows. On top of that foreign key constraints with ON DELETE CASCADE
are used to delete all data linked to a usecase when that usecase is reset. This all causes
huge gaps in the primary key sequence.

Because of this we chose quite large integer types. For the time being we recommend to purge
the whole database in case you want to rerun a full test suite. We plan to make writing of
data single threaded and cached through redis in the near future, this should minimize
the problem.
