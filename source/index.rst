dojot documentation
============================

This is the high-level documentation for dojot IoT platform developed by CPqD. This platform
is largely based on [FIWARE](fiware.org), and aims to provide the application and device developers
with a more concise and integrated interaction, while benefiting for the highly customizable and
efficient infrastructure provided by FIWARE.

While based on FIWARE, this platform actually has a large set of components of its own, and interaction
between components was modified to allow better packaging and performance for the solution as a whole.

While this does provide an overall glimpse of the platform, this documentation is not suited for
middleware developers that might want to better understand the components that compose the solution
themselves. For that, please check the component's own documentation repositories and ReadTheDocs
pages.


.. toctree::
   :maxdepth: 2
   :caption: Contents:
   :glob:

   architecture
   ops_guide
   user_guide
   apis
   install/*
   vms/vbox/guide
   faq/faq

.. toctree::
   :maxdepth: 2
   :caption: Security

   security/mutual-authentication
   security/crypto-service


.. Indices and tables
.. ==================
..
.. * :ref:`genindex`
.. * :ref:`modindex`
.. * :ref:`search`
