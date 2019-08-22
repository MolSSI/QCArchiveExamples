==================
QCArchive Examples
==================

These examples demonstrate how to use QCPortal to access data on the QCArchive in a variety of situations. 

All examples are available in your browser through `Binder <https://mybinder.org/v2/gh/MolSSI/QCArchiveExamples/master?urlpath=lab/tree/basic_examples>`_.
Alternatively, you may run these examples locally. To do so, please install QCPortal with either ``conda`` or ``pip``. 

Local Installation
------------------
Install ``qcportal`` using `conda <https://www.anaconda.com/download/>`_::

    conda install qcportal -c conda-forge

or with ``pip``::

    pip install qcportal


.. toctree::
   :maxdepth: 1
   :caption: Basic Examples
    

   basic_examples/getting_started.ipynb
   basic_examples/reaction_datasets.ipynb
   basic_examples/torsiondrive_datasets.ipynb

.. toctree::
   :maxdepth: 1
   :caption: Cookbook


   cookbook/overview.rst
   cookbook/molecules.ipynb
   cookbook/molecule_records.ipynb
   cookbook/dataset_fields.ipynb