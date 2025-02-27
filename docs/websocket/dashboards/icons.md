# Icons

## Entity icons

If you have fetched a dashboard (using something like `{"id":4,"type":"lovelace/config","url_path":"dashboard-stuff"}`), you might soon discover that most entities do not contain an icon. This appears to be because entity icons are usually inherited - except when set explicitly by the user.

"Inherited" icon information needs to be fetched separately.

The priorities for assigning icons appear to be as follows:

1. an `icon` attribute on the entity (tends to be `mdi:something`)
2. an `entity_picture` attribute on the entity (tends to *not* be `mdi:something`)
3. a default from the [Entity registry](#entity-registry) on platform+translation key
4. a default from the [Icon registry](#icon-registry) on domain+device_class

On top of that, there can be state-dependent icons. (FIXME: put example in)

# Entity registry

First, ask for `config/entity_registry/list_for_display`.
This yields something like (many entities omitted here):

```json
{
  "id": 4,
  "result": {
    "entities": [
      {
        "di": "c387a39656f2b0c6ca2fac9e96cd2e02",
        "ec": 1,
        "ei": "sensor.sun_next_dawn",
        "en": "Next dawn",
        "hn": true,
        "lb": [],
        "pl": "sun",
        "tk": "next_dawn"
      },
      {
        "di": "c387a39656f2b0c6ca2fac9e96cd2e02",
        "ec": 1,
        "ei": "sensor.sun_next_dusk",
        "en": "Next dusk",
        "hn": true,
        "lb": [],
        "pl": "sun",
        "tk": "next_dusk"
      },
      {
        "ei": "person.voorkant",
        "en": "voorkant",
        "lb": [],
        "pl": "person"
      },
      {
        "di": "e06293557a341d6f8e5d088af9075f78",
        "ei": "sensor.reistijd_naar_x",
        "en": "reistijd naar x",
        "lb": [],
        "pl": "waze_travel_time",
        "tk": "waze_travel_time"
      }
    ],
    "entity_categories": {
      "0": "config",
      "1": "diagnostic"
    }
  },
  "success": true,
  "type": "result"
}
```

From [`homeassistant/helpers/entity_registry.py` in homeassistant-core](https://github.com/home-assistant/core/blob/24fbc366a68a73959d20992b1c4ce782c2fa8e44/homeassistant/helpers/entity_registry.py#L150) we can learn about a few of those keys:

```
    ("ai", "area_id", False),
    ("lb", "labels", True),
    ("di", "device_id", False),
    ("ic", "icon", False),
    ("tk", "translation_key", False),
```

Further down into the file, we learn `ei` is `entity_id` and `pl` is `platform`.

For reference, a state for the `waze_travel_time` platform might look like this (edited for privacy):

```json
  "attributes": {
    "attribution": "Powered by Waze",
    "destination": "xx",
    "device_class": "duration",
    "distance": 49.697,
    "duration": 31.1,
    "friendly_name": "reistijd naar x",
    "origin": "xxx",
    "route": "xxxx",
    "state_class": "measurement",
    "unit_of_measurement": "min"
  },
  "context": {
    "id": "01JE4H00W6X0QVSK6T2JQV1Z04",
    "parent_id": null,
    "user_id": null
  },
  "entity_id": "sensor.reistijd_naar_x",
  "last_changed": "2024-12-02T18:09:09.781158+00:00",
  "last_reported": "2024-12-02T20:29:08.358965+00:00",
  "last_updated": "2024-12-02T20:29:08.358965+00:00",
  "state": "31"
}

```

Note the lack of icon.
Here's a state with an explicit icon:

```json
{
  "attributes": {
    "attribution": "Data provided by Frank Energie",
    "friendly_name": "Average electricity price today",
    "icon": "mdi:currency-eur",
    "state_class": "measurement",
    "unit_of_measurement": "€/kWh"
  },
  "context": {
    "id": "01JDYJG6VE9NAXRP6HSS6T7GWV",
    "parent_id": null,
    "user_id": null
  },
  "entity_id": "sensor.average_electricity_price_today",
  "last_changed": "2024-11-30T13:00:00.750150+00:00",
  "last_reported": "2024-11-30T20:00:00.764866+00:00",
  "last_updated": "2024-11-30T13:00:00.750150+00:00",
  "state": "0.27983"
}
```

But for the `wave_travel_time` (`sensor.reistijd_naar_x`) entity, we shall need to do more work to find an icon.
From the entity registry, we have learned that for that entity, the platform is `waze_travel_time` and the translation key is (also, but this is not always so) `waze_travel_time`.
It appears that the translation key is useful as a key for other things too.

# Icon registry

We send `{"category":"entity","id":4,"integration":"waze_travel_time","type":"frontend/get_icons"}`.

```json
{
  "id": 4,
  "result": {
    "resources": {
      "waze_travel_time": {
        "sensor": {
          "waze_travel_time": {
            "default": "mdi:car"
          }
        }
      }
    }
  },
  "success": true,
  "type": "result"
}
```

So, the default icon for platform `waze_travel_time`, sensor `waze_travel_time` is `mdi:car`.

Note that you can leave out `integration` to get icons for all relevant integrations.

For a platform with multiple sensors/sensor types, here's the response to `{"category":"entity","id":4,"integration":"sun","type":"frontend/get_icons"}`

```json
{
  "id": 4,
  "result": {
    "resources": {
      "sun": {
        "sensor": {
          "next_dawn": {
            "default": "mdi:sun-clock"
          },
          "next_dusk": {
            "default": "mdi:sun-clock"
          },
          "next_midnight": {
            "default": "mdi:sun-clock"
          },
          "next_noon": {
            "default": "mdi:sun-clock"
          },
          "next_rising": {
            "default": "mdi:sun-clock"
          },
          "next_setting": {
            "default": "mdi:sun-clock"
          },
          "solar_azimuth": {
            "default": "mdi:sun-angle"
          },
          "solar_elevation": {
            "default": "mdi:theme-light-dark"
          },
          "solar_rising": {
            "default": "mdi:sun-clock"
          }
        }
      }
    }
  },
  "success": true,
  "type": "result"
}
```

And so, the default icon for `next_dusk` is `mdi:sun-clock`.
