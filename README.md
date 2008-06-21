Page Attachments
===

About
---

A [Radiant][rd] Extension by [Sean Cribbs][sc] that adds page-attachment-style asset management.

Installation
---

If you want `page_attachments` to generate and display thumbnails of your uploaded images you'll first need to install one of, [`image_science`][is], [`mini-magick`][mm] or [`rmagick`][rm] on your server. This is completely optional, `page_attachments` will still function in every other way without any of these packages installed.

Next, you need to install the `attachment_fu` plugin.

    `cd /path/to/radiant`
    `git clone git://github.com/technoweenie/attachment_fu.git vendor/plugins/attachment_fu`

If you don't have `git` installed you can simply [download a tarball][af] of `attachment_fu`, upload and unpack it to your `vendor/plugins` directory.

**Failing to install the `attachment_fu` plugin before proceeding will result in an "`uninitialized constant Technoweenie::AttachmentFu`" error when you try to install `page_attachments`.**

Now you're ready to install `page_attachments`.

    `cd /path/to/radiant`
    `git clone git://github.com/radiant/radiant-page-attachments-extension.git vendor/extensions/page_attachments`
    `rake radiant:extensions:page_attachments:migrate`
    `rake radiant:extensions:page_attachments:update`

If you don't have `git` you can [download a tarball][pa].

During the `migrate` task you may encounter an "`uninitialized constant Technoweenie::AttachmentFu::Backends`". If this happens try `git clone git://github.com/technoweenie/attachment_fu.git vendor/extensions/page_attachments/vendor/plugins/attachment_fu` and re-run the `migrate` task.

Restart your server and refresh the admin interface.

Managing Attachments
---

Now when you login and edit a page, you'll find the "Attachments" interface below the text editing area. To add a new attachment to the page click the **+** icon. If you need to you can upload multiple attachments at once by clicking the **+** icon once for each attachment you'll be adding. Attachments are **not** added or deleted until the page is saved. Therefore, if you accidentally deleted something you meant to keep, simply cancel the page edit.

Usage
---

* Reference an attachment by name `<r:attachment name="file.txt">...</r:attachment>`
* Display an attachment's URL `<r:attachment:url name="file.jpg"/>`
* Display an attachment's `#{key}` attribute `<r:attachment:#{key} name="file.jpg"/>`
* Display the date an attachment was added `<r:attachment:date name="file.txt"/>`
* Display an attached image `<r:attachment:image name="file.jpg"/>`
* Display a link to an attachment `<r:attachment:link name="file.jpg"/>` or `<r:attachment:link name="file.jpg">Click Here</r:attachment:link>`
* Display name of the user who added the attachment `<r:attachment:author name="file.jpg"/>`
* Iterate through all the attachments on a page `<r:attachment:each><r:link/></r:attachment:each>`

Troubleshooting
---

If you have a problem running the migrate task, and it fails with an error something like this:

    ld: library not found for -lfreeimage
    collect2: ld returned 1 exit status

It means you have installed the ImageScience gem but you don't have FreeImage installed. So, either install FreeImage or uninstall the gem with

    gem uninstall image_science

---

If you're using ImageScience as a user without a home directory, you may see this error:

    Define INLINEDIR or HOME in your environment and try again

This is caused by RubyInline not having a place to store its generated files and can be easily fixed by specifying a path in your environment file like so:

    ENV['INLINEDIR'] = File.join(RAILS_ROOT,'tmp','ruby_inline')

Amazon S3 for Attachment storage
---

Since `page_attachments` uses `attachment_fu` for the handling of attachments it's just as easy to use [S3][s3] as it is to use your hard drive. Before you get started with this there are a few things to keep in mind:

* If you've already started storing attachments on your hard drive **this will break** any `<r:attachment...>` tags pointing to those files. You'll need to remove all existing attachments and re-add them to Amazon S3.
* You have to install the `AWS::S3` gem. In some shared hosting environments this might not be possible.
* The `AWS::S3` gem does not (currently) support EU buckets, so if that's all you have you'll need to create a bucket in the US.

Before you start make sure you have `page_attachments` working using your hard drive. Once you've tested an upload or two to the hard drive and feel confident the basic setup is working, dive right in.

1. `gem install aws-s3`
2. `cd /path/to/radiant`
3. `cp vendor/plugins/attachment_fu/amazon_s3.yml.tpl config/amazon_s3.yml`
4. edit `config/amazon_s3.yml` with your S3 credentials
5. `cp vendor/extensions/page_attachments/app/models/page_attachment.rb vendor/extensions/page_attachments/app/models/page_attachment.rb.bak`
6. edit line 2 of `vendor/extensions/page_attachments/app/models/page_attachment.rb` changing `:file_system` to `:s3`
7. restart your server

Add an attachment and make sure the link it gives back is on S3. You should see all your attachments start showing up at `http://s3.amazonaws.com/bucket-name/page_attachments/`. While it is possible to customize the URL to Amazon (i.e. http://attachments.your-domain.com/) but it's beyond the scope of this document and a task best left for those that really need custom URLs.

[rd]: http://radiantcms.org/
[sc]: http://seancribbs.com/
[is]: http://seattlerb.rubyforge.org/ImageScience.html
[mm]: http://rubyforge.org/projects/mini-magick/
[rm]: http://rmagick.rubyforge.org/
[af]: http://github.com/technoweenie/attachment_fu/tree/master
[pa]: http://github.com/radiant/radiant-page-attachments-extension/tarball/master
[s3]: http://www.amazon.com/gp/browse.html?node=16427261