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

Configuration

Provided services
=================

Required services
=================

FindService
================
