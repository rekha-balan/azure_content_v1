<properties
   pageTitle="Non-technical prerequisites for creating an offer for the Azure Marketplace | Microsoft Azure"
   description="Understand the requirements for creating and deploying an offer to the Azure Marketplace for others to purchase."
   services="marketplace-publishing"
   documentationCenter=""
   authors="HannibalSII"
   manager=""
   editor=""/>

<tags
  ms.service="marketplace"
  ms.devlang="na"
  ms.topic="article"
  ms.tgt_pltfrm="Azure"
  ms.workload="na"
  ms.date="01/07/2016"
  ms.author="hascipio; v-divte"/>

# General prerequisites for creating an offer for the Azure Marketplace
Understand the general, business-process-centric prerequisites that are needed to move forward with the offer creation process.

## Ensure that you are registered as a seller with Microsoft
For detailed instructions on registering a seller account with Microsoft, go to [Account creation and registration](marketplace-publishing-accounts-creation-registration.md).

* If you are **already registered**, find out who in your company owns it or which credentials were used to register.
* If you are **not the owner of the publishing account**, you can have the account owner add your Microsoft account as a co-admin to the [publishing portal](https://publish.windowsazure.com). On the **Publishers** tab, use the **Administrators** link.
* Ensure that stakeholders in the Azure publishing process receive the email that goes to this address. It must be monitored and responded to in order to complete the publishing process.
* Avoid having the account associated with a single person. If that person leaves your company, they could impact your ability to access information about and publish your SKUs.

> [!IMPORTANT]
> You do not have to complete company tax and bank information if you are planning to publish only free offers (or bring your own license).
> 
> The company registration must be completed to get started. However, while your company works on the tax and bank information in the Microsoft Developer account, the developers can start working on creating the virtual machine image in the [publishing portal](https://publish.windowsazure.com), getting it certified, and testing it in the Azure staging environment. You will need the complete seller account approval only for the final step of publishing your offer to the Azure Marketplace.
> 
> If you have issues with completing the seller registration, log a support ticket as below:
> 
> 1. Contact [Support](https://support.microsoft.com/getsupport?wf=0tenant=ClassicCommercialoaspworkflow=start_1.0.0.0supportregion=en-uspesid=15635ccsid=635847950577064286).
> 2. Choose **Developer Center**.
> 3. Choose **Publisher Profile**.
> 4. Choose the contact method.
> 
> 
## Acquire an Azure "pay-as-you-go" subscription
This is the subscription that you will use to create your VM images and hand over the images to the [Azure Marketplace](https://azure.microsoft.com/marketplace/). If you do not have an existing subscription, then please sign up at https://account.windowsazure.com/signup?offer=ms-azr-0003p.

## "Sell-from" countries
> [!WARNING]
> In order to sell your services on the Azure Marketplace, you must make sure that your registered entity is from one of the approved “sell-from” countries. This restriction is for payout and taxation reasons. We are actively looking to expand this list of countries in the near future, so stay tuned. For the complete list, see section 1b of the [Azure Marketplace participation policies](http://go.microsoft.com/fwlink/?LinkID=526833).
> 
> 
## Next steps
Next are the technical prerequisites for each offer type. Click the link to the article for the respective offer type that you would like to create for the Azure Marketplace.

| Virtual machine image | Developer service | Data service | Solution template |
| --- | --- | --- | --- |
| [VM technical prerequisites](marketplace-publishing-vm-image-creation-prerequisites.md) |Developer service technical prerequisites |[Data service technical prerequisites](marketplace-publishing-data-service-creation-prerequisites.md) |[Solution template technical prerequisites](marketplace-publishing-solution-template-creation-prerequisites.md) |

## See also
* [Getting started: How to publish an offer to the Azure Marketplace](marketplace-publishing-getting-started.md)
