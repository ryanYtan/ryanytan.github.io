# Using ES6 Modules in WordPress Plugins
So you want to split up your Javascript code into multiple files so you aren't
staring at a 1000-line long file every time you want to make changes. The great
way to do this is with ES6 modules. However, attempting to do this will result
in getting errors when you attempt to import a function from an ES6 module. This
is because in the raw HTML served by the server, you need the `type="module"`
attribute within the `<script>` tag.

Depending on how your website is built, you may be able to directly add the
attribute to the PHP file that is being served to the front-end. However, in
this context, we are using a plugin that uses `add_action` to enqueue scripts to
the front-end, so we don't have direct control over a PHP file being sent to the
front-end. What we can do, instead, is to hook into the part of WordPress that
writes the `<script>` tag to the HTML page, then insert the `type="module"`
attribute ourselves. To do this, we define the following filter:
```php
//in whatever file you register your hooks in
function set_scripts_type_attribute( $tag, $handle, $src )
{
    if (str_contains($handle, MY_HANDLE)) {
        $tag = '<script type="module" src="' . $src . '"></script>';
    }
    return $tag;
}

//Use "script_loader_tag" hook. Note the final argument is required to tell
//WordPress the number of arguments of our function, else you may get an error.
//The "10" is a default value for the priority of the hook
add_filter("script_loader_tag", "set_scripts_type_attribute", 10, 3);
```
I've found that after doing this, the `import` statements must be
written a certain way, otherwise you may get errors. The following seems to
work well in most cases:
```js
import { functionToImport, function2ToImport } from "./other-module.js";
```
