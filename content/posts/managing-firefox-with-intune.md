+++
title = "Managing Firefox with Intune"
date = "2023-06-26T19:00:00Z"
description = "Customising Mozilla Firefox with Microsoft Intune using Group Policy ADMX Templates"
+++

## Import ADMX Files

You'll need to download the latest policy_templates ZIP file from the release assets on [this page](https://github.com/mozilla/policy-templates/releases). Extract these, then you need to upload them to Intune.

You'll need to go to Devices > Windows > [Configuration Profiles](https://endpoint.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesWindowsMenu/~/configProfiles) and then choose **Import ADMX** from the top. 

From this page, choose **Import**. Intune will ask for an ADMX file and an ADML file. First, upload **windows\mozilla.admx** and **windows\en-US\mozilla.adml** from the extracted policies ZIP file. 

![Intune ADMX upload for mozilla.admx](/images/managing_firefox_with_intune/intune_admx_import.png)

Wait for this to finish uploading and importing, then do the same for **windows\firefox.admx** with **windows\en-US\firefox.adml**. Once completed, it should look like below. Note the time difference in the **created** field; you have to wait for the mozilla template to process before it'll let you upload the firefox template.

![Intune with Imported ADMX templates for mozilla.admx and firefox.admx](/images/managing_firefox_with_intune/intune_admx_post_import.png)

Once you're at this stage, you're ready to setup a configuration profile

## Create Configuration Profile

Go to Devices > Windows > Configuration profiles, once there click **Create profile**. In the pop-up, choose **Windows 10 and later** as the platform, and **Templates** for profile type. In the list of templates, you're looking to create a profile from **Imported Administrative templates**. 

On the Configuration settings panel, depending on how you're going to deploy this profile you can either set things in the **Computer Configuration** or **User Configuration** settings. You can go through to set the configuration to how you want it, similar to any other GPO.

One important one is the **Windows SSO** enablement, which lets Firefox automatically login to online services the same way Edge does without requiring the user to re-type their password. 

Here's my test configuration I have set up on my lab, which changes the homepage, disables private browsing, removes the pop-up for setting the default browser, removes the first run page, and also prevents the [Firefox Privacy Notice](https://www.mozilla.org/en-US/privacy/firefox/) from displaying at first run.

![Firefox Configuration Profile in Intune](/images/managing_firefox_with_intune/firefox_config_profile.png)

Removing the first-run privacy notice tab was the hardest part to understand. It's configured via the **Mozilla\Firefox\Preferences** option (not the one that says deprecated). This option takes settings as a JSON object similar to the one below.

```
{
    "datareporting.policy.dataSubmissionPolicyBypassNotification": {
    	"Value": true,
    	"Status": "default"
    }
}
```

The options for this are documented [here](https://github.com/mozilla/policy-templates/tree/master#preferences), albeit confusingly. The basic idea is that **preferences.json** is sent via policy and read from the registry, but why that isn't written like that is beyond me.

It's basically a list of JSON objects stored in a keyed array, so you can push multiple preferences from this one policy option. The one below will disable the privacy notice and remove the warning when closing with multiple tabs open.

```
{
    "datareporting.policy.dataSubmissionPolicyBypassNotification": {
    	"Value": true,
    	"Status": "default"
    },
    "browser.tabs.warnOnClose": {
        "Value": false,
        "Status": "default"
    }
}
```

Hopefully it's a little easier to understand than the Mozilla documentation was.

Once you've set everything you need to, just go through and assign it to whatever you want to assign it to. If all's gone well, you'll have Firefox running how you want it.