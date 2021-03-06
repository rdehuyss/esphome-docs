RF Bridge Component
===================

.. seo::
    :description: Instructions for setting up the RF Bridge in ESPHome.
    :image: rf_bridge.jpg
    :keywords: RF Bridge

The ``RF Bridge`` Component provides the ability to send and receive 433MHz remote codes without hardware
hacking the circuit board to bypass the ``efm8bb1`` MCU. This component implements the communcation protocol
that the original ``efm8bb1`` firmware implements. The device is connected via the
:doc:`UART bus </components/uart>`. The uart bus must be configured at the same speed of the module
which is 19200bps.

.. warning::

    If you are using the :doc:`logger` make sure you disable the uart logging with the
    ``baud_rate: 0`` option.

.. figure:: images/rf_bridge-full.jpg
    :align: center
    :width: 60.0%

.. code-block:: yaml

    # Example configuration entry
    uart:
      baud_rate: 19200

    rf_bridge:
      on_code_received:
        - homeassistant.event:
            event: esphome.rf_code_received
            data:
              sync: !lambda 'char buffer [10];return itoa(data.sync,buffer,16);'
              low: !lambda 'char buffer [10];return itoa(data.low,buffer,16);'
              high: !lambda 'char buffer [10];return itoa(data.high,buffer,16);'
              code: !lambda 'char buffer [10];return itoa(data.code,buffer,16);'

Configuration variables:
------------------------

- **uart_id** (*Optional*, :ref:`config-id`): Manually specify the ID of the UART hub.
- **id** (*Optional*, :ref:`config-id`): Manually specify the ID used for code generation.
- **on_code_received** (*Optional*, :ref:`Automation <automation>`): An action to be
  performed when a code is received. See :ref:`rf_bridge-on_code_received`.

.. _rf_bridge-on_code_received:

``on_code_received`` Trigger
----------------------------

With this configuration option you can write complex automations whenever a code is
received. To use the code, use a :ref:`lambda <config-lambda>` template, the code
and the corresponding protocol timings are available inside that lambda under the
variables named ``code``, ``sync``, ``high`` and ``low``.

.. code-block:: yaml

    on_code_received:
      - homeassistant.event:
          event: esphome.rf_code_received
          data:
            sync: !lambda 'char buffer [10];return itoa(data.sync,buffer,16);'
            low: !lambda 'char buffer [10];return itoa(data.low,buffer,16);'
            high: !lambda 'char buffer [10];return itoa(data.high,buffer,16);'
            code: !lambda 'char buffer [10];return itoa(data.code,buffer,16);'


.. _rf_bridge-send_code_action:

``rf_bridge.send_code`` Action
------------------------------

Send an RF code using this action in automations.

.. code-block:: yaml

    on_...:
      then:
        - rf_bridge.send_code:
            sync: 0x700
            low: 0x800
            high: 0x1000
            code: 0xABC123

Configuration options:

- **sync** (**Required**, int, :ref:`templatable <config-templatable>`): RF Sync timing
- **low** (**Required**, int, :ref:`templatable <config-templatable>`): RF Low timing
- **high** (**Required**, int, :ref:`templatable <config-templatable>`): RF high timing
- **code** (**Required**, int, :ref:`templatable <config-templatable>`): RF code
- **id** (*Optional*, :ref:`config-id`): Manually specify the ID of the RF Bridge if you have multiple components.

.. note::

    This action can also be written in :ref:`lambdas <config-lambda>`:

    .. code-block:: cpp

        id(rf_bridge).send_code(0x700, 0x800, 0x1000, 0xABC123);


.. _rf_bridge-learn_action:

``rf_bridge.learn`` Action
--------------------------

Tell the RF Bridge to learn new protocol timings using this action in automations.
A new code with timings will be returned to :ref:`rf_bridge-on_code_received`

.. code-block:: yaml

    on_...:
      then:
        - rf_bridge.learn

Configuration options:

- **id** (*Optional*, :ref:`config-id`): Manually specify the ID of the RF Bridge if you have multiple components.

.. note::

    This action can also be written in :ref:`lambdas <config-lambda>`:

    .. code-block:: cpp

        id(rf_bridge).learn();


Getting started with Home Assistant
-----------------------------------

The following code will get you up and running with a configuration sending codes to
Home Assistant as events and will also setup a service so you can send codes with your RF Bridge.

.. code-block:: yaml

    api:
      services:
        - service: send_rf_code
          variables:
            sync: int
            low: int
            high: int
            code: int
          then:
            - rf_bridge.send_code:
                sync: !lambda 'return sync;'
                low: !lambda 'return low;'
                high: !lambda 'return high;'
                code: !lambda 'return code;'
        - service: learn
          then:
            - rf_bridge.learn

    uart:
      tx_pin: 1
      rx_pin: 3
      baud_rate: 19200

    logger:
      baud_rate: 0

    rf_bridge:
      on_code_received:
        then:
          - homeassistant.event:
              event: esphome.rf_code_received
              data:
                sync: !lambda 'char buffer [10];return itoa(data.sync,buffer,16);'
                low: !lambda 'char buffer [10];return itoa(data.low,buffer,16);'
                high: !lambda 'char buffer [10];return itoa(data.high,buffer,16);'
                code: !lambda 'char buffer [10];return itoa(data.code,buffer,16);'


Now your latest received code will be in an event.

To trigger the automation from Home Assistant you can invoke the service with this code:

.. code-block:: yaml

    automation:
      # ...
      action:
      - service: esphome.rf_bridge_send_rf_code
        data:
          sync: 0x700
          low: 0x800
          high: 0x1000
          code: 0xABC123

See Also
--------

- :apiref:`rf_bridge/rf_bridge.h`
- :doc:`/components/uart`
- :ghedit:`Edit`
