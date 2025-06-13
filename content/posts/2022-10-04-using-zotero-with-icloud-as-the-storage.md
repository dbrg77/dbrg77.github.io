---
title: "Using Zotero With iCloud As The Storage"
date: 2022-10-04T15:07:53+08:00
tags: ['reference manager', 'zotero', 'icloud', 'paperpile', 'papers', 'writing']
type: post
---  

I used to use [__Paperpile__](https://paperpile.com) and [__Google Doc__](https://docs.google.com) for writing scientific papers and managing the literature PDFs. The process was very smooth. I did not really have the habit of saving PDFs locally, because I did not need to. In 2019, I relocated to my current place, and accessing Google Doc has been a pain ever since. Even if I managed to use Google Doc, the speed was pathetic and the connection was intermittent. The internet situation at my current university is pretty bad. To show you an example:

{{< figure src="/images/2022-10-04/dl_speed.png" width="600px" >}}

Sometimes, not always, it takes minutes to open a ~10 MB PDFs in my browser. For some journals it takes >10 minutes. I'm not exaggerating, and it happens more often than I can tolerate. Occasionally, I cannot even open the journal website. It would be nice to have an offline version of Paperpile. [__After 8 years of the initial feature request__](https://forum.paperpile.com/t/add-support-for-offline-mode/26), it may happen in the near future, but not now.

Then I tried [__Papers__](https://www.papersapp.com), because some of my friends recommended it. It had a nice look, and you could also save PDFs locally. However, without the access to Google Doc, I had to use Microsoft Word to write and use the Papers Word add-in [__SmartCite__](https://www.papersapp.com/smartcite-for-word/). In order to insert citations, I needed to sign in to their server with my account, and it was the same internet problem all over again. The Papers desktop app does support local PDFs, but it is difficult to manage with multiple computers. In addition, there were also many other minor problems that I found annoying. For example, this [__issue with doubled last names__](https://support.papersapp.com/support/discussions/topics/30000001673) is still not resolved at this time of writing (4th October 2022).

What I really need is a reference manager that has an offline mode, meaning that I can write and insert citations without internet connections, and at the same time offers a simple and straightforward way of synchronising reference data and PDFs across different computers. It turns out that [__Zotero__](https://www.zotero.org) is the perfect choice here. I did come across Zotero a long long time ago, and the user experience was not great back then. The current version (_Zotero 6_) seems to be much better than I remembered.

Basically, inserting citations offline is the default in Zotero. I just need to figure out how to synchronise the reference metadata (_i.e._ data syncing) and PDFs (_i.e._ file syncing) across different computers. The metadata is pretty small, so I'm okay with using the Zotero online storage for that even the connection is slow. The PDFs take a lot of space, and only 300 MB are available for free users. I need to find a cloud storage service that is fast and stable in my area. It turns out that iCloud is pretty good. I'm already a daily user, so it would be great if I could use iCloud as the PDFs storage. Some just put the entire Zotero data folder on the cloud, but it was [__strongly discouraged__](https://www.zotero.org/support/kb/data_directory_in_cloud_storage_folder). Therefore, the idea is to keep the metadata synced via the Zotero server, but keep PDFs on iCould. It turns out to be relative easy if we symlink the `storage` directory under the Zotero data folder. Linked files also work, but I found that concept a bit confusing.

First, we need to perform some initial setup in one computer, say _Computer 0_. Install Zotero, and sign in with your Zotero account in the `Sync` tab under Zotero `Preferences`. Uncheck the `File Syncing` options like this:

![](/images/2022-10-04/zotero_sync.png)

Then, create a folder called `zotero_storage` under the iCloud directory. Now go to the default Zotero data folder. If we did not change anything during installation, the Zotero data folder should be in your home directory: `/Users/<username>/Zotero`. There may or may not be a `storage` directory inside that folder. All our local PDFs are supposed to be there. If we just did a fresh install, the `storage` directory should not be there. We could simply created a symlink called `storage`, pointing to the `zotero_storage` folder we just created under iCloud. To this end, open the terminal and do this:

```bash
# this is done in Computer 0
cd /Users/<username>/Zotero
ln -s /Users/<username>/Library/Mobile\ Documents/com~apple~CloudDocs/zotero_storage storage
```

After that, our Zotero data folder should look like this:

![](/images/2022-10-04/zotero_data_folder.png)

Note the small black arrow at the bottom left corner of the `storage` directory, meaning it is a link.

If we already have Zotero in this computer, the `storage` directory is probably already in your Zotero data folder. Move all the content inside it to the `zotero_storage` folder under iCloud. Then delete the empty `storage` folder, and create a symlink called `storage`, pointing to the `zotero_storage` folder under iCloud. This can be achieved by opening the terminal and do:

```bash
# this is done in Computer 0
cd /Users/<username>/Zotero
rmdir storage
ln -s /Users/<username>/Library/Mobile\ Documents/com~apple~CloudDocs/zotero_storage storage
```

Now, you could add new PDFs to the reference as stored files, and Zotero will by default put those PDFs under `/Users/<username>/Zotero/storage`, which points to `/Users/<username>/Library/Mobile\ Documents/com~apple~CloudDocs/zotero_storage`. This effectively puts all files under our iCloud storage.

All the above steps should be done in _Computer 0_. 

After that, we could set up Zotero in other computers we need to use. In the new computers where you have your Apple ID signed in and iCloud working, do a fresh install of Zotero. Then, __with the Zotero program closed__, create a symlink in the same way as previous described:

```bash
# this is done in the new computers
cd /Users/<username>/Zotero
ln -s /Users/<username>/Library/Mobile\ Documents/com~apple~CloudDocs/zotero_storage storage
```

After that, open Zotero, sign in with your account, and turn off `File Syncing`, just like what we did in _Computer 0_. Then, everything should be in sync. The reference metadata is synced via the Zotero server, and the PDFs are located on iCloud, which could be accessed via Zotero in all computers.

I have this set up with Mac computers and iCloud, but the same principle should hold, in theory, for Windows/Linux and other cloud storage service providers.

***UPDATE 1 (2024):*** It is better to keep the `zotero_storage` folder in iCloud always available in all your computers by right clicking the folder and selecting **Keep Downloaded**, like this:

![](/images/2022-10-04/keep_dl.png)

This solves the rare problem that Zotero may not be able to locate some PDF files in some computers.

***UPDATE 2 (2025):*** For a Windows setup, check [this post](/posts/2025-02-08-work-syncing-across-different-oss/).