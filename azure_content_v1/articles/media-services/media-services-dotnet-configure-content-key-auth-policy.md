<properties 
    pageTitle="Configure Content Key Authorization Policy using Media Services .NET SDK" 
    description="Learn how to configure an authorization policy for a content key using Media Services .NET SDK." 
    services="media-services" 
    documentationCenter="" 
    authors="juliako,Mingfeiy" 
    manager="dwrede" 
    editor=""/>

<tags 
    ms.service="media-services" 
    ms.workload="media" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="11/06/2015"
    ms.author="juliako"/>



# Dynamic Encryption: Configure Content Key Authorization Policy
> [AZURE.SELECTOR] 
- [Portal](../articles/media-services/media-services-portal-configure-content-key-auth-policy.md)
- [.NET](../articles/media-services/media-services-dotnet-configure-content-key-auth-policy.md)
- [REST API](../articles/media-services/media-services-rest-configure-content-key-auth-policy.md)
- [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)

## Overivew
Microsoft Azure Media Services enables you to deliver your content dynamically encrypted with Advanced Encryption Standard (AES) (using 128-bit encryption keys) and/or PlayReady DRM. Media Services also provides a service for delivering keys and PlayReady licenses to authorized clients. 

Currently, you can encrypt the following streaming formats: HLS, MPEG DASH, and Smooth Streaming. You cannot encrypt HDS streaming format, or progressive downloads.

If you want for Media Services to encrypt an asset, you need to associate an encryption key (**CommonEncryption** or **EnvelopeEncryption**) with the asset (as described [here](media-services-dotnet-create-contentkey.md)) and also configure authorization policies for the key (as described in this article). 

When a stream is requested by a player, Media Services uses the specified key to dynamically encrypt your content using AES or PlayReady encryption. To decrypt the stream, the player will request the key from the key delivery service. To decide whether or not the user is authorized to get the key, the service evaluates the authorization policies that you specified for the key.

Media Services supports multiple ways of authenticating users who make key requests. The content key authorization policy could have one or more authorization restrictions: **open** or **token** restriction. The token restricted policy must be accompanied by a token issued by a Secure Token Service (STS). Media Services supports tokens in the **Simple Web Tokens** ([SWT](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2)) format and **JSON Web Token **(JWT) format.  

Media Services does not provide Secure Token Services. You can create a custom STS or leverage Microsoft Azure ACS to issue tokens. The STS must be configured to create a token signed with the specified key and issue claims that you specified in the token restriction configuration (as described in this article). The Media Services key delivery service will return the encryption key to the client if the token is valid and the claims in the token match those configured for the content key.

For more information, see 

[JWT token authenitcation](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)

[Integrate Azure Media Services OWIN MVC based app with Azure Active Directory and restrict content key delivery based on JWT claims](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/).

[Use Azure ACS to issue tokens](http://mingfeiy.com/acs-with-key-services).

### Some considerations apply:
* To be able to use dynamic packaging and dynamic encryption, you must make sure to have at least one streaming reserved unit. For more information, see [How to Scale a Media Service](media-services-manage-origins.md#scale_streaming_endpoints). 
* Your asset must contain a set of adaptive bitrate MP4s or  adaptive bitrate Smooth Streaming files. For more information, see [Encode an asset](media-services-encode-asset.md).  
* Upload and encode your assets using **AssetCreationOptions.StorageEncrypted** option.
* If you plan to have multiple content keys that require the same policy configuration, it is strongly recommended to create a single authorization policy and reuse it with multiple content keys.
* The Key Delivery service caches ContentKeyAuthorizationPolicy and its related objects (policy options and restrictions) for 15 minutes.  If you create a ContentKeyAuthorizationPolicy and specify to use a “Token” restriction, then test it, and then update the policy to “Open” restriction, it will take roughly 15 minutes before the policy switches to the “Open” version of the policy.
* If you add or update your asset’s delivery policy, you must delete an existing locator (if any) and create a new locator.

## AES-128 Dynamic Encryption
### Open Restriction
Open restriction means the system will deliver the key to anyone who makes a key request. This restriction might be useful for testing purposes.

The following example creates an open authorization policy and adds it to the content key.

    static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
    {
        // Create ContentKeyAuthorizationPolicy with Open restrictions 
        // and create authorization policy             
        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("Open Authorization Policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions =
            new List<ContentKeyAuthorizationPolicyRestriction>();

        ContentKeyAuthorizationPolicyRestriction restriction =
            new ContentKeyAuthorizationPolicyRestriction
            {
                Name = "HLS Open Authorization Policy",
                KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                Requirements = null // no requirements needed for HLS
            };

        restrictions.Add(restriction);

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create(
            "policy", 
            ContentKeyDeliveryType.BaselineHttp, 
            restrictions, 
            "");

        policy.Options.Add(policyOption);

        // Add ContentKeyAutorizationPolicy to ContentKey
        contentKey.AuthorizationPolicyId = policy.Id;
        IContentKey updatedKey = contentKey.UpdateAsync().Result;
        Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);
    }


### Token Restriction
This section describes how to create a content key authorization policy and associate it with the content key. The authorization policy describes what authorization requirements must be met to determine if the user is authorized to receive the key (for example, does the “verification key” list contain the key that the token was signed with).

To configure the token restriction option, you need to use an XML to describe the token’s authorization requirements. The token restriction configuration XML must conform to the following XML schema.

#### <a id="schema"></a>Token restriction schema
    <?xml version="1.0" encoding="utf-8"?>
    <xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" xmlns:xs="http://www.w3.org/2001/XMLSchema">
      <xs:complexType name="TokenClaim">
        <xs:sequence>
          <xs:element name="ClaimType" nillable="true" type="xs:string" />
          <xs:element minOccurs="0" name="ClaimValue" nillable="true" type="xs:string" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="TokenClaim" nillable="true" type="tns:TokenClaim" />
      <xs:complexType name="TokenRestrictionTemplate">
        <xs:sequence>
          <xs:element minOccurs="0" name="AlternateVerificationKeys" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
          <xs:element name="Audience" nillable="true" type="xs:anyURI" />
          <xs:element name="Issuer" nillable="true" type="xs:anyURI" />
          <xs:element name="PrimaryVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
          <xs:element minOccurs="0" name="RequiredClaims" nillable="true" type="tns:ArrayOfTokenClaim" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="TokenRestrictionTemplate" nillable="true" type="tns:TokenRestrictionTemplate" />
      <xs:complexType name="ArrayOfTokenVerificationKey">
        <xs:sequence>
          <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="ArrayOfTokenVerificationKey" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
      <xs:complexType name="TokenVerificationKey">
        <xs:sequence />
      </xs:complexType>
      <xs:element name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
      <xs:complexType name="ArrayOfTokenClaim">
        <xs:sequence>
          <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenClaim" nillable="true" type="tns:TokenClaim" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="ArrayOfTokenClaim" nillable="true" type="tns:ArrayOfTokenClaim" />
      <xs:complexType name="SymmetricVerificationKey">
        <xs:complexContent mixed="false">
          <xs:extension base="tns:TokenVerificationKey">
            <xs:sequence>
              <xs:element name="KeyValue" nillable="true" type="xs:base64Binary" />
            </xs:sequence>
          </xs:extension>
        </xs:complexContent>
      </xs:complexType>
      <xs:element name="SymmetricVerificationKey" nillable="true" type="tns:SymmetricVerificationKey" />
    </xs:schema>

When configuring the **token** restricted policy, you must specify the primary** verification key**, **issuer** and **audience** parameters. The **primary verification key **contains the key that the token was signed with, **issuer** is the secure token service that issues the token. The **audience** (sometimes called **scope**) describes the intent of the token or the resource the token authorizes access to. The Media Services key delivery service validates that these values in the token match the values in the template. 

When using **Media Services SDK for .NET**, you can use the **TokenRestrictionTemplate** class to generate the restriction token.
The following example creates an authorization policy with a token restriction. In this example, the client would have to present a token that contains: signing key (VerificationKey), a token issuer, and required claims.

    public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
    {
        string tokenTemplateString = GenerateTokenRequirements();

        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("HLS token restricted authorization policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions =
                new List<ContentKeyAuthorizationPolicyRestriction>();

        ContentKeyAuthorizationPolicyRestriction restriction =
                new ContentKeyAuthorizationPolicyRestriction
                {
                    Name = "Token Authorization Policy",
                    KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                    Requirements = tokenTemplateString
                };

        restrictions.Add(restriction);

        //You could have multiple options 
        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create(
                "Token option for HLS",
                ContentKeyDeliveryType.BaselineHttp,
                restrictions,
                null  // no key delivery data is needed for HLS
                );

        policy.Options.Add(policyOption);

        // Add ContentKeyAutorizationPolicy to ContentKey
        contentKey.AuthorizationPolicyId = policy.Id;
        IContentKey updatedKey = contentKey.UpdateAsync().Result;
        Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);

        return tokenTemplateString;
    }

    static private string GenerateTokenRequirements()
    {
        TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

        template.PrimaryVerificationKey = new SymmetricVerificationKey();
        template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();

        template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

        return TokenRestrictionTemplateSerializer.Serialize(template);
    }

#### <a id="test"></a>Test token
To get a test token based on the token restriction that was used for the key authorization policy, do the following.

    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate =
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the the data in the given TokenRestrictionTemplate.
    // Note, you need to pass the key id Guid because we specified 
    // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
    Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);

    //The GenerateTestToken method returns the token without the word “Bearer” in front
    //so you have to add it in front of the token string. 
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
    Console.WriteLine();


## PlayReady Dynamic Encryption
Media Services enables you to configure the rights and restrictions that you want for the PlayReady DRM runtime to enforce when a user is trying to play back protected content. 

When protecting your content with PlayReady, one of the things you need to specify in your authorization policy is an XML string that defines the [PlayReady license template](media-services-playready-license-template-overview.md). In Media Services SDK for .NET, the **PlayReadyLicenseResponseTemplate** and **PlayReadyLicenseTemplate** classes will help you define the PlayReady License Template. 

### Open Restriction
Open restriction means the system will deliver the key to anyone who makes a key request. This restriction might be useful for testing purposes.

The following example creates an open authorization policy and adds it to the content key.

    static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
    {

        // Create ContentKeyAuthorizationPolicy with Open restrictions 
        // and create authorization policy          

        List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
        {
            new ContentKeyAuthorizationPolicyRestriction 
            { 
                Name = "Open", 
                KeyRestrictionType = (int)ContentKeyRestrictionType.Open, 
                Requirements = null
            }
        };

        // Configure PlayReady license template.
        string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
                ContentKeyDeliveryType.PlayReadyLicense,
                    restrictions, newLicenseTemplate);

        IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                    ContentKeyAuthorizationPolicies.
                    CreateAsync("Deliver Common Content Key with no restrictions").
                    Result;


        contentKeyAuthorizationPolicy.Options.Add(policyOption);

        // Associate the content key authorization policy with the content key.
        contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
        contentKey = contentKey.UpdateAsync().Result;
    }

### Token Restriction
To configure the token restriction option, you need to use an XML to describe the token’s authorization requirements. The token restriction configuration XML must conform to the XML schema shown in [this](#schema.md) section.

    public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
    {
        string tokenTemplateString = GenerateTokenRequirements();

        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("HLS token restricted authorization policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
        {
            new ContentKeyAuthorizationPolicyRestriction 
            { 
                Name = "Token Authorization Policy", 
                KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                Requirements = tokenTemplateString, 
            }
        };

        // Configure PlayReady license template.
        string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                ContentKeyDeliveryType.PlayReadyLicense,
                    restrictions, newLicenseTemplate);

        IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                    ContentKeyAuthorizationPolicies.
                    CreateAsync("Deliver Common Content Key with no restrictions").
                    Result;

        policy.Options.Add(policyOption);

        // Add ContentKeyAutorizationPolicy to ContentKey
        contentKeyAuthorizationPolicy.Options.Add(policyOption);

        // Associate the content key authorization policy with the content key
        contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
        contentKey = contentKey.UpdateAsync().Result;

        return tokenTemplateString;
    }

    static private string GenerateTokenRequirements()
    {

        TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

        template.PrimaryVerificationKey = new SymmetricVerificationKey();
        template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();


        template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

        return TokenRestrictionTemplateSerializer.Serialize(template);
    } 

    static private string ConfigurePlayReadyLicenseTemplate()
    {
        // The following code configures PlayReady License Template using .NET classes
        // and returns the XML string.

        //The PlayReadyLicenseResponseTemplate class represents the template for the response sent back to the end user. 
        //It contains a field for a custom data string between the license server and the application 
        //(may be useful for custom app logic) as well as a list of one or more license templates.
        PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();

        // The PlayReadyLicenseTemplate class represents a license template for creating PlayReady licenses
        // to be returned to the end users. 
        //It contains the data on the content key in the license and any rights or restrictions to be 
        //enforced by the PlayReady DRM runtime when using the content key.
        PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
        //Configure whether the license is persistent (saved in persistent storage on the client) 
        //or non-persistent (only held in memory while the player is using the license).  
        licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;

        // AllowTestDevices controls whether test devices can use the license or not.  
        // If true, the MinimumSecurityLevel property of the license
        // is set to 150.  If false (the default), the MinimumSecurityLevel property of the license is set to 2000.
        licenseTemplate.AllowTestDevices = true;


        // You can also configure the Play Right in the PlayReady license by using the PlayReadyPlayRight class. 
        // It grants the user the ability to playback the content subject to the zero or more restrictions 
        // configured in the license and on the PlayRight itself (for playback specific policy). 
        // Much of the policy on the PlayRight has to do with output restrictions 
        // which control the types of outputs that the content can be played over and 
        // any restrictions that must be put in place when using a given output.
        // For example, if the DigitalVideoOnlyContentRestriction is enabled, 
        //then the DRM runtime will only allow the video to be displayed over digital outputs 
        //(analog video outputs won’t be allowed to pass the content).

        //IMPORTANT: These types of restrictions can be very powerful but can also affect the consumer experience. 
        // If the output protections are configured too restrictive, 
        // the content might be unplayable on some clients. For more information, see the PlayReady Compliance Rules document.

        // For example:
        //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);

        responseTemplate.LicenseTemplates.Add(licenseTemplate);

        return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
    }


To get a test token based on the token restriction that was used for the key authorization policy see [this](#test.md) section. 

## <a id="types"></a>Types used when defining ContentKeyAuthorizationPolicy
### <a id="ContentKeyRestrictionType"></a>ContentKeyRestrictionType
    public enum ContentKeyRestrictionType
    {
        Open = 0,
        TokenRestricted = 1,
        IPRestricted = 2,
    }

### <a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType
    public enum ContentKeyDeliveryType
    {
      None = 0,
      PlayReadyLicense = 1,
      BaselineHttp = 2,
      Widevine = 3
    }

### <a id="TokenType"></a>TokenType
    public enum TokenType
    {
        Undefined = 0,
        SWT = 1,
        JWT = 2,
    }



## Media Services learning paths
You can view AMS learning paths here:

- [AMS Live Streaming Workflow](https://azure.microsoft.com/documentation/learning-paths/media-services-streaming-live/)
- [AMS on Demand Streaming Workflow](https://azure.microsoft.com/documentation/learning-paths/media-services-streaming-on-demand/)

## Provide feedback
Use the [User Voice](http://go.microsoft.com/fwlink/?linkid=698785&clcid=0x409) forum to provide feedback and make suggestions on how to improve Azure Media Services. You can also go directly to one of the following categories: 

- [Azure Media Player](https://feedback.azure.com/forums/169396-media-services/category/109320-azure-media-player/)
- [Client SDK Libraries](https://feedback.azure.com/forums/169396-media-services/category/144435-client-sdks/)
- [Encoding and Processing](https://feedback.azure.com/forums/169396-media-services/category/144411-encoding-and-processing/)
- [Live Streaming](https://feedback.azure.com/forums/169396-media-services/category/144414-live-streaming/)
- [Azure Portal](https://feedback.azure.com/forums/169396-media-services/category/144432-portal/)
- [REST API and Platform](https://feedback.azure.com/forums/169396-media-services/category/144423-rest-api-and-platform/)
- [VoD Streaming](https://feedback.azure.com/forums/169396-media-services/category/144429-vod-streaming/)

## Next Steps
Now that you have configured content key's authorization policy, go to the [How to configure asset delivery policy](media-services-dotnet-configure-asset-delivery-policy.md) topic.
