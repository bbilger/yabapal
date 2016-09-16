---
id: 70
title: Using jQuery Mobile Icon Pack with PrimeFaces Mobile
date: 2013-03-02T00:07:35+00:00
author: Björn Bilger
layout: post
guid: http://yabapal.wordpress.com/?p=70
permalink: /2013/03/02/using-jquery-mobile-icon-pack-with-primefaces-mobile/
categories:
  - Java EE
  - JSF
tags:
  - Font Awesome
  - jQuery Mobile
  - jQuery Mobile Icon Pack
  - jQuery UI
  - JSF
  - PrimeFaces
  - PrimeFaces Mobile
---
If you want to use jQuery Mobile Icon Pack together with PrimeFaces Mobile, then there are several things you must fix. This posting describes the necessary steps.<!--more-->

For simplicity we will use the following structure to include jQuery Mobile Icon Pack into our project:

```
+ resources
    + css
        + font
            FontAwesome.otf
            fontawesome-webfont.eot
            fontawesome-webfont.ttf
            fontawesome-webfont.woff
        + images
            ajax-loader.png (nonessential)
            icons-18-black-pack.png
            icons-18-white-pack.png
            icons-36-black-pack.png
            icons-36-white-pack.png
        jqm-icon-pack-3.0.0-fa.css</pre>
```

If this worked, I wouldn&#8217;t have written this post. The problem is that &#8220;jqm-icon-pack-3.0.0-fa.css&#8221; cannot find the fonts and images. This is because the requests from the client are relative to the request path of the CSS file. The requests look like that:

`http://.../javax.faces.resource/font/fontawesome-webfont.woff`

But they must look like that:

`http://.../javax.faces.resource/font/fontawesome-webfont.woff.jsf?ln=css`

So the solution to this problem is to fix those requests. This can either be done by rewriting the requests with UrlRewriteFilter, for example, or by &#8220;patching&#8221; jqm-icon-pack-3.0.0-fa.css&#8221;. We want to patch the CSS file. In order to fix the requests we have to replace **all** &#8220;relative&#8221; url references.

So we replace for instance:

`url("font/fontawesome-webfont.woff")`

by:

`url("#{resource['css:font/fontawesome-webfont.woff']}")`

That&#8217;s all we have to do for patching the CSS file. Knowing this technique now, you are able to rewrite the urls, too, and restructure your resources poperly: place fonts in a font folder and images in an image folder.

In addition you should define proper MIME types for the font files. This can be done by adding the following lines to your web.xml:

``` xml
<mime-mapping>
    <extension>otf</extension>
    <mime-type>font/opentype</mime-type>
</mime-mapping>
<mime-mapping>
    <extension>ttf</extension>
    <mime-type>application/x-font-ttf</mime-type>
</mime-mapping>
<mime-mapping>
    <extension>woff</extension>
    <mime-type>application/x-font-woff</mime-type>
</mime-mapping>
<mime-mapping>
    <extension>eot</extension>
    <mime-type>application/vnd.ms-fontobject</mime-type>
</mime-mapping>
```

Note: There aren&#8217;t standardized MIME types for those files and there are a lot of debates on which MIME types are the best for those files. So feel free to use different ones.

This fixes the extended images of the icon pack, only, but Font Awesome icons still won&#8217;t show up. The reason is that PrimeFaces uses jQuery UI and jQuery UI interferes with jQuery Mobile used by PrimeFaces Mobile. The problem is that both use the CSS class &#8220;ui-icon&#8221;. With a simple CSS rule, we can fix that problem:

``` css
.ui-btn-inner > .ui-icon {
    text-indent : 0px;
}
```

Now the Font Awesome icons will show up, too.

There&#8217;s one last issue, I had with icons positioned to the top, using &#8220;jqm-icon-pack-3.0.0-fa.css&#8221;. The margin was set to a wrong value. I think that this cannot be fixed with some CSS, so I fixed the following &#8220;.ui-icon&#8221; rule in &#8220;jqm-icon-pack-3.0.0-fa.css&#8221; by removing the declaration &#8220;margin-top:-10px !important;&#8221;.

If you followed the steps in this posting, the following example page should show up properly.

``` xml
  <f:view xmlns="http://www.w3.org/1999/xhtml"
    xmlns:h="http://java.sun.com/jsf/html"
    xmlns:f="http://java.sun.com/jsf/core"
    xmlns:p="http://primefaces.org/ui"
    xmlns:pm="http://primefaces.org/mobile"
    contentType="text/html">


<h:outputStylesheet library="css" name="fixes.css"/>
<h:outputStylesheet library="css" name="jqm-icon-pack-3.0.0-fa.css"/>
<pm:page title="Hello World">
    <pm:view id="main">
        <pm:header title="Header" />
        <pm:content>
            <a href="index.html" data-role="button" data-icon="spinner" data-iconpos="top">Test</a>
        </pm:content>
    </pm:view>
</pm:page>

</f:view>
```

Note: &#8220;fixes.css&#8221; contains the &#8220;text-indent&#8221; fix, shown above.

Hope this helps!