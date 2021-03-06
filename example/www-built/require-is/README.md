Require-is
===

A conditional loading plugin for [Require-JS](http://requirejs.org/) that works with optimizer builds and build layers.

Use cases:
* Provide a module on the server only, and exclude it on the client.
* Provide a different build of a project for mobile.
* Provide an entire build layer bringing together IE-specific JS across all modules, so that most users need not download these.
* etc.!

Installation
---

Using [volo](https://github.com/volojs/volo):
```
volo add guybedford/require-is
```

Alternaively, download is.js and place it in the baseUrl folder of a Require-JS project.

Usage
---

Where conditional loads are needed, use the RequireJS plugin syntax:

```
is![conditionId]?[moduleId]
```

The negation form is also accepted:

```
is!~[conditionId]?[moduleId]
```

### Example:

```javascript
define(['is!mobile?mobile-code'], function(mobileCode) {
  
});
```

If the condition, `[mobile]` is true, then the module `mobile-code` is loaded, otherwise `null` is returned.

The above applies equally well to builds with the [RequireJS optimizer](http://requirejs.org/docs/optimization.html).

Setting Conditions
---

There are three ways to set conditions.

**1. Using the RequireJS configuration:**

```javascript
requirejs.config({
  config: {
    is: {
      mobile: true
    }
  }
});
```

Any number of conditions can be set to true or false in this way.

**2. Dynamically during runtime:**

```javascript
require(['is'], function(is) {
  is.set('mobile');
});
```

Any further loads will then use the condition.

**3. Most importantly, as a conditional checking module, located at featureId:**

Eg:
**In 'iphone.js':**
```javascript
define(function() {
  if (typeof navigator === 'undefined' || typeof navigator.userAgent === 'undefined')
    return false;
  return navigator.userAgent.match(/iPhone/i);
});
```

**The condition will be checked in the above order. If none of the above return either 'true' or 'false', an error will be thrown.**

**Note that conditions are constants. They can only be set once. As soon as a condition has been used in the loading of a module, it cannot be changed again and any attempts to set it will throw an error.**

This is done in order that conditions can always be built into modules with the RequireJS optimizer.

### Checking conditions

Sometimes it may be useful to check the value of a condition as well.

To do this, simply use the plugin with square brackets around the condition name.

Example:

```javascript
require(['is![mobile]', function(mobile) {
  if (mobile) {
    //only true if mobile condition has been set
  }
});
```

Optimizer Configuration
---

When running a build with the RequireJS Optimizer, the environment must be fully specified through the build configuration, as in (1) above.

Each condition must be set to `true` or `false` indicating whether to include those conditional modules in the build or not.

By default, dynamic conditions defined as in (3) above will all be built in.

In this way, the modules are all built in or not, and depending on the runtime conditions which can be entirely different to the build config,
modules can be dynamically requested either remotely or from the build.

For example, one can do an inclusive 'desktop and mobile' build with the following config:

```javascript
config: {
  is: {
    mobile: true
  }
}
```

This builds the mobile modules into the built layers, as well as the mobile detection module, 'mobile.js'. Then dependening on the environment configuration, these modules
will either get excluded or included as needed.

To exclude the detection module, a separate build config can be specified of the form:

```javascript
{
  isExcludeDetection: ['mobile', ...]
}
```

###Separate build layers

To separate, say, all the mobile code into its own build layer, provide the following build configuration:

```javascript
config: {
  is: {
    mobile: false
  }
},
modules: [
  {
    name: 'main',
    include: ['main', 'modules', 'here'],
  },
  {
    name: 'mobile-only',
    include: ['is!mobile*'],
    exclude: ['main']
  }
]
```

In this way, all the separate `is!mobile?mobile-script` requires called by all the previous modules, can be compiled into a single 'mobile' layer that when loaded
results in no need to separately load the mobile modules from many resources.