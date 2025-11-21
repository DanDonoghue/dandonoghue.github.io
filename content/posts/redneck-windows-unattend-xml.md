+++
title = "Redneck Windows Unattend.xml"
date = "2023-05-12T23:12:00Z"
description = "How to make Windows use an unattend.xml file, the bodge way"
+++

So I wanted a base VHD boot image for my laptop so I can quickly create a new Windows installation without having to go through the tedium of running a bunch of WIM commands. The basic idea was to create one base image setup with drivers and updates, then Sysprep that image, and then I can create a new differencing VHD backed by that base image.

The only downside was that the Windows Out-of-box-"experience" (OOBE) slowed the whole process down by asking asinine questions about privacy and stuff. I know! I'll automate it.

## Unattend.xml

While playing around, I wanted to test out a few different ways of using unattend.xml but applying it when running the Sysprep command was taking too damn long. I wanted a quicker way... what if I ran Sysprep, took a snapshot, and then dumped `unattend.xml` into the VM where it's supposed to go? Hmm that could work.

Now, finding out where that XML went was a bit of a pain because Google would come up with millions of results about using it with `Sysprep.exe` or `setup.exe` or with MDT. Eventually I found a post that said that it gets saved to `\Windows\Panther\unattend.xml`. Finally, so I came up with this simple file that'll set the keyboard to the good old English UK layout, set the Administrator password to `Pa$$w0rd`, and auto-logon a couple of times to save having to login and watch for hours while Windows sets up the new user profile.

{{< highlight xml >}}
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State">
    <settings pass="windowsPE">
        <component name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <SetupUILanguage>
                <UILanguage>en-US</UILanguage>
            </SetupUILanguage>
            <InputLocale>0809:00000809</InputLocale>
            <SystemLocale>en-GB</SystemLocale>
            <UILanguage>en-US</UILanguage>
            <UserLocale>en-GB</UserLocale>
        </component>
        <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <UserData>
                <ProductKey>
                    <Key></Key>
                </ProductKey>
                <AcceptEula>true</AcceptEula>
            </UserData>
        </component>
    </settings>
    <settings pass="oobeSystem">
        <component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <InputLocale>0809:00000809</InputLocale>
            <SystemLocale>en-GB</SystemLocale>
            <UILanguage>en-US</UILanguage>
            <UserLocale>en-GB</UserLocale>
        </component>
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <UserAccounts>
                <AdministratorPassword>
                    <Value>UABhACQAJAB3ADAAcgBkAEEAZABtAGkAbgBpAHMAdAByAGEAdABvAHIAUABhAHMAcwB3AG8AcgBkAA==</Value>
                    <PlainText>false</PlainText>
                </AdministratorPassword>
            </UserAccounts>
            <AutoLogon>
                <Username>Administrator</Username>
                <Enabled>true</Enabled>
                <LogonCount>1</LogonCount>
                <Password>
                    <Value>UABhACQAJAB3ADAAcgBkAFAAYQBzAHMAdwBvAHIAZAA=</Value>
                    <PlainText>false</PlainText>
                </Password>
            </AutoLogon>
            <OOBE>
                <ProtectYourPC>3</ProtectYourPC>
                <HideEULAPage>true</HideEULAPage>
                <HideWirelessSetupInOOBE>true</HideWirelessSetupInOOBE>
            </OOBE>
        </component>
    </settings>
    <cpi:offlineImage cpi:source="wim:c:/users/dan/documents/install.wim#Windows 11 Pro" xmlns:cpi="urn:schemas-microsoft-com:cpi" />
</unattend>

{{< / highlight >}}

Now I can spin up a fresh copy of Windows in a couple of minutes, while only taking up ~12GB of my Surface Pro 5's tiny SSD no matter how many copies I make; and because the image is Sysprepped it's all good for fuckign about with Active Directory.