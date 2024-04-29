+++
title = 'How to Convert Multiple Blog Posts to EPUB Easily'
date = 2024-04-15T00:00:00-07:00
+++

**TL;DR: I created a Chrome extension that allows you to select multiple blog posts and convert them into a single EPUB file for easy reading.**

As a blog reader, I often found myself bookmarking posts to read later, only to have a cluttered list of bookmarks that I would easily forget about. I wanted a better way to save and read blog content offline.

After exploring various options for saving webpages as EPUB, I couldn't find a solution that allowed me to organize multiple posts into one file easily. So I decided to build my own.

## Here's how it works

Detailed steps are also outlined in https://blog2epub.vercel.app/:

### Step 1: Install the extension

Head to the Chrome Web Store and install the Blog to EPUB [extension](https://chromewebstore.google.com/detail/blog-to-epub/ffolllebnagcedlagopfobpaohndjbmb)

![step 1](/step1.gif)

### Step 2: Select articles to convert

When you're on a blog homepage, open the extension and click "Select Articles". You can then single-click on article links to select or deselect them, or double-click to select or deselect an entire list of articles.

![step 2](/step2.gif)

### Step 3: Confirm your selection

After selecting the articles you want to convert, click "Generate EPUB" in the extension. This will open a new page where you can review your selections and download the EPUB file.

![step 3](/step3.gif)

### Step 4: Download the EPUB

Click the "Download EPUB" button, and your customized EPUB file will be ready in your downloads folder, ready to be transferred to your e-reader.

## Takeaways

Without much prior experience in building browser extensions, I gained a deeper understanding of managing message passing between different extension components and how they work together.

One challenge I faced was finding the right approach for fetching and parsing the content of the selected blog posts. I explored various libraries and methods, and ultimately made the decision on using Puppeteer and Readability to reliably extract the content from the webpages.

Another consideration was the hosting and scalability of the project. Since the blog post conversion is handled on server side, I'll need to monitor the usage and potentially upgrade the hosting plan if there's higher traffic.

Overall, this project has been a valuable exercise in building a tool to solve a personal pain point. I'm excited to continue improving and expanding the functionality of Blog to EPUB, and I welcome feedback from users to help shape its future development.
