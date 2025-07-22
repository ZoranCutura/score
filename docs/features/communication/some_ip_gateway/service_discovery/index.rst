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

- provided / required services must be configured
- provided services are discovered via lola and then Offer message is sent according to configuration
- required services may trigger sending a FindService message
- When a FindService message is received, the SOME/IP checks if it can find the service locally via lola and when found responds
  - this will not trigger creation of the service internally

Configuration
=============

The configuration tells the SOME/IP communication stack which services it should provide and which services it should require on the network.
The configuration contains SOME/IP-SD settings and IP interface bindings.

Configuration is provided as ``json`` files and read at startup.

TODO: is ``json`` a problem?

Provided services
=================

Services available in the local ECU / VM are offered by the SOME/IP gateway once they become internally available and are present in the configuration.
This is done by sending an OfferService message to the SOME/IP network.

Once the service ceased to exist the SOME/IP communication stack sends a StopOfferService message to the SOME/IP network.

For IPC service discovery the features of lola are used by the SOME/IP gateway.

TODO: Does lola inform us that a service is stopped?

Required services
=================

For configured required services, the SOME/IP communication stack will send a FindService message to the SOME/IP network.

FindService message might not be needed, if OfferService messages for that services have been already received.

Upon reception of a OfferService message, the SOME/IP communication stack creates a connection and provides the service in the VM.

FindService
================

Upon reception of a FindService message, the SOME/IP communication stack checks if the service is available locally and has been configured as provided service.
If both questions are answered positively, the SOME/IP communication stack sends a ServiceFound message to the sender of the FindService message.
