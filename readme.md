WP Consent API
======================
**Contributors**: RogierLankhorst, mrxkon, aurooba

**Tags**: consent, privacy

**Requires at least**: 5.0

**License**: GPL2

**Tested up to**: 5.3

**Requires PHP**: 5.6

**Stable tag**: 1.0.0

Description
-----------
Simple Consent API to read and register the current consent category, allowing consent management plugins and other plugins to work together, improving compliancy.

What problem does this plugin solve?
------------------------------------
Currently it is possibly for a consent management plugin to block third party services like Facebook, Google Maps, Twitter, etc. But if a WordPress plugin places a PHP cookie, a consent management plugin cannot prevent this.

Secondly, there are plugins that integrate the cookie placing code on the clientside in javascript files that, when blocked, break the site.

Or, if such a plugin's javascript is minified, causing the URL to be unrecognizable, it won't get detected by an automatic blocking script.

Lastly, the cookie blocking approach requires a list of all types of URL's that place cookies. A generic API where plugins adhere to can greatly
facilitate a webmaster in getting a site compliant.

Does usage of this API prevent third party cookies from being set?
------------------------------------------------------------------
Primary this API is aimed at compliant setting of first party cookies by WordPress plugins. If such a plugin triggers for example Facebook, usage of this API will be of help. If a user manually embeds a facebook iframe, a cookie blocker is needed that initially disables the iframe and or scripts.

Third party scripts have to blocked by a cookie blocking functionality in a consent management plugin. To do this in core would be to intrusive, and is also not applicable to all users: only users with visitors from opt in regions such as the European Union require such a feature. Such a feature also has a risk of breaking things. Additionally, blocking these and showing a nice placeholder, requires even more sophisticated code, all of which should in my opinion not be part of WordPress core, for the same reasons.

That said, the consent API can be used to decide if an iframe or script should be blocked. 

How does it work?
-----------------
There are two indicators that together tell if consent is given for a certain consent category, e.g. "marketing":

1) the region based `consent_type`, which
can be `opt-in`, `opt-out`, or other possible `consent_types`;
2) and the visitor's choice: `not set`, `allow` or `deny`.

The `consent_type` is a function that wraps a filter,`wp_get_consent_type`. If there's no consent management plugin to set it, it will return `false`. This will cause all consent categories to return `true`, allowing cookies to be set on all categories.

If `opt-in` is set using this filter, a category will only return `true` if the value of the visitor's choice is `allow`.

If the region based `consent_type` is `opt-out`, it will return `true` if the visitor's choice is not set or is `allow`.

Clientside, a consent management plugin can dynamically manipulate the consent type, and set the several cookie categories.

A plugin can use a hook to listen for changes, or check the value of a given category.

Categories, and most other stuff can be extended with a filter.

## Existing integrations

- Complianz https://github.com/rlankhorst/complianz-gdpr/compare/consent-API-integration
- Example plugin shipped with this plugin. The plugin basically consists of a shortcode, with a div that shows 
a tracking or not tracking message. No actual tracking is done :)

## Demo site
http://relieved-snake.w5.wpsandbox.pro/

plugins used to set this up:
- Complianz
- The example plugin https://github.com/rlankhorst/consent-api-example-plugin

Code Examples
-------------
If you have any other code suggestions, please PR them on GitHub!

### javascript, consent management plugin
```javascript
//dynamically set consent type
window.wp_consent_type = 'optin';

//consent management plugin sets cookie when consent category value changes
wp_set_consent('marketing', 'allow');
```

### javascript, tracking plugin
```javascript
//listen to consent change event
document.addEventListener("wp_listen_for_consent_change", function (e) {
  var changedConsentCategory = e.detail;
  for (var key in changedConsentCategory) {
    if (changedConsentCategory.hasOwnProperty(key)) {
      if (key === 'marketing' && changedConsentCategory[key] === 'allow') {
        console.log("just given consent, set marketing cookie")
      }
    }
  }
});

//basic implementation of consent check:
if (wp_has_consent('marketing')){
  activateMarketing();
  console.log("set marketing stuff now!");
} else {
  console.log("No marketing stuff please!");
}
```
### PHP
```php
//declare complianz with consent level API
$plugin = plugin_basename( __FILE__ );
add_filter( "wp_consent_api_registered_{$plugin}", '__return_true' );

//check if user has given marketing consent. Possible consent categories/purposes:
//functional, preferences', statistics', statistics-anonymous', statistics', marketing',
if (wp_has_consent('marketing')){
  //do marketing stuff
}

//set the consent type (optin, optout, default false)
add_filter( 'wp_get_consent_type', 'my_set_consenttype' , 10, 1 );
function my_set_consenttype($consenttype){
  return 'optin';
}

//filter consent categories types, example: remove the preferences category
add_filter( 'wp_consent_categories', 'my_set_consentcategories' , 10, 1 );
function my_set_consentcategories($consentcategories){
  unset($consentcategories['preferences']);
  return $consentcategories;
}
```

Installation
------------
To install this plugin:

1. Download the plugin
2. Upload the plugin to the wp-content/plugins directory,
3. Go to “plugins” in your WordPress admin, then click activate.

Frequently asked questions
--------------------------
**Does this plugin block cookies from being placed?**
No, this plugin provides a framework through which plugins can know if they are allowed to place cookies.
The plugin requires both a consent management plugin for consent management, and a plugin that follows the consent level as can be read from this API.

Changelog
---------
### 1.0.0

Upgrade notice
--------------

Screenshots
-----------
