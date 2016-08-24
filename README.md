# CMservice
A webservice for collecting and storing end-user consent of attributes.

# Install dependencies
To install all necessary dependencies, run `python setup.py install` in the root directory.

# Run CMservice 

Copy the **settings.cfg.example** and rename the copy **settings.cfg**. At the moment it's not 
possible to name the configuration anything other then **settings.cfg**. If needed modify the 
necessary configurations. 

```shell
CMSERVICE_CONFIG=/Users/regu0004/dev/consent_service/server/settings.cfg gunicorn cmservice.service.run:app
```

# Configuration
| Parameter name | Data type | Example value | Description |
| -------------- | --------- | ------------- | ----------- |
| SSL | boolean | True | Should the server use https or not |
| SERVER_CERT | String | "./keys/server.crt" | The path to the certificate file used by SSL comunication |
| SERVER_KEY | String | "./keys/server.key" | The path to the key file used by SSL comunication |
| JWT_PUB_KEY | List of strings | ["./keys/mykey.pub"] | A list of signature verification keys |
| SECRET_KEY | String | "t3ijtgglok432jtgerfd" | A random value used by cryptographic components to for example to sign the session cookie |
| PORT | Integer | 8166 | Port on which the CMservice should start |
| HOST | String | "127.0.0.1" | The IP-address on which the CMservice should run |
| DEBUG | boolean | False | Turn on or off the Flask servers internal debuggin, should be turned off to ensure that all log information get stored in the log file |
| TICKET_TTL | Integer | 600 | For how many seconds the ticket should be valid |
| CONSENT_DATABASE_CLASS_PATH | String | "cmservice.database.SQLite3ConsentDB" | Specifies which python database class the CMservice should use. Currently there exists two modules `DictConsentDB` and `SQLite3ConsentDB` |
| CONSENT_DATABASE_CLASS_PARAMETERS | List of strings | ["test.db"] | Input parameters which should be passed into the database class specified above. SQLite3ConsentDB needs a single parameter, a path where the database should be stored. DictConsentDB does not take any parameters so [] should be specified |
| AUTO_SELECT_ATTRIBUTES | boolean | True | Specifies if all the attributes in the GUI should be selected or not |
| MAX_CONSENT_EXPIRATION_MONTH | Integer | 12 | The maximum numbers of months a consent could be valid |
| USER_CONSENT_EXPIRATION_MONTH | List of integers | [3, 6] | A list of alternatives for how many months a user wants to give consent |
| LOGGING_FILE | String | "cmservice.log" | A path to the log file, if none exists it will be created |
| LOGGING_LEVEL | String | "WARNING" | Which logging level the application should use. Possible values: INFO, DEBUG, WARNING, ERROR and CRITICAL |
| CONSENT_REQUEST_DATABASE_CLASS_PATH | string | "cmservice.database.DictConsentRequestDB" | Specifies which python database class the CMservice should use. Currently there exists two modules `DictConsentRequestDB` and `SQLite3ConsentRequestDB` |
| CONSENT_REQUEST_DATABASE_CLASS_PARAMETERS | list of strings | [] | Input parameters which should be passed into the database class specified above. `SQLite3ConsentRequestDB` needs a single parameter, a path where the database should be stored. `DictConsentRequestDB` does not take any parameters so [] should be specified |
| CONSENT_SALT | String | "VFT0yZ" | A SALT used to hash the consent ID before stroed in the database |

# Storage
Some information has to be stored in order for the CMservice to work

## Database alternatives
Currently the consent could be stored in either in memory database or in a 
SQLite database. Please read the "Configuration" section 
for more information on how to switch between the different database instances.

## Stored infomation


### Consent database
| Database column | Description |
| --------------- | ----------- |
| consent_id | An unique id for a consent, generated by the client. Before the consent ID is strored in the database a SALT is concatinated with the consent id recived from the client and it is then hashed |
| timestamp | Time for when the consent where given. |
| month | For how many months the consent where given. After that particular date the user needs to give consent again. |
| attributes | All the attributes for which consent where given. Note that it's only for which attributes consent where given and not the values. |
| question_hash | The consent question sent by the client. It's a hash over who sent the original request, the consent_id, the selected attributes and values |

### Ticket database
| Database column | Description |
| --------------- | ----------- |
| Ticket | A identifier for the ticket |
| Timestamp | The time then the ticket where created |
| TicketData | The unpacked original consent request which where sent as a singed JWT. The request contains user attributes and values in plain text, a user ID a redirect URL, information about the service provider making the request |


# Development

## i18n

To extract all i18n string:

```bash
python setup.py extract_messages --input-paths src/cmservice/ --output-file src/cmservice/service/data/i18n/messages.pot
```

To compile .po->.mo:

```bash
python setup.py compile_catalog --directory src/cmservice/service/data/i18n/locales/
```


See [Babel docs](http://babel.pocoo.org/en/latest/setup.html) for more info.