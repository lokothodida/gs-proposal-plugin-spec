# GetSimple Proposal Plugin Specification
This is a proposal specification for how plugin development could eventually look in
a later version of the GetSimple CMS. This is not a repository of usable code, but
a sketch of a possible core API for plugin development.

# Aims
* Encapsulate the plugin's environment in a sandbox
* Reduce the usage of global variables
* Modularize the codebase with objects as much as possible
* Provide a more flexible hooking system to allow greater communication between plugins
* Provide a cleaner/simpler interface for getting/saving data
* Provide a simpler interface for accessing GetSimple site data (e.g. settings)

# Plugin Structure
```
- plugin_name/
  - plugin_name/manifest.json
  - plugin_name/index.php
  - plugin_name/admin.php
  - .htaccess                       "Deny from all"
```

## `plugin_name/`
All plugin files (except data) are saved in the plugin's folder. The folder's name
acts as the id for the plugin.

## `plugin_name/manifest.json`
Defines core properties about the plugin and rudimentary configuration for the plugin
author. Plugin registration will use the data from this manifest.

### Example

```javascript
{
  "name"         : "Plugin Name",           // Plugin name
  "version"      : "0.1",                   // Plugin version
  "author"       : "You",                   // Plugin author
  "website"      : "https://yoursite.com",  // Author website
  "default_lang" : "en_US",                 // Default language
  "main"         : "index.php"
  "admin"        : "admin.php",             // Script to run for the admin panel
  "description"  : "i18n.PLUGIN_DESC",
  "tab"          : "pages",
  "dependencies" : [],
  "sidebars" : [
    {
      "label" : "i18n.SIDEBAR1_LABEL",
      "url"   : "action=action1"
    },
    {
      "tab"   : "settings",
      "label" : "i18n.SIDEBAR2_LABEL",
      "url"   : "action=action2"
    },
  ]
}
```

## `plugin_name/index.php`
Code that will be executed on both the front and back end of the site.
Most plugin hooks will be registered here.

### Example
```php
<?php
// All variables declared in this file are protected by a closure
// $plugin is given to the author: it is an object with data about
// the plugin and has utility methods

// Add a hook that executes when all plugins are loaded
$plugin->hook('common', function() {
  
});

// Add a filter that executes on the page content
$plugin->filter('content', function($content) {
  return str_replace('somestring', 'someotherstring', $content);
});

// Trigger a hook
$plugin->trigger('some-hook', $args = array());
```

## `plugin_name/admin.php`
Code that will be executed on in the administration panel.

### Example
```html+php
<h3><?php echo $plugin->i18n('PLUGIN_TITLE'); ?></h3>

<p><?php echo $plugin->i18n('ADMIN_MESSAGE'); ?></p>
```
