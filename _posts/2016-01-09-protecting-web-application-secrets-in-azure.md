---
layout: post
title: "Protecting web application secrets in Azure"
date: 2016-01-09 16:33:00 +0300
---

Most web applications have some configuration settings that are secrets - passwords, access keys etc. With traditional hosting based on (virtual) machines or IaaS these might not be such an issue or [sections of the configuration file can be encrypted][webConfigEncryption]. However, when hosting web applications on Azure, there are some special considerations and limitations related to managing application settings. This post focuses on hosting a web application on Azure App Service. 

In App Service Web Applications, the web.config application settings can be set via the management portal and those settings take precedence over the values defined in the config files. This allows not defining the plaintext settings in source control but they will still be visible to anyone who has access to the web application in Azure portal, which may be a problem for some. Overriding values manually also complicates the *configuration management* as the master values have to then be stored somewhere else, away from the code.

## Certificate-based encryption in Azure
This came as a bit of a surprise to me but in App Service Web Applications it actually possible to access the certificates that have been uploaded via the management portal from code. The trick is to define an application setting with key `WEBSITE_LOAD_CERTIFICATES` and set its value to either the thumbprint of the certificate to be loaded or `*` for all certificates. The certificates will then be loaded into the Personal store of the application pool user and can be accessed using standard .NET certificate APIs.

## Azure Key Vault
[Azure Key Vault][keyVault] is a rather new Azure offering to help safeguard keys and other secrets in cloud applications and services. It supports several use cases, including:
 - Create or impory keys (they call certificates keys)
 - Encrypt and decrypt data using keys
 - Store secrets

We could use Key Vault to store the secrets but that would basically have same limitations as setting them via the management portal - the configuration management is still an issue. So instead I prefer to just use Key Vault to encrypt & decrypt the secrets and put the encrypted values to source control.

The only caveat of the approach is that the Key Vault access is authenticated with Azure Active Directory username and password. See the chicken-and-egg problem there? Yep, now we need to store that password somewhere.

## Hybrid approach
Combining both encryption mechanisms provides us with the optimal solution, in my opinion. Key Vault is used to encrypt & decrypt application secrets and the Key Vault password is encrypted with a certificate. This way we can benefit from encryption, access control and auditing features of the Key Vault but still follow good configuration management practices by keeping application settings in source control.

If you got this far, you might have also realized that there's still one problem with this approch - we still need to manage the certificate that is used to encrypt the Key Vault password. For that, I don't have a single solution. Just keep it Somewhere Safe ™ after uploading it to Azure. Alternatively you could even throw it away and acquire a new certificate when the need to re-install the web application arises. The Key Vault "password" is actually just an Azure AD application key can as such be easily recreated. Yet another option is to authenticate Key Vault (Azure AD) access with a certificate, as shown in [this sample][aadCertAuthSample], removing the need for a password.

[keyVault]: https://azure.microsoft.com/en-us/services/key-vault/
[aadCertAuthSample]: https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential
[webConfigEncryption]: https://msdn.microsoft.com/fi-fi/library/zhhddkxy.aspx