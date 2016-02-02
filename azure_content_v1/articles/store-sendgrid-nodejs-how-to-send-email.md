<properties 
    pageTitle="How to use the SendGrid email service (Node.js) | Microsoft Azure" 
    description="Learn how send email with the SendGrid email service on Azure. Code samples written using the Node.js API." 
    services="" 
    documentationCenter="nodejs" 
    authors="erikre" 
    manager="wpickett" 
    editor=""/>

<tags 
    ms.service="multiple" 
    ms.workload="na" 
    ms.tgt_pltfrm="na" 
    ms.devlang="nodejs" 
    ms.topic="article" 
    ms.date="01/05/2016" 
    ms.author="erikre"/>

# How to Send Email Using SendGrid from Node.js
This guide demonstrates how to perform common programming tasks with the
SendGrid email service on Azure. The samples are written using
the Node.js API. The scenarios covered include **constructing email**,
**sending email**, **adding attachments**, **using filters**, and
**updating properties**. For more information on SendGrid and sending
email, see the [Next Steps](#next-steps.md) section.

## What is the SendGrid Email Service?
SendGrid is a [cloud-based email service](https://sendgrid.com/email-solutions) that provides reliable
[transactional email delivery](https://sendgrid.com/transactional-email), scalability, and real-time analytics along with flexible APIs
that make custom integration easy. Common SendGrid usage scenarios
include:

* Automatically sending receipts to customers
* Administering distribution lists for sending customers monthly
e-fliers and special offers
* Collecting real-time metrics for things like blocked e-mail, and
customer responsiveness
* Generating reports to help identify trends
* Forwarding customer inquiries
* Email notifications from your application

For more information, see [https://sendgrid.com](https://sendgrid.com).

## Create a SendGrid Account
Azure customers can unlock 25,000 free emails each month. These 25,000 free monthly emails will give you access to advanced reporting and analytics and [all APIs][] (Web, SMTP, Event, Parse and more). For information about additional services provided by SendGrid, see the [SendGrid Features][] page.

### To sign up for a SendGrid account

1. Log in to the [Azure Management Portal][].

2. In the lower pane of the management portal, click **New**.

	![command-bar-new][command-bar-new]

3. Click **Marketplace**.

	![sendgrid-store][sendgrid-store]

4. In the **Choose an Application and Service** dialog, select **SendGrid** and click the right arrow.

5. In the **Personalize Application and Service** dialog select the **SendGrid** plan you want to sign up for.

6. Enter a name to identify your **SendGrid** service in your Azure settings, or use the default value of **SendGridEmailDelivery.Simplified.SMTPWebAPI**. Names must be between 1 and 100 characters in length and contain only alphanumeric characters, dashes, dots, and underscores. The name must be unique in your list of subscribed Azure Store Items.

	![store-screen-2][store-screen-2]

7. Choose a value for the region; for example, West US.

8. Click the right arrow.

9. On the **Review Purchase** tab, review the plan and pricing information, and review the legal terms. If you agree to the terms, click the check mark. After you click the check mark, your SendGrid account will begin the [SendGrid provisioning process].

	![store-screen-3][store-screen-3]

10. After confirming your purchase you are redirected to the add-ons dashboard and you will see the message **Purchasing SendGrid**.

	![sendgrid-purchasing-message][sendgrid-purchasing-message]

	Your SendGrid account is provisioned immediately and you will see the message **Successfully purchased Add-On SendGrid**. Your account and credentials are now created. You are ready to send emails at this point. 

	To modify your subscription plan or see the SendGrid contact settings, click the name of your SendGrid service to open the SendGrid Marketplace dashboard. 

	![sendgrid-add-on-dashboard][sendgrid-add-on-dashboard]

	To send an email using SendGrid, you must supply your  account credentials (username and password).

### To find your SendGrid credentials ###

1. Click **Connection Info**.

	![sendgrid-connection-info-button][sendgrid-connection-info-button]

2. In the *Connection info* dialog, copy the **Password** and Username to use later in this tutorial.

	![sendgrid-connection-info][sendgrid-connection-info]

	To set your email deliverability settings, click the **Manage** button. This will redirect to your SendGrid Control Panel. 

	![sendgrid-control-panel][sendgrid-control-panel]

	For more information on getting started with SendGrid, see [SendGrid Getting Started][].

<!--images-->

[command-bar-new]: ./media/sendgrid-sign-up/sendgrid_BAR_NEW.PNG
[sendgrid-store]: ./media/sendgrid-sign-up/sendgrid_offerings_store.png
[store-screen-2]: ./media/sendgrid-sign-up/sendgrid_store_scrn2.png
[store-screen-3]: ./media/sendgrid-sign-up/sendgrid_store_scrn3.png
[sendgrid-purchasing-message]: ./media/sendgrid-sign-up/sendgrid_purchasing_message.png
[sendgrid-add-on-dashboard]: ./media/sendgrid-sign-up/sendgrid_add-on_dashboard.png
[sendgrid-connection-info]: ./media/sendgrid-sign-up/sendgrid_connection_info.png
[sendgrid-connection-info-button]: ./media/sendgrid-sign-up/sendgrid_connection_info_button.png
[sendgrid-control-panel]: ./media/sendgrid-sign-up/sendgrid_control_panel.png

<!--Links-->

[SendGrid Features]: http://sendgrid.com/features
[Azure Management Portal]: https://manage.windowsazure.com
[SendGrid Getting Started]: http://sendgrid.com/docs
[SendGrid Provisioning Process]: https://support.sendgrid.com/hc/articles/200181628-Why-is-my-account-being-provisioned-
[all APIs]: https://sendgrid.com/docs/API_Reference/index.html



## Reference the SendGrid Node.js Module
The SendGrid module for Node.js can be installed through the node
package manager (npm) by using the following command:

    npm install sendgrid

After installation, you can require the module in your application by
using the following code:

    var sendgrid = require('sendgrid')(sendgrid_username, sendgrid_password);

The SendGrid module exports the **SendGrid** and **Email** functions.
**SendGrid** is responsible for sending email through Web API, 
while **Email** encapsulates an email message.

## How to: Create an Email
Creating an email message using the SendGrid module involves first
creating an email message using the Email function, and then sending it
using the SendGrid function. The following is an example of creating a
new message using the Email function:

    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

You can also specify an HTML message for clients that support it by
setting the html property. For example:

    html: This is a sample <b>HTML<b> email message.

Setting both the text and html properties provides graceful fallback to
text content for clients that cannot support HTML messages.

For more information on all properties supported by the Email function,
see [sendgrid-nodejs](https://github.com/sendgrid/sendgrid-nodejs).

## How to: Send an Email
After creating an email message using the Email function, you can send
it using the Web API provided by SendGrid. 

### Web API
    sendgrid.send(email, function(err, json){
        if(err) { return console.error(err); }
        console.log(json);
    });

> [!NOTE]
> While the above examples show passing in an email object and
> callback function, you can also directly invoke the send
> function by directly specifying email properties. For example:  
> 
> `````
> sendgrid.send({
>     to: 'john@contoso.com',
>     from: 'anna@contoso.com',
>     subject: 'test mail',
>     text: 'This is a sample email message.'
> });
> `````
> 
> 
## How to: Add an Attachment
Attachments can be added to a message by specifying the file name(s) and
path(s) in the **files** property. The following example demonstrates
sending an attachment:

    sendgrid.send({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.',
        files: [
            {
                filename:     '',           // required only if file.content is used.
                contentType:  '',           // optional
                cid:          '',           // optional, used to specify cid for inline content
                path:         '',           //
                url:          '',           // == One of these three options is required
                content:      ('' | Buffer) //
            }
        ],
    });

> [!NOTE]
> When using the **files** property, the file must be accessible
> through [fs.readFile](http://nodejs.org/docs/v0.6.7/api/fs.html#fs.readFile). If the file you wish to attach is hosted in Azure Storage, such as in a Blob container, you must first copy the file to local storage or to an Azure drive before it can be sent as an attachment using the **files** property.
> 
> 
## How to: Use Filters to Enable Footers and Tracking
SendGrid provides additional email functionality through the use of
filters. These are settings that can be added to an email message to
enable specific functionality such as enabling click tracking, Google
analytics, subscription tracking, and so on. For a full list of filters,
see [Filter Settings](https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html).

Filters can be applied to a message by using the **filters** property.
Each filter is specified by a hash containing filter-specific settings.
The following examples demonstrate the footer and click tracking filters:

### Footer
    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

    email.setFilters({
        'footer': {
            'settings': {
                'enable': 1,
                'text/plain': 'This is a text footer.'
            }
        }
    });

    sendgrid.send(email);

### Click Tracking
    var email = new sendgrid.Email({
        to: 'john@contoso.com',
        from: 'anna@contoso.com',
        subject: 'test mail',
        text: 'This is a sample email message.'
    });

    email.setFilters({
        'clicktrack': {
            'settings': {
                'enable': 1
            }
        }
    });

    sendgrid.send(email);

## How to: Update Email Properties
Some email properties can be overwritten using **set*Property*** or
appended using **add*Property***. For example, you can add additional
recipients by using

    email.addTo('jeff@contoso.com');

or set a filter by using

    email.addFilter('footer', 'enable', 1);
    email.addFilter('footer', 'text/html', '<strong>boo</strong>');

For more information, see [sendgrid-nodejs](https://github.com/sendgrid/sendgrid-nodejs).

## How to: Use Additional SendGrid Services
SendGrid offers web-based APIs that you can use to leverage additional
SendGrid functionality from your Azure application. For full
details, see the [SendGrid API documentation](https://sendgrid.com/docs).

## Next Steps
Now that you've learned the basics of the SendGrid Email service, follow
these links to learn more.

* SendGrid Node.js module repository: [sendgrid-nodejs](https://github.com/sendgrid/sendgrid-nodejs)
* SendGrid API documentation:
[https://sendgrid.com/docs](https://sendgrid.com/docs)
* SendGrid special offer for Azure customers:
[http://sendgrid.com/azure.html](https://sendgrid.com/windowsazure.html)

  [special offer]: https://sendgrid.com/windowsazure.html
  [sendgrid-nodejs]: https://github.com/sendgrid/sendgrid-nodejs
  [Filter Settings]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html
  [SendGrid API documentation]: https://sendgrid.com/docs
  [cloud-based email service]: https://sendgrid.com/email-solutions
  [transactional email delivery]: https://sendgrid.com/transactional-email
