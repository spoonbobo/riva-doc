Riva - Rasa - Stories (Dialog Manager)
======================================

Stories (Dialog Manager) are defined to train Dialog Manager model in order to inform Rasa chatbots what to do next (or next next, next next next, ...)

Sample stories.yml
------------------

.. code-block::

    version: '2.0'

    stories:
        - story: happy path
          steps:
              - intent: greet
              - action: utter_greet
              - intent: mood_great
              - action: utter_happy

        - story: sad path 1
          steps:
              - intent: greet
              - action: utter_greet
              - intent: mood_unhappy
              - action: utter_cheer_up
              - action: utter_did_that_help
              - intent: affirm
              - action: utter_happy

        - story: sad path 2
          steps:
              - intent: greet
              - action: utter_greet
              - intent: mood_unhappy
              - action: utter_cheer_up
              - action: utter_did_that_help
              - intent: deny
              - action: utter_goodbye
