# Nameko eventlog dispatcher


[Nameko](http://nameko.readthedocs.org) dependency provider that
dispatches log data using ``Events`` (Pub-Sub).



## Usage


### Dispatching event log data


Include the ``EventLogDispatcher`` dependency in your service class:

```python
from nameko.rpc import rpc
from nameko_eventlog_dispatcher import EventLogDispatcher


class FooService:

    name = 'foo'

    eventlog_dispatcher = EventLogDispatcher()

    @rpc
    def foo_method(self):
        self.eventlog_dispatcher(
          'foo_event_type', {'value': 1}, metadata={'meta': 2}
        )
```

``event_type``, ``event_data`` (optional) and ``metadata`` (optional)
can be provided as arguments. Both ``event_data`` and ``metadata`` must
be dictionaries and contain JSON serializable data.

Calling ``foo_method`` will dispatch an event from the ``foo`` service
with ``log_event`` as the event type. However ``foo_event_type`` will be
the event type stored as part of the event metadata.

Then, any nameko service will be able to handle this event.

```python
from nameko.events import event_handler


class BarService:

    name = 'bar'

    @event_handler('foo', 'log_event')
    def foo_log_event_handler(self, body):
        """`body` will contain the event log data."""
```


### Capturing log data when entrypoints are fired


Enable auto capture event logs in your nameko configuration file:

```yaml
    # config.yaml

    EVENTLOG_DISPATCHER:
      auto_capture: true
      entrypoints_to_exclude: []
      event_type: log_event
```

All the attributes above are optional and only used to override their
default values.

With ``auto_capture`` set to ``true``, a nameko event will be dispatched
every time an entrypoint is fired:

- They can also be handled by listening ``log_event`` events from the
  service dispatching them.
- ``entrypoint_fired`` will be the event type stored as part of the
  event metadata.
- Only entrypoints listed in the ``ENTRYPOINT_TYPES_TO_LOG`` class
  attribute will be logged.
- ``entrypoints_to_exclude`` can be used to provide a list of entrypoint
  method names to exclude when firing events automatically.

``event_type`` can be added to the config to override the default nameko
event type used to dispatch this kind of events.

## Format of the event log data


This is the format of the event log data:

```python
    {
      "entrypoint_name": "foo_method",
      "service_name": "foo",
      "timestamp": "2017-06-12T13:48:16+00:00",
      "event_type": "foo_event_type",  # "entrypoint_fired", ...
      "data": {},
      "call_stack": [
        "standalone_rpc_proxy.call.3f349ea4-ed3e-4a3b-93d0-a36fbf928ecb",
        "bla.bla_method.21d623b4-edc4-4232-9957-4fad72533b75",
        "foo.foo_method.d7e907ee-9425-48a6-84e6-89db19e3ce50"
      ],
      "entrypoint_protocol": "Rpc",
      "call_id": "foo.foo_method.d7e907ee-9425-48a6-84e6-89db19e3ce50"
    }
```

The ``data`` attribute will contain the event data that was provided as
an argument for the ``event_data`` parameter when dispatching the event.


## Tests


It is assumed that RabbitMQ is up and running on the default URL
``guest:guest@localhost`` and uses the default ports.

```bash
    $ make test
    $ make coverage
```

A different RabbitMQ URI can be provided overriding the following
environment variables: ``RABBIT_CTL_URI`` and ``AMQP_URI``.

Additional ``pytest`` parameters can be also provided using the ``ARGS``
variable.

```bash
$ make test RABBIT_CTL_URI=http://guest:guest@dockermachine:15673 AMQP_URI=amqp://guest:guest@dockermachine:5673 ARGS='-x -vv --disable-pytest-warnings'
$ make coverage RABBIT_CTL_URI=http://guest:guest@dockermachine:15673 AMQP_URI=amqp://guest:guest@dockermachine:5673 ARGS='-x -vv --disable-pytest-warnings'
```
