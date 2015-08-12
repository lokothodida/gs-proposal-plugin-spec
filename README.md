# GetSimple Proposal Plugin Specification
This is a proposal specification for how plugin development could eventually look in
a later version of the GetSimple CMS. This is not a repository of usable code, but
a sketch of a possible core API for plugin development.

* [Aims](#aims)
* [Plugin Structure](#plugin-structure)
* [API](#api)

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
  - lang/
    - .htaccess                     "Deny from all"
    - en_US.json
  - .htaccess                       "Deny from all"
  - manifest.json
  - index.php
  - admin.php
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

// Create an administration panel
$plugin->admin(function() {
  echo '<h3>' . $plugin->i18n('PLUGIN_TITLE') . '</h3>';
  echo '<p>' . $plugin->i18n('ADMIN_MESSAGE') . '</p>';
});
```

## `plugin_name/admin.php`
Code that will be executed on in the administration panel.

### Example
```html+php
<h3><?php echo $plugin->i18n('PLUGIN_TITLE'); ?></h3>

<p><?php echo $plugin->i18n('ADMIN_MESSAGE'); ?></p>
```

# API
## `Plugin`
The `Plugin` class encapsulates some helpful methods for plugin developers.

### `i18n(string $hash)`
Internationalized string in accordance with the hashes defined in the lang/ directory.

```php
echo $plugin->i18n('PLUGIN_TITLE');
```

### `hook(string $name, function $callback)`
Registers a hook.

```php
$plugin->hook('common', function() {
  echo 'All plugins have loaded at this point!';
});
```

### `trigger(string $name, array $arguments)`
Trigger a hook. Used for communicating between plugins.

```php
// Some other plugin has the following code to register a hook:
// $plugin->hook('some-custom-hook', function(/* */) { /* */ });
// To execute it, run:
$plugin->trigger('some-custom-hook', $args = array(/* */));
```

### `filter(string $name, function $callback)`
Register a filter.

```php
$plugin->filter('content', function($content) {
  // Do something to the page content and return it
});
```

### `admin(function $callback)`
Registers function that will be executed on the admin panel

```php
$plugin->admin(function() {
  // Your admin panel
});
```

### `admin(mixed $pattern, function $callback)`
Registers function that will be executed on the admin panel when the
url matches the `$pattern`.

```php
$plugin->admin(array('', false), function() {
  // Main admin panel page
  // Also executes when no other admin page can be found
});

$plugin->admin('/action=edit&slug=[a-zA-Z0-9]*/', function($slug) {
  // Executes when url is ?id=plugin_name&action=edit&slug=some-slug
  // The matches in the regex are passed to the function's parameters
});
```

### `url([string $suffix])`
URL to the admin panel for the plugin

```html+php
<!--link to admin panel-->
<a href="<?php echo $plugin->url(); ?>">
  ...
</a>

<!--link to a page in the admin panel-->
<a href="<?php echo $plugin->url('action=action1'); ?>">
  ...
</a>
```

### `path([string $suffix])`
Canonical path to the plugin's folder

```php
// Includes the plugins/plugin_name/inc/myscript file
include $plugin->path('inc/myscript.php');
```

## `FileJSON`

The `FileJSON` class handles data manipulation (through JSON files).

### `__construct()`

```php
$file = new FileJSON(GSDATAOTHERPATH . '/datum.json');
```

### `exists()`

```php
if (!$file->exists()) {
  // ...
}
```

### `save()`
Save data to the file.

```php
$file->save(array(
  'title'   => 'Title',
  'content' => 'Page content',
  'pubdate' => time(),
));
```

### `get()`
Get data from the file

```php
$data = $file->get();
echo $data['title'];
```

### `move()`

```php
$file->move('somethingelse.json');
```

### `delete()`

```php
$file->delete();
```
