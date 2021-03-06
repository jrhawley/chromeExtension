# Pubpeer for Microsoft Edge

Based on PubPeer extension for Chrome and Firefox, with updates from [Microsoft Edge Extension Toolkit](https://www.microsoft.com/en-ca/store/p/microsoft-edge-extension-toolkit/9nblggh4txvb)

## Installation

To use this unsigned extension as is, please do the following:

1. Ensure that apps can be sideloaded in Edge.
    Go to Settings > Update & security > For developers, and select "Sideload apps" or "Developer mode".
    Then, go to Edge > about:flags, and check "Enable extension developer features".
1. Load extension into Edge: ... > Extensions > Load Extension, and browse to this repo clone


## Packaging this application for the Microsoft Store

To convert this code into an AppX package that can be submitted to the Microsoft Store, follow these steps from [this Microsoft guide](https://docs.microsoft.com/en-us/microsoft-edge/extensions/guides/packaging/creating-and-testing-extension-packages):

1. [Use `manifoldjs`] to create the packaged verion
    ```powershell
    cd ..

    # create initial package
    manifoldjs -l debug -p edgeextension -f edgeextension -m PubPeerEdge\manifest.json

    # create final package
    manifoldjs -l debug -p edgeextension package <EXTENSION NAME>\edgeextension\manifest\
    ```
1. Manually edit AppXManifest.xml
    In `PubPeer\edgeextension\manifest\appxmanifest.xml`, edit the corresponding entries as per [this article](https://docs.microsoft.com/en-us/microsoft-edge/extensions/guides/packaging/creating-and-testing-extension-packages#preparing-the-submission-folder)
1. Create AppX package
    ```powershell
    cd "C:\Program Files (x86)\Windows Kits\10\bin\x64"
    makeappx.exe pack /h SHA256 /d PubPeer\edgeextension\manifest /p PubPeer\edgeextension\edgeExtension.appx
    ```

If this extension will then be uploaded to the Microsoft Store, you don't need to go any further, since the store signs and certifies app packages from here.

If you want to test the extension locally, you need to follow the next section.

## Signing AppX package for installation

This is for testing your AppX package before submission to the Microsoft Store.
I'm following 

1. Create a self-signed certificate in PowerShell (admin). See [this article](https://docs.microsoft.com/en-us/windows/uwp/packaging/create-certificate-package-signing) from Microsoft.
    ```powershell
    New-SelfSignedCertificate -Type Custom -Subject <Package/Identity/Publisher from Microsoft Developer Dashboard> -KeyUsage DigitalSignature -FriendlyName <Package/Properties/PublisherDisplayName from Microsoft Developer Dashboard> -CertStoreLocation "Cert:\LocalMachine\My"
    ```
1. Export the certificate
    ```powershell
    $pwd = ConvertTo-SecureString -String <Your Password> -Force -AsPlainText 
    Export-PfxCertificate -cert "Cert:\LocalMachine\My\<Certificate Thumbprint>" -FilePath <FilePath>.pfx -Password $pwd
    ```
1. Sign AppX package using new certificate
    See [this article](https://docs.microsoft.com/en-us/windows/uwp/packaging/sign-app-package-using-signtool) from Microsoft.
    ```powershell
    cd C:\Program Files (x86)\Windows Kits\10\bin\x64
    signtool.exe sign /fd SHA256 /a /f <Path to Certificate>.pfx /p <Your Password> PubPeer\edgeextension\edgeExtension.appx
    ```
1. Trust AppX certificate on local machine
    1. Right-Click > Properties on the AppX package.
    1. Digital Signatures Tab, click on signature, click Details button
    1. View Certificate
    1. Install Certificate
    1. Local Machine, Next, Place all certificates in the following store: "Trusted Root Certificaiton Authorities"
    1. Repeat the above steps with the "Trusted People" store
1. Install AppX package by double-clicking on it

### After testing

To ensure security of your machine, after you finish testing the AppX package, remove the certificate from your computer's store.

```powershell
Certutil -delStore TrustedPeople <certID>
```

You can find `<certID>` by finding it in the list generated by:
```powershell
Certutil -store TrustedPeople
```