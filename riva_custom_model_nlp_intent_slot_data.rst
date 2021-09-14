Riva - Custom Model - NeMo - Data Format - NLP (Intent-slot)
============================================================

It's required to convert data into specific NeMo format when training NLP models for *Joint Intent and Slot Classification*. The model training requires necessary input files for *intent* training and *slot* training (:file:`dict.intents.csv`, :file:`dict.slots.csv`, :file:`train.tsv`, :file:`train_slots.tsv`, :file:`test.tsv`, :file:`test_slots.tsv`) which are expected to store in a directory. The path of data directory needs to be updated in the training config file.

Expected structure
------------------

.. code-block::
	
	|--<target_data_dir>
		|-- dict.intents.csv
		|-- dict.slots.csv
		|-- train.tsv
		|-- train_slots.tsv
		|-- test.tsv
		|-- test_slots.tsv

Sample dict.intents.csv
-----------------------

Suppose we have N intents

.. code-block::

	#intent #label
	interest	# 0
	transportation	# 1
	sleep		# 2
	...
	intentN 	# N-1

Sample dict.slots.csv
---------------------

Suppose we have N slots

.. code-block::

	#slot	#label
	interest_organisation	# 0
	transportation_mathod	# 1
	sleep_bahaviour		# 2
	...
	O 			# N-1

Sample train.tsv
----------------

For every first line of any :file:`train.tsv`, we have a header line

.. code-block::

	sentence <tab> label

and, for rest of the lines follows :code:`<sentence> <tab> <label>` where :code:`<label>` refers to label in :file:`dict.intents.csv`

.. code-block:: bash

	I love NVIDIA <tab> <label>
	I go to company by bus <tab> <label>
	I sleep late everyday <tab> <label>
	...

Sample train_slots.tsv
----------------------

There is no header line for any :file:`train_slots.tsv`. For each line of the file, each token of the line is presented a label to slots (defined in :file:`dict.slots.csv`).

.. code-block::

	# there is space between labels
	N-1 N-1 0
	N-1 N-1 N-1 N-1 N-1 1
	N-1 N-1 2 N-1
	...

Sample test.tsv
---------------

Similar with :file:`train.tsv`

Sample test_slots.tsv
---------------------

Similar with :file:`train_slots.tsv`

.. seealso::

	* `Joint_Intent_and_Slot_Classification.ipynb <https://colab.research.google.com/github/NVIDIA/NeMo/blob/stable/tutorials/nlp/Joint_Intent_and_Slot_Classification.ipynb>`_