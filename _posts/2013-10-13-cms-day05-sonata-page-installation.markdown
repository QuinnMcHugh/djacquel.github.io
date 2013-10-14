---
layout: post
title: "Symfony CMS - day 05 - Sonata Page Installation"
description: ""
category:
tags: []
published: true
comments: false
---

Installing Donata Page Bundle in order to finish our CMS.


Hello ! Fifth day of our tuto on how to make a small CMS with Symfony and Sonata bundles.
Today we'll install Sonata Page Bundle.

##Install

In your *composer.json* file, add :

{% highlight json %}
[...]
    "require": {
        [...],
        "symfony-cmf/routing": "1.1.*@dev",
        "symfony-cmf/routing-bundle": "1.1.*@dev",
        "sonata-project/page-bundle": "2.3.*@dev",
        "sonata-project/seo-bundle": "1.*",
        "sonata-project/cache-bundle": "2.1.*@dev",
        "sonata-project/notification-bundle": "2.2.*@dev"
    },
[...]
{% endhighlight %}

Then run a
{% highlight bash %}
composer update
{% endhighlight %}

## Sonata Page Bundle configuration

Enable our new bundles in the kernel :
{% highlight php %}
// app/AppKernel.php
public function registerBundles()
{
    return array(
        // ...
        new Sonata\PageBundle\SonataPageBundle(),
        new Sonata\SeoBundle\SonataSeoBundle(),
        new Sonata\CacheBundle\SonataCacheBundle(),
        new Sonata\NotificationBundle\SonataNotificationBundle(),
        new Symfony\Cmf\Bundle\RoutingBundle\CmfRoutingBundle(),
    );
}
{% endhighlight %}

{% highlight yaml %}
# app/config/config.yml
[...]
cmf_routing:
    chain:
        routers_by_id:
            # enable the DynamicRouter with high priority to allow overwriting configured routes with content
            #cmf_routing.dynamic_router: 200
            # enable the symfony default router with a lower priority
            sonata.page.router: 150
            router.default: 100

sonata_page:
    multisite: host
    use_streamed_response: true # set the value to false in debug mode or if the reverse proxy does not handle streamed response
    ignore_route_patterns:
        - ^(.*)admin(.*)   # ignore admin route, ie route containing 'admin'
        - ^_(.*)          # ignore symfony routes

    ignore_routes:
        - sonata_page_cache_esi
        - sonata_page_cache_ssi
        - sonata_page_js_sync_cache
        - sonata_page_js_async_cache
        - sonata_cache_esi
        - sonata_cache_ssi
        - sonata_cache_js_async
        - sonata_cache_js_sync
        - sonata_cache_apc

    ignore_uri_patterns:
        - ^/admin\/   # ignore admin route, ie route containing 'admin'

    page_defaults:
        homepage: {decorate: false} # disable decoration for homepage, key - is a page route

    default_template: default # template key from templates section, used as default for pages
    templates:
        default: {path: 'SonataPageBundle::layout.html.twig', name: default }

    # manage the http errors
    catch_exceptions:
        not_found: [404]    # render 404 page with "not_found" key (name generated: _page_internal_error_{key})
        fatal:     [500]    # so you can use the same page for different http errors or specify specific page for each error

{% endhighlight %}

## Extending Page Bundle

We create our extension for Page Bundle. This mainly allows us to make change to the bundle behavior and model in a centralized maner.

{% highlight bash %}
app/console sonata:easy-extends:generate SonataPageBundle --dest="./src"
{% endhighlight %}

Then you can now update doctrine mapping :
{% highlight yaml %}
# app/config/config.yml
[...]
doctrine:
    orm:
        entity_managers:
            default:
                mappings:
                    ApplicationSonataUserBundle: ~
                    SonataUserBundle: ~
                    FOSUserBundle: ~
                    ApplicationSonataPageBundle: ~ # only once the ApplicationSonataPageBundle is generated
                    SonataPageBundle: ~

[...]
{% endhighlight %}

Enable the page bundle extension in the Kernel :
{% highlight php %}
// app/AppKernel.php
public function registerBundles()
{
    return array(
        // ...
        new Application\Sonata\PageBundle\ApplicationSonataPageBundle(),
    );
}
{% endhighlight %}