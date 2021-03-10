---
title: From WordPress to 11ty
description: Why I changed the tech stack of my blog and moved on to Netlify.
date: 2021-03-11
tags:
  - blog
  - 11ty
  - netlify
layout: layouts/post.njk
image: /img/ali-yilmaz-LqztBrFUkec-unsplash.jpg
---

Since the beginning of this blog 2019, I was using worpress.com. It was easy to use, fast to set up, has fancy themes, I didn't need much to know about the technologies behind it, and it came with a reasonable good text editor for postings. But the ease of use had a price:

![Hero Image: Keyboard, Photo by Ali Yılmaz on Unsplash](/img/ali-yilmaz-LqztBrFUkec-unsplash.jpg)

- expensive if I want to use plugins and customize my blog
- feels like some sort of vendor lock-in, e.g to export just the comments I would need to pay 300€ (1-year subscription of a business plan, 2021) to just be able to install the necessary plugins as far as I found out
- performance of the provided themes often not very good
- the themes are not really fitting to my needs
- no git-based versioning
- no possibilities to learn about web technologies
- and many tiny things which irritated me

But IMHO the worst problem with wordpress.com is that I had to switch the theme because of compatibility reasons.

![The original blog theme](/img/oldTheme.png)
*The original blog theme*

![The new, but way too heavy and overloaded Wordpress.com theme](/img/newTheme.png)
*The new, but way too heavy and overloaded Wordpress.com theme*

To be honest I didn't like the available themes at all. So I decided to migrate to a new solution using [11ty][1]. With 11ty I can generate fast static websites where my blog posts are based upon markdown files and all images are lazy loading with generated sizes. As a base, I used the 
[eleventy-high-performance-blog][2] template and therefore it was extremely simple to start a project with 11ty.

The only adjustments I had to do were to extend the [njk][3] based page templates with a [disqus][4], for commenting functionality, and newsletter component.

```html
<div id="disqus_thread"></div>
<script>
  var disqus_config = function () {
    this.page.url = "{{metadata.url}}{{ page.url }}";
    this.page.identifier = "{{ page.fileSlug }}";
  };
  (function() {
    var d = document, s = d.createElement('script');
    s.src = 'https://{{ metadata.disqusShortname }}.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
  })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
```
*The few lines of code needed to get the commenting system working*

```html
<div id="mc_embed_signup">
  <form action="https://gmail.us20.list-manage.com/subscribe/post?u=USERNAME&amp;id=ID" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
      <div id="mc_embed_signup_scroll">
        <h3>Subscribe</h3>
        <div class="mc-field-group">
          <label for="mce-EMAIL">Email Address</label>
          <input type="email" value="" name="EMAIL" class="required email" id="mce-EMAIL" style="width: 100%; height:40px">
        </div>
        <p>Please also join my mailing list, seldom mails, no spam, and no advertising. By clicking submit, you agree to share your email address with the site owner and Mailchimp to receive updates from the site owner. Use the unsubscribe link in those emails to opt out at any time. You can unsubscribe at any time by clicking the link in the footer of our emails. For information about our privacy practices, please visit our website. We use Mailchimp as our marketing platform. By clicking below to subscribe, you acknowledge that your information will be transferred to Mailchimp for processing. <a href="https://mailchimp.com/legal/" target="_blank" rel="noreferrer">Learn more about Mailchimp's privacy practices here.</a></p>
        <div id="mce-responses" class="clear">
          <div class="response" id="mce-error-response" style="display:none"></div>
          <div class="response" id="mce-success-response" style="display:none"></div>
        </div>    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
        <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="USERNAME_AND_ID" tabindex="-1" value=""></div>
        <div class="clear"><input style="width: 100%;" type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
      </div>
  </form>
</div>
<script type='text/javascript' src='//s3.amazonaws.com/downloads.mailchimp.com/js/mc-validate.js'></script><script type='text/javascript'>(function($) {window.fnames = new Array(); window.ftypes = new Array();fnames[0]='EMAIL';ftypes[0]='email';fnames[1]='FNAME';ftypes[1]='text';fnames[2]='LNAME';ftypes[2]='text';fnames[3]='ADDRESS';ftypes[3]='address';fnames[4]='PHONE';ftypes[4]='phone';fnames[5]='BIRTHDAY';ftypes[5]='birthday';}(jQuery));var $mcj = jQuery.noConflict(true);</script>
```
*html needed [Mailchimp][5] based newsletter*

To get [LATEX][6] formulas supported it was easy to use [KaTeX][7].

```html
<head>
...
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.13.0/dist/katex.min.css" integrity="sha384-t5CR+zwDAROtph0PXGte6ia8heboACF9R5l/DiY+WZ3P2lxNgvJkQk5n7GPvLMYw" crossorigin="anonymous">
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.13.0/dist/katex.min.js" integrity="sha384-FaFLTlohFghEIZkw6VGwmf9ISTubWAVYW8tG8+w2LAIftJEULZABrF9PPFv+tVkH" crossorigin="anonymous"></script>
    <script defer src="https://cdn.jsdelivr.net/npm/katex@0.13.0/dist/contrib/auto-render.min.js" integrity="sha384-bHBqxz8fokvgoJ/sc17HODNxa42TlaEhB+w8ZJXTc2nZf1VgEaFZeZvT4Mznfz0v" crossorigin="anonymous" 
      onload="renderMathInElement(document.body,
          {
              delimiters: [
                  {left: '$$', right: '$$', display: true},
                  {left: '$', right: '$', display: false},
              ]
          });">
    </script>
...
</head>
```

## tl;dr

I migrated and simplified my thoughts-on-coding-com blog with 11ty but unfortunately, I can't migrate the already existing comments. Because of 11ty and the nice eleventy-high-performance-blog template, the blog got a nice and clean appearance and good results at Googles Lighthouse.

![Google Lighthouse results](/img/lighthouse.png)
*Google Lighthouse results*

[1]: https://www.11ty.dev/
[2]: https://github.com/google/eleventy-high-performance-blog
[3]: https://mozilla.github.io/nunjucks/
[4]: https://disqus.com/
[5]: https://mailchimp.com
[6]: https://www.latex-project.org/
[7]: https://katex.org/