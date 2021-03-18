- Feature Name: Simplify Config Customization
- Start Date: 2021-03-18
- RFC PR: [amundsen-io/rfcs#28](https://github.com/amundsen-io/rfcs/pull/28)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Simplify Config Customization

## Summary

Currently customization of API components require defining your own config files, mounting them correctly in docker/k8s and making it avaiable in Amundsen PATH.
Looking on config paths possible in more mature products (like Airflow or Confluent Kafka) we should enable defining the properties of config classes through environment variables.

Note: 
- This does not change the current behaviour, and just implements another way to define config variables
- This RFC does not propose any changes for customization of frontend (custom-config.js)

## Motivation

We should aim for simplicity and removing obstacles when it comes to customizing and deploying Amundsen. Reducing the complexity of customizing and deploying amundsen will 
make the path of onboarding Amundsen much easier. 

## Guide-level Explanation (aka Product Details)

1. Users still can define configs the way before (backwards compatibility)
2. There is a mechanism for collecting amundsen specific configuration from environment variables (overriding configuration is possible without a need to define custom config classes).

For example setting env variables:

```shell script
export AMUNDSEN__METADATA__PROXY_HOST=localhost:21000
```

Would become equivalent of having to:

1. create file `/tmp/configs/amundsen_custom/custom_config.py`:
```python
import os
from amundsen_metadata.config import LocalConfig

class CustomConfig(LocalConfig):
    PROXY_HOST = 'localhost:21000'
```

2. set proper env variables

```bash script
export PATH=$PATH:/tmp/configs/
export METADATA_SVC_CONFIG_CLASS=amundsen_custom.custom_config.CustomConfig
```

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

1. Add `get_config_class(component: string, base_class: Config) -> Config` method to amundsen common
2. Use `get_config_class` in each service, for example:

```python
import importlib
import os

from amundsen.common.config import get_config_class


_config_module_class = \
        os.getenv('METADATA_SVC_CONFIG_MODULE_CLASS') or _config_module_class

config_module = importlib.import_module(''.join(_config_module_class.split('.')[:-1]))
config_class = getattr(config_module, _config_module_class.split('.')[-1])

config_module_class = get_config_class('metadata', config_class)

app.config.from_object(config_module_class())
``` 

Pseudocode:

##### Amundsen Common

```python
import os
from json import loads


def get_config_class(component, base_class):
    prefix = f'AMUNDSEN__{component.upper()}__'

    types_spec = [(str, lambda x: x),
                  (int, lambda x: int(x)),
                  (list, lambda x: x.split(',')),
                  (dict, lambda x: loads(x))]

    class ConfigClass(base_class):
        def __init__(self, *args, **kwargs):
            super(ConfigClass, self).__init__(*args, **kwargs)

            for k, v in dict(os.environ).items():
                if k.startswith(prefix):
                    setting_name = k.split(prefix)[1]

                    try:
                        current_setting_value = getattr(self, setting_name)
                        setting_type = type(current_setting_value)
                    except AttributeError:
                        print(f'You are trying to set not existing configuration setting: {k}')

                        continue

                    new_setting_value = None

                    for spec in types_spec:
                        _type, _function = spec

                        if issubclass(setting_type, _type):
                            new_setting_value = _function(v)

                            break

                    if not new_setting_value:
                        print(setting_name, setting_type)

                        continue

                    setattr(self, setting_name, new_setting_value)

    return ConfigClass
```

##### Amundsen Metadata

```python
import os
from amundsen_common.config import get_config_class


class LocalConfig:
    HOST_URL = 'http://localhost:1234'
    LIST_OF_VARS = []
    INT_SETTING = 0


os.environ['AMUNDSEN__METADATA__HOST_URL'] = 'http://localhost:9999'
os.environ['AMUNDSEN__METADATA__LIST_OF_VARS'] = 'a,b,c,d,e'
os.environ['AMUNDSEN__METADATA__INT_SETTING'] = '2'
os.environ['AMUNDSEN__METADATA__GIGGLES'] = 'http://localhost:9999'

t = get_config_class('metadata', LocalConfig)()

print(t.__dict__)

# Output:
# You are trying to set not existing configuration setting: AMUNDSEN__METADATA__GIGGLES
# {'HOST_URL': 'http://localhost:9999', 'LIST_OF_STH': ['a', 'b', 'c', 'd', 'e'], 'SETTING_INT': 2}
```

## Drawbacks

- customization of options setting fuctom functions (AUTH_USER_METHOD REQUEST_HEADERS_METHOD INIT_CUSTOM_ROUTES):
    - still requires custom file to be present in PATH
    - would need to be explicitly handled by get_config_class

## Alternatives

- Make every config option use os.getenv with default value
    - pros: config will stay more flexible in terms of defining more complex variables (functions, dicts)
    - cons: there is no centralization of parsing logic

- Keep current approach forcing users to provide config files, mounting them and keeping in PATH.

## Prior art

N/A

## Unresolved questions

N/A

## Future possibilities

