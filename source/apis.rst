APIs
===========

These are all APIs exposed by dojot.

============= =============================== ===================================================================== ================
 Endpoint         Purpose                         Component                                                             Repository
============= =============================== ===================================================================== ================
 /device       Device management               `DeviceManager <https://dojot.github.io/device-manager/apis.html>`_   `GitHub - device-manager <https://github.com/dojot/device-manager>`_ 
 /template     Template management             `DeviceManager`_                                                      `GitHub - device-manager`_
 /flows        Flow management                 mashup                                                                `GitHub - mashup <https://github.com/dojot/mashup>`_ 
 /auth         User authentication             `auth <https://dojot.github.io/auth/auth.html>`_                      `GitHub - auth <https://github.com/dojot/auth>`_ 
 /auth/revoke  User authentication             `auth`_                                                               `GitHub - auth <https://github.com/dojot/auth>`_ 
 /auth/user    User authentication             `auth`_                                                               `GitHub - auth <https://github.com/dojot/auth>`_ 
 /history      Device historical data          STH                                                                   `GitHub - STH <https://github.com/telefonicaid/fiware-sth-comet>`_ 
 /metric       Context broker                  Orion                                                                 `GitHub - Orion <https://github.com/dojot/fiware-orion>`_ 
 /gui          Graphical User Interface        GUI                                                                   `GitHub - GUI <https://github.com/dojot/gui>`_ 
 /sign         Public key signing              ejbca                                                                 `GitHub - ejbca-rest <https://github.com/dojot/ejbca-rest>`_ 
 /ca           Certification-Auth. functions   ejbca                                                                 `GitHub - ejbca-rest`_ 
============= =============================== ===================================================================== ================


The API gateway used in dojot reroutes some of these endpoints so that they become uniform: all of them are accessible through the same port (default is TCP port 8000) and have the same naming scheme. Each component, though, might have something different in its configuration and API documentation. The following table shows which endpoint exposed by the API gateway is mapped to which component endpoint.

======================== =======================
 Endpoint                 Original endpoint     
======================== =======================
 host:8000/device         host:5000/device      
 host:8000/template       host:5000/template    
 host:8000/flows          host:3000/            
 host:8000/auth           host:5000/            
 host:8000/auth/revoke    host:5000/auth/revoke 
 host:8000/auth/user      host:5000/user        
 host:8000/history        host:8666/            
 host:8000/metric         host:1026/            
 host:8000/gui            host/                 
 host:8000/sign           host:5583/            
 host:8000/ca             host:5583/            
======================== =======================
