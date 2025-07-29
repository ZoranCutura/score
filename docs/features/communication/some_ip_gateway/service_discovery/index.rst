..
   # *******************************************************************************
   # Copyright (c) 2025 Contributors to the Eclipse Foundation
   #
   # See the NOTICE file(s) distributed with this work for additional
   # information regarding copyright ownership.
   #
   # This program and the accompanying materials are made available under the
   # terms of the Apache License Version 2.0 which is available at
   # https://www.apache.org/licenses/LICENSE-2.0
   #
   # SPDX-License-Identifier: Apache-2.0
   # *******************************************************************************

.. _some_ip_gateway_service_discovery:

SOME/IP-Gateway Service Discovery
#################################

Abstract
========

Draft plan:

- provided / required services (including their service elements like events, methods, and fields) must be configured
- (locally) provided services are discovered (locally) via lola and then offered on the network by sending a SOME/IP-SD message with an OfferService entry according to configuration
- (locally) required services may trigger the discovery on the network by sending of a SOME/IP-SD message with a FindService entry
- When a SOME/IP-SD message containing a FindService entry is received, the SOME/IP gateway checks this service has been already offered locally via lola

 - If this is the case, the SOME/IP gateway answers with a SOME/IP-SD message containing a corresponding OfferService entry
 - Otherwise, the SOME/IP gateway initiates a local service discovery via lola. - Once this is successful, he SOME/IP gateway answers with a SOME/IP-SD message containing a corresponding OfferService entry

  - this will not trigger creation of the service internally

TODO: show how services from the network are presented to the local system

Configuration
=============

The configuration tells the SOME/IP communication stack which services it should provide and which services it should require on the network.
The configuration contains SOME/IP and SOME/IP-SD settings as well as IP interface bindings.

Configuration is provided as ``json`` files and read at startup.

TODO: is ``json`` a problem? Examples also read `json` at startup.

Provided services
=================

Services available in the local ECU / VM are offered by the SOME/IP gateway once they become internally available and are present in the configuration.
This is done by sending an SOME/IP-SD messages containing an OfferService entry to the network.
As long as the service is locally available the service offering is periodically (according to the configuration) renewed by sending a SOME/IP-SD messages containing an OfferService entry to the network.
Once the service ceased to exist locally, the SOME/IP communication stack sends a SOME/IP-SD message containing a StopOfferService entry to the network.

For IPC service discovery the features of lola are used by the SOME/IP gateway.

.. uml::
   :alt: Gateway offers a service to the network

   @startuml
   participant "IPC_client" as IPC_client
   participant "Service\nvia IPC" as Service
   participant "SOME/IP Gateway" as Someipgateway
   actor "Network" as Network

   IPC_client -> Service ** : create()
   Someipgateway -> Service: service_discovery()
   Someipgateway -> Network: OfferService
   @enduml

.. uml::
   :alt: Gateway stops service offer

   @startuml
   participant "IPC_client" as IPC_client
   participant "Service\nvia IPC" as Service
   participant "SOME/IP Gateway" as Someipgateway
   actor "Network" as Network

   IPC_client -> Service !! : delete()
   Someipgateway -> Service: service_discovery()
   Someipgateway -> Network: StopOfferService
   @enduml

.. note:: ``mw::com::<Proxy>::StartFindService()`` function can tell us that a service is offered or stopped. But using it is awkward / complicated.

Required services
=================

For configured required services, the SOME/IP communication stack will send a SOME/IP-SD message containing a FindService entry to the network.

FindService message might not be needed, if OfferService messages for that services have been already received.

Upon initial reception of a SOME/IP-SD message containing an OfferService entry, the SOME/IP communication stack checks if the service has been configured as required service.
If so, the SOME/IP communication stack offers the service locally in the ECU / VM.
Here, initial reception is the first reception after a previous service loss or withdrawal, i.e.,

- after the reception of a SOME/IP-SD message containing an StopOfferService entry
- after a TTL expiry of the previous OfferService entry
- after the detection of a reboot of the (remote) service provider
- after a link-down/link-up event

.. uml::
   :alt: Gateway receives OfferService from the network

   @startuml
   actor "Network" as Network
   participant "SOME/IP Gateway" as Someipgateway
   participant "Service\nvia IPC" as Service
   participant "IPC_client" as IPC_client

   Network -> Someipgateway: OfferService
   Someipgateway -> Service ** : create()
   IPC_client -> Service: service_discovery()
   IPC_client -> Service: connect()
   @enduml

.. uml::
   :alt: Gateway receives StopOfferService from the network

   @startuml
   actor "Network" as Network
   participant "SOME/IP Gateway" as Someipgateway
   participant "Service" as Service
   participant "IPC_client" as IPC_client

   Network -> Someipgateway: StopOfferService
   Someipgateway -> Service !! : delete()
   IPC_client -> Service: service_discovery()
   IPC_client -> IPC_client: handle_disconnect()
   @enduml

FindService
================

Upon reception of a SOME/IP-SD message containing a FindService entry, the SOME/IP communication stack checks if the service is available locally and has been configured as provided service.
If both questions are answered positively, the SOME/IP communication stack responds by sending a SOME/IP-SD message containing an OfferService to the sender of the SOME/IP-SD message containing a FindService entry.

.. uml::
   :alt: Gateway receives FindService from the network

   @startuml
   actor "Network" as Network
   participant "SOME/IP Gateway" as Someipgateway
   participant "Service\nvia IPC" as Service

   Network -> Someipgateway: FindService
   Someipgateway -> Service: service_discovery()
   alt Service available
       Someipgateway -> Network: OfferService
   end
   @enduml

.. uml::
   :alt: Gateway sends FindService to the network

   @startuml
   actor "Network" as Network
   participant "SOME/IP Gateway" as Someipgateway

   loop required Service
      alt no OfferService received
         Someipgateway -> Network: FindService
      end
   end
   @enduml
