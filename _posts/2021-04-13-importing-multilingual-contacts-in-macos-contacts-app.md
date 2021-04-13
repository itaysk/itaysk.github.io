---
title: Importing multilingual contacts in MacOS Contacts app
date: 2021-04-13 18:00:00
categories: [Technical-Howto]
tags: [awk, ui]
---

I needed to move my contacts from Outlook.com to iCloud.

## Export the data

In Outlook.com, you can export your contacts, but only to CSV. Not the best format for inter-operability. But this is what we got.

## Attempt 1 - Import CSV

I have found that the MacOS "Contacts" app has a customizable import feature. You can load the csv, and map fields into contact cards. That sounds promising! but:

As you can see there's something wrong with encoding. I have some Hebrew text in my contacts, which got messed up in translation.

My initial suspicion was the origin file encoding, but the CSV is using UTF-8 which is supposed to be good:

![gibberish](/images/2021-04-13-importing-multilingual-contacts-in-macos-contacts-app_1.png)

```bash
❯ file -I contacts.csv
contacts.csv: application/csv; charset=utf-8
```

## CSV to VCF

VCF (vCard) is the common format for contact information management and exchange. Maybe the contacts app will like it better.

I started by searching for tools to do the conversion, and I did find some, but didn't trust them with my contacts data, so I looked for an open source script. I did find a few ones, but they either didn't work for me or was too complicated so I figured it would be easier to write my own 🤓.

I used AWK to scan the csv, and manually mapped the fields. Here's the result:

{% gist 4e1091bacb2b9d44d37a9b9274217576 csv2vcf.awk %}

## Attempt 2 - Import VCF

I took the converted contacts, in VCF format, and tried to load it - Again same issue with the gibberish. I verified that the VCF file is UTF-8 as well, which is what VCF should be, and so I figured it's a problem with the Contacts app.

```bash
❯ file -I contacts.vcf
contacts.vcf: text/plain; charset=utf-8
```

## Confusing UI

I took a quick look at the Contacts app's preferences, and saw this:

![grayed out](/images/2021-04-13-importing-multilingual-contacts-in-macos-contacts-app_2.png)

As you can see, encoding is only relevant in VCF 2.1, and I'm exporting VCF 3.0.

I decided to still try, so I momentarily switched to 2.1, changed it's encoding to UTF-8, and then back to 3.0. According to the UI I did nothing, but let's see.

![enable](/images/2021-04-13-importing-multilingual-contacts-in-macos-contacts-app_3.png)
![back](/images/2021-04-13-importing-multilingual-contacts-in-macos-contacts-app_4.png)

### Attempt 3 - Success

I loaded the same VCF file and viola! All the info was imported correctly both in English and Hebrew.
