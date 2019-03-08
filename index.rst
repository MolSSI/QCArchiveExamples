=========
QCArchive
=========

The core of this project sets out to answer the fundamental question of "How do
we compile, aggregate, query, and share quantum chemistry data to accelerate
the understanding of new method performance, fitting of novel force fields, and
supporting the incredible data needs of machine learning for computational
molecular science?".

Benchmark Datasets
------------------

Often, benchmark datasets are found in an assortment of CSV, PDF, and raw ASCII
files. This makes aggregating (or even using) these benchmark sets quite the
onerous task when they are core to our fundamental understanding of more
popular and approximate methods.

Forcefield Construction
-----------------------

Forcefield construction requires large numbers of computations that elucidate
bond, angle, dihedral, electrostatic potential, polarizabilities, and
other quantites. Computation of these values is often done in a bespoke manner
where computations are generated for a single molecule and then discarded.
Keeping these results will assist in building a large repository of data to
query rather than to continue to generate.


Machine Learning
----------------

The machine learning explosion has made its way into quantum chemistry,
materials science, and biomolecular simulation. At the heart of all machine
learning projects is the generation of large, high-quality datasets, and rapid
progress in the field (following work in other areas of machine learning)
requires the ability to share and readily access standard high-quality
datasets. The infrastructure for doing this, however, does not yet exist.
MolSSI is in a unique position to build an easy-to-use data backend to
accelerate progress and prevent fragmentation in this burgeoning field.

****

**Getting Started**

* :doc:`install`
* :doc:`community`
* :doc:`roadmap`

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Getting Started

   install
   community
   roadmap

****

**Ecosystem**

* :doc:`qcarchive_overview`
* :doc:`use_cases`

.. toctree::
   :maxdepth: 1
   :hidden:
   :caption: Ecosystem

   use_cases
   qcarchive_overview

****

**Basic Examples**

.. toctree::
   :maxdepth: 1
   :caption: Basic Examples

   basic_examples/getting_started.ipynb
