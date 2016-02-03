<properties
   pageTitle="Azure IoT protocol gateway | Microsoft Azure"
   description="Describes how to use Azure IoT protocol gateway to extend the capabilities and protocol support of Azure IoT Hub."
   services="iot-hub"
   documentationCenter=""
   authors="kdotchkoff"
   manager="timlt"
   editor=""/>

<tags
   ms.service="iot-hub"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/29/2015"
   ms.author="kdotchko"/>

# Supporting additional protocols for IoT Hub
Azure IoT Hub natively supports communication over the HTTPS and AMQP protocols. In some cases, devices or field gateways might not be able to use one of these standard protocols and will require protocol adaptation. In such cases, you can use a custom gateway. A custom gateway can enable protocol adaptation for IoT Hub endpoints by bridging the traffic to and from IoT Hub. An Azure Internet of Things (IoT) protocol gateway enables protocol adaptation for IoT Hub. It also implements an MQTT protocol adapter to enable communication between an IoT device and IoT Hub over the MQTT protocol.

## Azure IoT protocol gateway
The Azure IoT protocol gateway is a framework for protocol adaptation that is designed for high-scale, bidirectional device communication with IoT Hub. The protocol gateway is a pass-through component that accepts device connections over a specific protocol. It bridges the traffic to IoT Hub over AMQP 1.0. The IoT protocol gateway is available as an open-source software project to provide flexibility for adding support for a variety of protocols and protocol versions.

You can deploy the protocol gateway in Azure in a highly scalable way by using Azure Cloud Services worker roles. In addition, the protocol gateway can be deployed in on-premises environments, such as field gateways.

The Azure IoT protocol gateway includes an MQTT adapter to facilitate communication with devices over the MQTT v3.1.1 protocol. The protocol gateway and MQTT implementation are provided as an open-source software project for flexibility. This allows you to customize the implementation as needed.

The MQTT adapter also demonstrates the programming model for building protocol adapters for other protocols. In addition, the IoT protocol gateway programming model allows you to plug in custom components for specialized processing--such as custom authentication, message transformations, compression/decompression, or encryption/decryption of traffic between the devices and IoT Hub.

## Next steps
To learn more about the Azure IoT protocol gateway and how to use and deploy it as part of your IoT solution, see:

* [Azure IoT protocol gateway repository on GitHub](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)
* [Azure IoT protocol gateway developer guide](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/docs/DeveloperGuide.md)
