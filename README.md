figgypy
=======

[![Chat on Gitter](https://badges.gitter.im/theherk/figgypy.svg)](https://gitter.im/theherk/figgypy?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Build Status](https://travis-ci.org/theherk/figgypy.svg)](https://travis-ci.org/theherk/figgypy)

A simple configuration parser.

Installation
------------

    pip install figgypy

_note_ - figgypy requires python-gnupg and gnupg to decode secrets. It will install python-gnupg at install time. If you don't have gnupg on your system by default (it probably is) you'll need to install it. If either of these two are missing, the configuration tool will still work, it just won't decrypt secrets.

Usage
-----

    from figgypy import Config

    cfg = Config(conf_file)

Config object can be created with a filename only, relative path, or absolute path.
If only name or relative path is provided, look in this order:

1. current directory
2. `~/.config/<file_name>`
3. `/etc/<file_name>`

It is a good idea to include you `__package__` in the file name.
For example, `cfg = Config(os.path.join(__package__, 'config.yaml'))`.
This way it will look for `your_package/config.yaml`,
`~/.config/your_package/config.yaml`, and `/etc/your_package/config.yaml`.

This will create a `cfg` variable with attributes for each top level item in the configuration file. Each attribute will be a dictionary with the remaining nested structure.

The configuration file currently supports json, _xml*_, and yaml.

_* note_ - xml will work, but since it requires having only one root, all of the configuration will be in a dictionary named that root. See examples below.

Examples
--------

### json

```json
{
    "db": {
        "url": "mydburl.com",
        "name": "mydbname",
        "user": "myusername",
        "pass": "correcthorsebatterystable"
    },
    "log": {
        "file": "/var/log/cool_project.log",
        "level": "INFO"
    }
}
```

    cfg = Config('theabove.json')

This yields object `cfg` with attributes `db` and `log`, each of which are dictionaries.

### xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<config>
    <db>
        <url>mydburl.com</url>
        <name>mydbname</name>
        <user>myusername</user>
        <pass>correcthorsebatterystable</pass>
    </db>
    <log>
        <file>/var/log/cool_project.log</file>
        <level>INFO</level>
    </log>
</config>
```

    cfg = Config('theabove.xml')

This yields object `cfg` with attribute `config`, which is the complete dictionary.

### yaml

```yaml
db:
  url: mydburl.com
  name: mydbname
  user: myusername
  pass: correcthorsebatterystable
log:
  file: /var/log/cool_project.log
  level: INFO
```

    cfg = Config('theabove.yaml')

This yields object `cfg` with attributes `db` and `log`, each of which are dictionaries. This is the exact same behaviour as json, which makes sense given the close relationship of yaml and json.

Secrets
--------
It is possible to use gpg to store PGP and KMS encrypted secrets in a config file.

```yaml
db:
  host: db.heck.ya
  pass: |
    -----BEGIN PGP MESSAGE-----
    Version: GnuPG v2

    hQIMAzf92ZrOUZL3ARAAgWexav8+pc2lnqISEuQafFZrqYI0pU3xCuMXnFZp+hpU
    gb0LsaExZ136p4ATIinFHuaLt94hFx7gULgqoSigt/2fubnUCsOGedq122xYZdtV
    Ep/24WPVQPcMVIP9pDTJTk82A41BQsOrVYorAGjjB13zFizizYHApNTcWKr4/gfR
    jmCqAX5qusXB84fXBecCJ886uEQI2v7+Vxnk+fQMqNt3ybd/uLuBLShMSygr6uLX
    zktyeZvP2QqPSWe0OpttdcvD792/SI/CTznsjbMe0wr1L81csEQcj++4o5wJop3Y
    mbQvG/FxeDdRi2aCxh7JK2xdCsrQzXKTNG2QZMwWqatB5Lb6lJ1mNiJQGX2YK+nI
    lbjy5Cp2lHlNxa9QfB+KglueMnH9gDku5YqBDos6rCEuqK/aTDdMx0V7YGYTamZ3
    3Za+OGi+hl/+4WX2gm+bOM2WWrIysiu9k1HMI1/onui/3hr1nClR8rGb4a5qDlpg
    yRrt7LuLRU4vGXpYm05dXlUeI3uT04ur/DwLo32ujnPo3dc8LFegX8N8p1LLS9vq
    vvrvXRnWsgeAvAYFBprbEYcz7sOU04HM9OGcyjYREMs3Ih6H2oBi3GavJ2x0MG75
    M9JSTu/yytD8GCM3s+3RncKuEAxfZIk1Gbdz0pjb+U6G43qq8/vQPKtKuAeqJHDS
    SAER9YkKqbp0y85LbhUWNWPpHQ2zy8WB71TfYE6vBP5qjoxiqP/QGWjT/3jhCY+t
    5k7R6XqvdvbSu1avFlEgApknzn94I+gsWQ==
    =QuDe
    -----END PGP MESSAGE-----
```

If you are using json, you'll need newlines. I achieved the following example with `cat the_above.yaml | seria -j -`.

```json
{
    "db": {
        "host": "db.heck.ya",
        "pass": "-----BEGIN PGP MESSAGE-----\nVersion: GnuPG v2\n\nhQIMAzf92ZrOUZL3ARAAgWexav8+pc2lnqISEuQafFZrqYI0pU3xCuMXnFZp+hpU\ngb0LsaExZ136p4ATIinFHuaLt94hFx7gULgqoSigt/2fubnUCsOGedq122xYZdtV\nEp/24WPVQPcMVIP9pDTJTk82A41BQsOrVYorAGjjB13zFizizYHApNTcWKr4/gfR\njmCqAX5qusXB84fXBecCJ886uEQI2v7+Vxnk+fQMqNt3ybd/uLuBLShMSygr6uLX\nzktyeZvP2QqPSWe0OpttdcvD792/SI/CTznsjbMe0wr1L81csEQcj++4o5wJop3Y\nmbQvG/FxeDdRi2aCxh7JK2xdCsrQzXKTNG2QZMwWqatB5Lb6lJ1mNiJQGX2YK+nI\nlbjy5Cp2lHlNxa9QfB+KglueMnH9gDku5YqBDos6rCEuqK/aTDdMx0V7YGYTamZ3\n3Za+OGi+hl/+4WX2gm+bOM2WWrIysiu9k1HMI1/onui/3hr1nClR8rGb4a5qDlpg\nyRrt7LuLRU4vGXpYm05dXlUeI3uT04ur/DwLo32ujnPo3dc8LFegX8N8p1LLS9vq\nvvrvXRnWsgeAvAYFBprbEYcz7sOU04HM9OGcyjYREMs3Ih6H2oBi3GavJ2x0MG75\nM9JSTu/yytD8GCM3s+3RncKuEAxfZIk1Gbdz0pjb+U6G43qq8/vQPKtKuAeqJHDS\nSAER9YkKqbp0y85LbhUWNWPpHQ2zy8WB71TfYE6vBP5qjoxiqP/QGWjT/3jhCY+t\n5k7R6XqvdvbSu1avFlEgApknzn94I+gsWQ==\n=QuDe\n-----END PGP MESSAGE-----"
    }
}
```

To store a KMS secret, just add the `_kms` key to the configuration file.

```yaml
db:
  host: db.heck.ya
  pass:
    _kms: your KMS encrypted value
```

See [below](#kms) for instructions on generating this value.

That's easy, right? Now this value will be decrypted and available just like you had typed in the value in the configuration file.

### Environment Variables

+ `FIGGYPY_GPG_BINARY` For specifying where GPG is. Defaults to `gpg`.
+ `FIGGYPY_GPG_HOMEDIR` The GPG home. Basically where to look for the keyring. Defaults to `~/.gnupg/`.
+ `FIGGYPY_GPG_KEYRING` The file that houses the keys. Defaults to `pubring.gpg`; may need to be `pubring.kbx`.

AWS configuration uses the standard boto3 configuration, but can also be passed in explicitly. (see below)

### Passed in parameters

These can also be passed in as arguments when initializing.

```python
aws_config = {'aws_access_key_id': aws_access_key_id,
              'aws_secret_access_key': aws_secret_access_key,
              'region_name': 'us-east-1'}
gpg_config = {'homedir': 'noplace/like/home',
              'keyring': 'pubring.kbx'}
cfg = figgypy.Config('config.yaml', aws_config=aws_config, gpg_config=gpg_config)
```

### To encrypt a value

#### GPG

    echo -n "Your super secret password" | gpg --encrypt --armor -r KEY_ID

Add the resulting armor to your configuration where necessary. If you are using yaml, this is very simple. Here is an example:

#### KMS

    aws kms encrypt --key-id 'alias/your-key' --plaintext "your secret" --query CiphertextBlob --output text

or the preferred method:

```python
from figgypy.utils import kms_encrypt
encrypted = kms_encrypt('your secret', 'key or alias/key-alias', optional_aws_config)
```

Thanks
------

This tool uses [Seria](https://github.com/rtluckie/seria) to serialize between supported formats. Seria is a great tool if you want convert json, xml, or yaml to another of the same three formats.
