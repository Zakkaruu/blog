---
layout: post
title: Using BackBlaze B2 Storage as a Synchronization Target for Joplin
date: 2022-02-19 23:15
---
<!-- wp:heading -->
<h2 id="introduction">Introduction</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>If you are already using BackBlaze B2 for storage and you are using Joplin, it makes sense to combine the two. Joplin supports syncing notes with AWS S3 buckets (beta feature at time of writing). How does this help us? BackBlaze B2 provides an <a rel="noreferrer noopener" href="https://www.backblaze.com/b2/docs/s3_compatible_api.html" target="_blank">AWS S3 compatible API</a> that Joplin can use.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><em>Please see my recommended steps at the end of this tutorial if you are changing from a previous sync target to BackBlaze B2.</em></p>
<!-- /wp:paragraph -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="prerequisites">Prerequisites</h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li><a href="https://www.backblaze.com/" target="_blank" rel="noreferrer noopener">BackBlaze B2 account</a>.</li><li><a href="https://joplinapp.org/" target="_blank" rel="noreferrer noopener">Joplin application</a> <em>(Desktop or Mobile)</em> that has the S3 target feature.</li><li>Using <span style="text-decoration:underline;">S3 (beta)</span> as the <strong>Synchronization target</strong></li></ul>
<!-- /wp:list -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="required-information-to-sync">Required Information to Sync</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The following information is required to sync your Joplin notes. BackBlaze B2 provides the information using different names.</p>
<!-- /wp:paragraph -->

<!-- wp:table -->
<figure class="wp-block-table"><table><tbody><tr><td><strong>AWS</strong></td><td><strong>BackBlaze B2</strong></td></tr><tr><td>AWS S3 Bucket</td><td>Bucket Unique Name</td></tr><tr><td>AWS S3 URL</td><td>Endpoint</td></tr><tr><td>AWS region                       </td><td><em>Region</em> portion of Endpoint</td></tr><tr><td>AWS access key</td><td>keyID</td></tr><tr><td>AWS secret key</td><td>applicationKey</td></tr></tbody></table></figure>
<!-- /wp:table -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="creating-a-bucket">Creating a Bucket</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>There are only two things you need to do on BackBlaze B2 to have all of the information you need to start syncing.</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Create a new Bucket that has the <strong>Endpoint URL</strong> feature.</li><li>Create a new <strong>App Key</strong> with permissions to new Bucket</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p><strong>Log into your BackBlaze B2 account and create a new bucket under </strong>"B2 Cloud Storage" &gt; Buckets.<strong> The unique bucket name must be globally unique across all B2 buckets.</strong></p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":379,"width":649,"height":190,"sizeSlug":"large","linkDestination":"none","className":"is-style-default"} -->
<div class="wp-block-image is-style-default"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i1.webp?w=1024" alt="" class="wp-image-379" width="649" height="190" /></figure></div>
<!-- /wp:image -->

<!-- wp:image {"align":"center","id":381,"width":495,"height":664,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i2.webp?w=763" alt="" class="wp-image-381" width="495" height="664" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>After creating this bucket, you should see something similar to the following screenshot. It contains an <strong>Endpoint URL</strong> (s3.us-west-000.backblazeb2.com), which we will need along with your <strong>Bucket Name</strong>. One important thing to note is how the endpoint URL is structured:</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"center"} -->
<p class="has-text-align-center"><em>(s3).(us-west-000).(backblazeb2).(com</em>)<br><em>(service).(region).(domain).(tld)</em></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>You will need to extract the region from the URL, as Joplin uses this in the sync target settings.<br><br>Bucket Name = AWS S3 bucket<br>Endpoint URL = AWS S3 URL<br>Region Portion of URL = AWS region<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":423,"width":586,"height":278,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i3.webp?w=1022" alt="" class="wp-image-423" width="586" height="278" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="creating-an-app-key">Creating an App Key</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Create a new App Key under "Account" &gt; App Keys. This is used with the bucket information to give private access to your bucket via Joplin.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":383,"width":510,"height":103,"sizeSlug":"large","linkDestination":"none","style":{"color":[]}} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i4.webp?w=1024" alt="" class="wp-image-383" width="510" height="103" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>It is recommended to limit the key to only the newly created bucket.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":389,"width":509,"height":582,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i5.webp?w=895" alt="" class="wp-image-389" width="509" height="582" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":419,"width":632,"height":206,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i6.webp?w=1024" alt="" class="wp-image-419" width="632" height="206" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The new key provides the last two items we need to set Joplin as a sync target:<br><br>keyID = AWS Access Key<br>applicationKey = AWS Secret Key</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The completed Joplin sync settings are below. <em>(I have already deleted all of the settings created in this blog post, such as the</em> <em>new bucket and app key. So don't get any ideas. </em>😉<em>)</em></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"textAlign":"center","level":5} -->
<h5 class="has-text-align-center" id="noteplease-see-the-following-section-preparing-your-data-for-sync-prior-to-applying-your-new-sync-settings-there-is-a-chance-of-losing-your-data-if-you-do-not-follow-these-next-directions">NOTE<br>Please see the following section <em>(Preparing Your Data for Sync)</em> prior to applying your new sync settings.<br><br><strong><em>There is a chance of losing your data if you do not follow these next directions.</em></strong></h5>
<!-- /wp:heading -->

<!-- wp:image {"align":"center","id":421,"width":382,"height":736,"sizeSlug":"large","linkDestination":"none"} -->
<div class="wp-block-image"><figure class="aligncenter size-large is-resized"><img src="/images/2022-02-19_i7.webp?w=531" alt="" class="wp-image-421" width="382" height="736" /></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="preparing-your-data-for-sync">Preparing Your Data for Sync</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>I'll be honest, I had a hell of a time getting everything to sync properly. Maybe it is just me, but I wanted to include this section to save others the same pain.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The general workflow is going to be:</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li>Export all of your Joplin data as a .jex file.</li><li>Rename the /Users/<em>your-username</em>/.config/joplin-desktop folder to /Users/<em>your-username</em>/.config/joplin-desktop-bak<ul><li>The provided path is for MacOS, it is generally the same on most operating systems. For instance, in Windows it is %userprofile%/.config/joplin-desktop</li><li>This is done to fallback to a working state if something goes wrong (done by deleting the newly created folder and renaming the backup folder to the original name).</li></ul></li><li>Open Joplin<ul><li>It will be as if you opened Joplin for the first time without any data.</li></ul></li><li>Set a new encryption Master Key in Joplin.</li><li>Add the new sync target settings and confirm successful sync.</li><li>Import your .jex backup file.</li><li>Sync all of your data to your BackBlaze B2 bucket.</li><li>Done!</li></ol>
<!-- /wp:list -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="conclusion">Conclusion</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>There are only a few steps and pieces of information needed to move your notes over to BackBlaze B2. I hope this blog post helps you do make the transition much faster than it took me!</p>
<!-- /wp:paragraph -->

<!-- wp:separator -->
<hr class="wp-block-separator" />
<!-- /wp:separator -->

<!-- wp:heading -->
<h2 id="changelog"><span style="text-decoration:underline;">Changelog</span></h2>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>Initial Release<ul><li>2022-02-19</li></ul></li></ul>
<!-- /wp:list -->
